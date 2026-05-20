<script setup>
import { ref, computed, onMounted } from 'vue'
import AdminLayout from '../../components/AdminLayout.vue'
import DashboardSummaryCards from '../../components/Dashboard/DashboardSummaryCards.vue'
import FinancialChart from '../../components/Dashboard/FinancialChart.vue'
import RecentLedger from '../../components/Dashboard/RecentLedger.vue'
import QuickActions from '../../components/Dashboard/QuickActions.vue'
import SiswaMenunggak from '../../components/Dashboard/SiswaMenunggak.vue'
import api from '../../services/api'

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

// Balance vs last month trend (simplified: just comparing month-over-month income for demonstration)
const balanceTrend = computed(() => {
  // If we want real balance vs last month, we need historical snapshot. 
  // Let's approximate based on Income this month vs last month
  const incomeThisMonth = payments.value.filter(p => new Date(p.tanggal_pemasukkan).getMonth() === new Date().getMonth()).reduce((s,p)=>s+Number(p.jumlah_pemasukkan),0)
  const incomeLastMonth = payments.value.filter(p => isLastMonth(p.tanggal_pemasukkan)).reduce((s,p)=>s+Number(p.jumlah_pemasukkan),0)
  if (incomeLastMonth === 0) return '+100%'
  const percent = ((incomeThisMonth - incomeLastMonth) / incomeLastMonth) * 100
  return `${percent > 0 ? '+' : ''}${percent.toFixed(1)}%`
})

// 2. Quick Stats
const totalStudents = computed(() => students.value.length)

const incomeThisMonthRaw = computed(() => {
  const currentMonth = new Date().getMonth();
  const currentYear = new Date().getFullYear();
  return payments.value
    .filter(p => {
      const d = new Date(p.tanggal_pemasukkan);
      return d.getMonth() === currentMonth && d.getFullYear() === currentYear;
    })
    .reduce((sum, p) => sum + Number(p.jumlah_pemasukkan), 0);
})
const incomeThisMonth = computed(() => formatRupiah(incomeThisMonthRaw.value))

const totalTunggakanRaw = computed(() => {
  const targetTotal = students.value.length * 4 * WEEKLY_DUES;
  const diff = targetTotal - incomeThisMonthRaw.value;
  return diff > 0 ? diff : 0;
})
const totalTunggakan = computed(() => formatRupiah(totalTunggakanRaw.value))

const studentsMenunggak = computed(() => {
  const currentMonth = new Date().getMonth();
  const currentYear = new Date().getFullYear();
  const targetThisMonth = 4 * WEEKLY_DUES;

  const data = students.value.map(student => {
    const paidThisMonth = payments.value
      .filter(p => Number(p.data_student_id) === Number(student.id) && new Date(p.tanggal_pemasukkan).getMonth() === currentMonth && new Date(p.tanggal_pemasukkan).getFullYear() === currentYear)
      .reduce((sum, p) => sum + Number(p.jumlah_pemasukkan), 0);
      
    const tunggakan = targetThisMonth - paidThisMonth;
    return {
      id: student.id,
      name: student.nama_siswa || student.nama_lengkap || 'Unknown',
      avatar: (student.nama_siswa || student.nama_lengkap || 'U').substring(0, 2).toUpperCase(),
      tunggakanRaw: tunggakan,
      tunggakan: formatRupiah(tunggakan)
    }
  }).filter(s => s.tunggakanRaw > 0);

  return data.sort((a, b) => b.tunggakanRaw - a.tunggakanRaw); // Pass all, slice in component or handle pagination there
})

// 3. Recent Ledger
const isLedgerExpanded = ref(false)
const toggleLedger = () => {
  isLedgerExpanded.value = !isLedgerExpanded.value
}

const recentLedger = computed(() => {
  const allLedger = [
    ...payments.value.map(p => {
      const student = students.value.find(s => Number(s.id) === Number(p.data_student_id))
      
      const currentMonth = new Date(p.tanggal_pemasukkan).getMonth();
      const currentYear = new Date(p.tanggal_pemasukkan).getFullYear();
      const targetThisMonth = 4 * WEEKLY_DUES;
      const paidThisMonth = payments.value
        .filter(pay => Number(pay.data_student_id) === Number(p.data_student_id) && new Date(pay.tanggal_pemasukkan).getMonth() === currentMonth && new Date(pay.tanggal_pemasukkan).getFullYear() === currentYear)
        .reduce((sum, pay) => sum + Number(pay.jumlah_pemasukkan), 0);
      const tunggakanRaw = targetThisMonth - paidThisMonth;

      return {
        id: `p-${p.id}`,
        type: 'income',
        title: `Class Fee - ${student ? (student.nama_siswa || student.nama_lengkap || '').split(' ')[0] : 'Unknown'}`,
        studentName: student ? (student.nama_siswa || student.nama_lengkap) : 'Unknown',
        amount: Number(p.jumlah_pemasukkan),
        tunggakan: tunggakanRaw > 0 ? tunggakanRaw : 0,
        date: p.tanggal_pemasukkan
      }
    }),
    ...expenses.value.map(e => ({
      id: `e-${e.id}`,
      type: 'expense',
      title: e.keterangan || 'Expense',
      studentName: '-',
      amount: Number(e.jumlah_pengeluaran),
      tunggakan: 0,
      date: e.tanggal_pengeluaran
    }))
  ]
  allLedger.sort((a, b) => new Date(b.date) - new Date(a.date))
  return isLedgerExpanded.value ? allLedger : allLedger.slice(0, 3)
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
  // Get last 5 months
  for (let i = 4; i >= 0; i--) {
    let m = currentMonth - i
    let y = new Date().getFullYear()
    if (m < 0) { m += 12; y--; }
    
    const inc = payments.value.filter(p => new Date(p.tanggal_pemasukkan).getMonth() === m && new Date(p.tanggal_pemasukkan).getFullYear() === y).reduce((s,p)=>s+Number(p.jumlah_pemasukkan),0)
    const exp = expenses.value.filter(e => new Date(e.tanggal_pengeluaran).getMonth() === m && new Date(e.tanggal_pengeluaran).getFullYear() === y).reduce((s,e)=>s+Number(e.jumlah_pengeluaran),0)
    
    data.push({ month: months[m], income: inc, expense: exp })
  }
  
  // Find max for scaling
  const maxVal = Math.max(...data.flatMap(d => [d.income, d.expense]), 10000) // min scale 10k
  
  return data.map(d => ({
    ...d,
    incomeHeight: `${(d.income / maxVal) * 100}%`,
    expenseHeight: `${(d.expense / maxVal) * 100}%`
  }))
})

// ... existing code ...
// We already replaced the computed properties above. We just need to remove matrixData and update the template.

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
    // Dummy fallback for UI testing
    students.value = Array.from({length: 10}, (_, i) => ({id: i+1, nama_siswa: `Student ${i+1}`}))
    payments.value = [
      { id: 1, data_student_id: 1, tanggal_pemasukkan: new Date().toISOString(), jumlah_pemasukkan: 20000 },
      { id: 2, data_student_id: 2, tanggal_pemasukkan: new Date().toISOString(), jumlah_pemasukkan: 20000 }
    ]
    expenses.value = [
      { id: 1, tanggal_pengeluaran: new Date(new Date().setDate(new Date().getDate()-1)).toISOString(), jumlah_pengeluaran: 45000, keterangan: 'Spidol' }
    ]
  } finally {
    loading.value = false;
  }
}

onMounted(() => {
  fetchData()
})
</script>

<template>
  <AdminLayout>
    <DashboardSummaryCards 
      :currentBalance="currentBalance"
      :balanceTrend="balanceTrend"
      :totalStudents="totalStudents"
      :incomeThisMonth="incomeThisMonth"
      :totalTunggakan="totalTunggakan"
    />

    <div class="grid grid-cols-1 xl:grid-cols-3 gap-6 mb-6">
      <div :class="isLedgerExpanded ? 'xl:col-span-1' : 'xl:col-span-2'">
        <FinancialChart 
          :chartData="chartData"
          :formatShortRupiah="formatShortRupiah"
        />
      </div>

      <div :class="isLedgerExpanded ? 'xl:col-span-2' : 'xl:col-span-1'">
        <RecentLedger 
          :recentLedger="recentLedger"
          :formatDate="formatDate"
          :formatShortRupiah="formatShortRupiah"
          :formatRupiah="formatRupiah"
          :weeklyTotalRaw="weeklyTotalRaw"
          :isExpanded="isLedgerExpanded"
          @toggle="toggleLedger"
        />
      </div>
    </div>

    <div class="grid grid-cols-1 xl:grid-cols-3 gap-6">
      <SiswaMenunggak :studentsMenunggak="studentsMenunggak" />
      <QuickActions />
    </div>
  </AdminLayout>
</template>
