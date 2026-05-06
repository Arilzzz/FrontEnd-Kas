<script setup>
import { ref, computed, onMounted } from 'vue'
import StudentLayout from '../components/StudentLayout.vue'
import DashboardSummaryCards from '../components/Dashboard/DashboardSummaryCards.vue'
import FinancialChart from '../components/Dashboard/FinancialChart.vue'
import RecentLedger from '../components/Dashboard/RecentLedger.vue'
import PaymentMatrix from '../components/Dashboard/PaymentMatrix.vue'
import api from '../services/api'

// Logic exactly like Dashboard.vue but simplified
const students = ref([])
const payments = ref([])
const expenses = ref([])
const loading = ref(true)

const WEEKLY_DUES = 2000;

// Formatters
const formatRupiah = (num) => new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 }).format(num)
const formatShortRupiah = (num) => {
  if (num >= 1000000) return `Rp ${(num / 1000000).toFixed(1)}M`
  if (num >= 1000) return `Rp ${(num / 1000).toFixed(0)}k`
  return `Rp ${num}`
}

const formatDate = (dateString) => {
  const d = new Date(dateString)
  if (isToday(d)) return `Today, ${d.toLocaleTimeString('en-US', { hour: '2-digit', minute: '2-digit' })}`
  if (isYesterday(d)) return `Yesterday`
  return d.toLocaleDateString('en-GB', { day: 'numeric', month: 'short', year: 'numeric' })
}

// Date Helpers
const isToday = (d) => {
  const today = new Date()
  return d.getDate() === today.getDate() && d.getMonth() === today.getMonth() && d.getFullYear() === today.getFullYear()
}
const isYesterday = (d) => {
  const y = new Date()
  y.setDate(y.getDate() - 1)
  return d.getDate() === y.getDate() && d.getMonth() === y.getMonth() && d.getFullYear() === y.getFullYear()
}

const getMonday = (d) => {
  d = new Date(d);
  var day = d.getDay(), diff = d.getDate() - day + (day == 0 ? -6 : 1);
  return new Date(d.setDate(diff)).setHours(0,0,0,0);
}

const isThisWeek = (dateString) => {
  const date = new Date(dateString).getTime();
  const startOfWeek = getMonday(new Date());
  const endOfWeek = startOfWeek + 7 * 24 * 60 * 60 * 1000;
  return date >= startOfWeek && date < endOfWeek;
}

const isLastMonth = (dateString) => {
  const d = new Date(dateString);
  const today = new Date();
  let lastMonth = today.getMonth() - 1;
  let year = today.getFullYear();
  if (lastMonth < 0) { lastMonth = 11; year--; }
  return d.getMonth() === lastMonth && d.getFullYear() === year;
}

// 1. Current Balance
const currentBalanceRaw = computed(() => {
  const totalIn = payments.value.reduce((sum, p) => sum + Number(p.jumlah_pemasukkan), 0)
  const totalOut = expenses.value.reduce((sum, e) => sum + Number(e.jumlah_pengeluaran), 0)
  return totalIn - totalOut
})
const currentBalance = computed(() => formatRupiah(currentBalanceRaw.value))

// Balance vs last month trend
const balanceTrend = computed(() => {
  const incomeThisMonth = payments.value.filter(p => new Date(p.tanggal_pemasukkan).getMonth() === new Date().getMonth()).reduce((s,p)=>s+Number(p.jumlah_pemasukkan),0)
  const incomeLastMonth = payments.value.filter(p => isLastMonth(p.tanggal_pemasukkan)).reduce((s,p)=>s+Number(p.jumlah_pemasukkan),0)
  if (incomeLastMonth === 0) return '+100%'
  const percent = ((incomeThisMonth - incomeLastMonth) / incomeLastMonth) * 100
  return `${percent > 0 ? '+' : ''}${percent.toFixed(1)}%`
})

// 2. Quick Stats
const totalStudents = computed(() => students.value.length)

const unpaidThisWeek = computed(() => {
  const paidThisWeekIds = payments.value
    .filter(p => isThisWeek(p.tanggal_pemasukkan))
    .map(p => Number(p.data_student_id))
  
  return students.value.filter(s => !paidThisWeekIds.includes(Number(s.id))).length
})

// 3. Recent Ledger
const recentLedger = computed(() => {
  const allLedger = [
    ...payments.value.map(p => {
      const student = students.value.find(s => Number(s.id) === Number(p.data_student_id))
      return {
        id: `p-${p.id}`,
        type: 'income',
        title: `Class Fee - ${student ? (student.nama_siswa || student.nama_lengkap || '').split(' ')[0] : 'Unknown'}`,
        amount: Number(p.jumlah_pemasukkan),
        date: p.tanggal_pemasukkan
      }
    }),
    ...expenses.value.map(e => ({
      id: `e-${e.id}`,
      type: 'expense',
      title: e.keterangan || 'Expense',
      amount: Number(e.jumlah_pengeluaran),
      date: e.tanggal_pengeluaran
    }))
  ]
  return allLedger.sort((a, b) => new Date(b.date) - new Date(a.date)).slice(0, 3)
})

const weeklyTotalRaw = computed(() => {
  const incomeThisWeek = payments.value.filter(p => isThisWeek(p.tanggal_pemasukkan)).reduce((s,p)=>s+Number(p.jumlah_pemasukkan), 0)
  const expenseThisWeek = expenses.value.filter(e => isThisWeek(e.tanggal_pengeluaran)).reduce((s,e)=>s+Number(e.jumlah_pengeluaran), 0)
  return incomeThisWeek - expenseThisWeek
})

// 4. Financial Performance Chart
const chartData = computed(() => {
  const months = ['JAN', 'FEB', 'MAR', 'APR', 'MAY', 'JUN', 'JUL', 'AUG', 'SEP', 'OCT', 'NOV', 'DEC']
  const data = []
  
  const currentMonth = new Date().getMonth()
  for (let i = 4; i >= 0; i--) {
    let m = currentMonth - i
    let y = new Date().getFullYear()
    if (m < 0) { m += 12; y--; }
    
    const inc = payments.value.filter(p => new Date(p.tanggal_pemasukkan).getMonth() === m && new Date(p.tanggal_pemasukkan).getFullYear() === y).reduce((s,p)=>s+Number(p.jumlah_pemasukkan),0)
    const exp = expenses.value.filter(e => new Date(e.tanggal_pengeluaran).getMonth() === m && new Date(e.tanggal_pengeluaran).getFullYear() === y).reduce((s,e)=>s+Number(e.jumlah_pengeluaran),0)
    
    data.push({ month: months[m], income: inc, expense: exp })
  }
  
  const maxVal = Math.max(...data.flatMap(d => [d.income, d.expense]), 10000)
  
  return data.map(d => ({
    ...d,
    incomeHeight: `${(d.income / maxVal) * 100}%`,
    expenseHeight: `${(d.expense / maxVal) * 100}%`
  }))
})

// 5. Payment Status Matrix
const matrixData = computed(() => {
  return students.value.map(student => {
    const studentPayments = payments.value.filter(p => Number(p.data_student_id) === Number(student.id))
    const paymentThisWeek = studentPayments.find(p => isThisWeek(p.tanggal_pemasukkan))
    
    return {
      id: student.id,
      name: student.nama_siswa || student.nama_lengkap,
      avatar: (student.nama_siswa || student.nama_lengkap || 'U').substring(0, 2).toUpperCase(),
      status: paymentThisWeek ? 'PAID' : 'UNPAID',
      date: paymentThisWeek ? formatDate(paymentThisWeek.tanggal_pemasukkan) : '—',
      amount: paymentThisWeek ? formatRupiah(paymentThisWeek.jumlah_pemasukkan) : 'Rp 0'
    }
  })
})

const fetchData = async () => {
  loading.value = true;
  try {
    const [studentsRes, paymentsRes, expensesRes] = await Promise.all([
      api.get('/student').catch(() => ({ data: { Data: [] } })),
      api.get('/pembayaran').catch(() => ({ data: { Data: [] } })),
      api.get('/pengeluaran').catch(() => ({ data: { Data: [] } }))
    ]);

    students.value = studentsRes.data.Data || studentsRes.data || [];
    payments.value = paymentsRes.data.Data || paymentsRes.data || [];
    expenses.value = expensesRes.data.Data || expensesRes.data || [];
    
  } catch (error) {
    console.error("Error fetching data:", error);
    // Dummy fallback
    students.value = Array.from({length: 36}, (_, i) => ({id: i+1, nama_siswa: `Student ${i+1}`}))
    payments.value = [
      { id: 1, data_student_id: 1, tanggal_pemasukkan: new Date().toISOString(), jumlah_pemasukkan: 20000 },
      { id: 2, data_student_id: 2, tanggal_pemasukkan: new Date().toISOString(), jumlah_pemasukkan: 20000 }
    ]
    expenses.value = [
      { id: 1, tanggal_pengeluaran: new Date(new Date().setDate(new Date().getDate()-1)).toISOString(), jumlah_pengeluaran: 45000, keterangan: 'Whiteboard Markers' }
    ]
  } finally {
    loading.value = false;
  }
}

onMounted(() => {
  fetchData()
})

const getMonthWeekText = () => {
  const d = new Date()
  const month = d.toLocaleString('default', { month: 'long' })
  const week = Math.ceil(d.getDate() / 7)
  return `${month} Week ${week}`
}
</script>

<template>
  <StudentLayout>
    <div class="mb-8">
      <h1 class="text-3xl font-bold text-gray-900 tracking-tight">Class Financial Dashboard</h1>
      <p class="text-gray-500 mt-1">View the transparent financial performance of our class treasury.</p>
    </div>

    <DashboardSummaryCards 
      :currentBalance="currentBalance"
      :balanceTrend="balanceTrend"
      :totalStudents="totalStudents"
      :unpaidThisWeek="unpaidThisWeek"
    />

    <div class="grid grid-cols-1 xl:grid-cols-3 gap-6 mb-6">
      <FinancialChart 
        :chartData="chartData"
        :formatShortRupiah="formatShortRupiah"
      />

      <RecentLedger 
        :recentLedger="recentLedger"
        :formatDate="formatDate"
        :formatShortRupiah="formatShortRupiah"
        :formatRupiah="formatRupiah"
        :weeklyTotalRaw="weeklyTotalRaw"
      />
    </div>

    <PaymentMatrix 
      :matrixData="matrixData"
      :getMonthWeekText="getMonthWeekText"
      :loading="loading"
      :readonly="true"
    />
  </StudentLayout>
</template>
