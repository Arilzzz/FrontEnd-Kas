<script setup>
import { ref, computed, onMounted } from 'vue'
import StudentLayout from '../components/StudentLayout.vue'
import DashboardSummaryCards from '../components/Dashboard/DashboardSummaryCards.vue'
import FinancialChart from '../components/Dashboard/FinancialChart.vue'
import RecentLedger from '../components/Dashboard/RecentLedger.vue'
import PaymentMatrix from '../components/Dashboard/PaymentMatrix.vue'
import ClassInfo from '../components/Dashboard/ClassInfo.vue'
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

const filterMonth = ref('')
const filterWeek = ref('')
const isLedgerExpanded = ref(false)
const toggleLedger = () => {
  isLedgerExpanded.value = !isLedgerExpanded.value
}

const parseLocalDate = (dateString) => {
  if (!dateString) return new Date();
  if (dateString.includes('T')) {
    return new Date(dateString);
  }
  const parts = dateString.split('-');
  if (parts.length === 3) {
    return new Date(parts[0], parts[1] - 1, parts[2]);
  }
  return new Date(dateString);
}

const formatDate = (dateString) => {
  const d = parseLocalDate(dateString);
  if (isToday(d)) return 'Hari Ini';
  if (isYesterday(d)) return 'Kemarin';
  return d.toLocaleDateString("id-ID", {
    day: "numeric",
    month: "long",
    year: "numeric",
  });
};

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

const totalExpenseThisMonthRaw = computed(() => {
  const currentMonth = new Date().getMonth();
  const currentYear = new Date().getFullYear();
  return expenses.value
    .filter(e => {
      const d = new Date(e.tanggal_pengeluaran);
      return d.getMonth() === currentMonth && d.getFullYear() === currentYear;
    })
    .reduce((sum, e) => sum + Number(e.jumlah_pengeluaran), 0);
})
const totalExpenseThisMonth = computed(() => formatRupiah(totalExpenseThisMonthRaw.value))

// Collection rate for ClassInfo
const collectionRate = computed(() => {
  if (totalStudents.value === 0) return '0%';
  const targetTotal = totalStudents.value * 4 * WEEKLY_DUES;
  if (targetTotal === 0) return '100%';
  const rate = Math.round((incomeThisMonthRaw.value / targetTotal) * 100);
  return `${rate > 100 ? 100 : rate}%`;
})

// 3. Recent Ledger
const recentLedger = computed(() => {
  let allLedger = [
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

  // Apply filters
  if (filterMonth.value !== "") {
    allLedger = allLedger.filter((item) => {
      return parseLocalDate(item.date).getMonth() === Number(filterMonth.value);
    });
  }
  if (filterWeek.value !== "") {
    allLedger = allLedger.filter((item) => {
      const weekMap = {
        'w1': 'Minggu 1',
        'w2': 'Minggu 2',
        'w3': 'Minggu 3',
        'w4': 'Minggu 4'
      }
      const itemWeek = getWeekOfMonth(item.date);
      return weekMap[itemWeek] === filterWeek.value;
    });
  }

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

const loggedInUser = ref(null)

// Personal Contribution status (e.g. Weeks 1, 2, 3, 4 of the current month)
const studentPaymentsFiltered = computed(() => {
  if (!loggedInUser.value) return []
  return payments.value.filter(p => Number(p.data_student_id) === Number(loggedInUser.value.id))
})

const myWeeksPaid = computed(() => {
  const currentMonth = new Date().getMonth()
  const currentYear = new Date().getFullYear()
  
  const thisMonthPayments = studentPaymentsFiltered.value.filter(p => {
    const d = new Date(p.tanggal_pemasukkan)
    return d.getMonth() === currentMonth && d.getFullYear() === currentYear
  })
  
  const paidWeeks = []
  thisMonthPayments.forEach(p => {
    const week = getWeekOfMonth(p.tanggal_pemasukkan)
    if (!paidWeeks.includes(week)) {
      paidWeeks.push(week)
    }
  })
  
  return paidWeeks
})

const myTotalPaidFormatted = computed(() => {
  const total = studentPaymentsFiltered.value.reduce((sum, p) => sum + Number(p.jumlah_pemasukkan), 0)
  return formatRupiah(total)
})

const myOutstandingFormatted = computed(() => {
  const target = 4 * WEEKLY_DUES
  const currentMonth = new Date().getMonth()
  const currentYear = new Date().getFullYear()
  
  const paidThisMonth = studentPaymentsFiltered.value
    .filter(p => {
      const d = new Date(p.tanggal_pemasukkan)
      return d.getMonth() === currentMonth && d.getFullYear() === currentYear
    })
    .reduce((sum, p) => sum + Number(p.jumlah_pemasukkan), 0)
    
  const diff = target - paidThisMonth
  return formatRupiah(diff > 0 ? diff : 0)
})

onMounted(() => {
  fetchData()
  const data = localStorage.getItem('user_data')
  if (data) {
    loggedInUser.value = JSON.parse(data)
  }
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
      <h1 class="text-3xl font-bold text-gray-900 tracking-tight">Dashboard Keuangan Kelas</h1>
      <p class="text-gray-500 mt-1">Pantau pemasukan, pengeluaran, dan saldo kas kelas secara transparan dan akurat.</p>
    </div>    <DashboardSummaryCards 
      :currentBalance="currentBalance"
      :balanceTrend="balanceTrend"
      :totalStudents="totalStudents"
      :incomeThisMonth="incomeThisMonth"
      :totalTunggakan="totalTunggakan"
      :totalExpense="totalExpenseThisMonth"
    />

    <div class="grid grid-cols-1 xl:grid-cols-3 gap-6 mb-6">
      <!-- Row 1 Left: Financial Chart -->
      <div class="xl:col-span-2">
        <FinancialChart 
          :chartData="chartData"
          :formatShortRupiah="formatShortRupiah"
        />
      </div>

      <!-- Row 1 Right: Personal Student Payment Status Card -->
      <div class="xl:col-span-1">
        <div v-if="loggedInUser" class="bg-white rounded-3xl p-6 border border-gray-100 shadow-sm flex flex-col justify-between h-full min-h-[350px]">
          <div class="mb-4">
            <div class="flex items-center gap-2.5">
              <span class="text-xl">👋</span>
              <div>
                <h3 class="text-sm font-black text-gray-900 leading-none">Status Kas Kamu</h3>
                <p class="text-[11px] font-semibold text-gray-500 mt-1">{{ loggedInUser.nama_siswa || loggedInUser.nama_lengkap }} (NIS: {{ loggedInUser.nis }})</p>
              </div>
            </div>
          </div>
          
          <!-- Weekly Indicators -->
          <div class="bg-gray-50 rounded-2xl p-4 mb-4 border border-gray-100/50">
            <p class="text-[10px] font-bold text-gray-500 mb-3 uppercase tracking-wider text-center">Progres Kas Bulan Ini</p>
            <div class="flex justify-between items-center px-2">
              <div v-for="(weekName, weekKey) in { 'w1': 'Minggu 1', 'w2': 'Minggu 2', 'w3': 'Minggu 3', 'w4': 'Minggu 4' }" :key="weekKey" class="flex flex-col items-center gap-1.5">
                <div class="w-8 h-8 rounded-full flex items-center justify-center text-[10px] font-bold transition-all shadow-sm border"
                     :class="myWeeksPaid.includes(weekKey) ? 'bg-green-600 text-white border-green-700 shadow-green-100' : 'bg-red-50 text-red-600 border-red-100 shadow-red-50'">
                  <svg v-if="myWeeksPaid.includes(weekKey)" class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M5 13l4 4L19 7"/></svg>
                  <span v-else>⚠️</span>
                </div>
                <span class="text-[9px] font-black text-gray-500 uppercase tracking-widest">{{ weekName.split(' ')[1] }}</span>
              </div>
            </div>
          </div>
          
          <div class="grid grid-cols-2 gap-3 mb-4">
            <div class="bg-blue-50/50 border border-blue-100/30 rounded-xl p-3 text-center">
              <p class="text-[9px] font-black text-blue-600 uppercase tracking-wider mb-0.5">Total Setor</p>
              <p class="text-sm font-black text-blue-800">{{ myTotalPaidFormatted }}</p>
            </div>
            <div class="bg-amber-50/50 border border-amber-100/30 rounded-xl p-3 text-center">
              <p class="text-[9px] font-black text-amber-600 uppercase tracking-wider mb-0.5">Tunggakan Kamu</p>
              <p class="text-sm font-black" :class="myOutstandingFormatted !== 'Rp 0' ? 'text-red-600' : 'text-green-600'">{{ myOutstandingFormatted }}</p>
            </div>
          </div>
          
          <!-- Instructions -->
          <div class="bg-blue-50/60 rounded-2xl p-4 border border-blue-100 flex gap-3 text-[11px] leading-relaxed">
            <svg class="w-5 h-5 text-blue-600 flex-shrink-0 mt-0.5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"/></svg>
            <div>
              <p class="font-bold text-blue-900 mb-1">Cara Pembayaran Kas:</p>
              <p class="text-blue-700 font-medium">Silakan setor kas secara langsung ke **Bendahara Kelas** (Rp 2.000 / minggu), atau transfer ke rekening kelas dan konfirmasikan pembayaran Anda.</p>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="grid grid-cols-1 xl:grid-cols-3 gap-6">
      <!-- Row 2 Left: Recent Ledger -->
      <div :class="isLedgerExpanded ? 'xl:col-span-3' : 'xl:col-span-2'">
        <RecentLedger 
          :recentLedger="recentLedger"
          :formatDate="formatDate"
          :formatShortRupiah="formatShortRupiah"
          :formatRupiah="formatRupiah"
          :weeklyTotalRaw="weeklyTotalRaw"
          :isExpanded="isLedgerExpanded"
          v-model:filterMonth="filterMonth"
          v-model:filterWeek="filterWeek"
          @toggle="toggleLedger"
        />
      </div>

      <!-- Row 2 Right: Class Info -->
      <div v-if="!isLedgerExpanded" class="xl:col-span-1">
        <ClassInfo
          :totalStudents="totalStudents"
          :collectionRate="collectionRate"
          :unpaidThisWeek="unpaidThisWeek"
          :weeklyDues="WEEKLY_DUES"
        />
      </div>
    </div>
  </StudentLayout>
</template>
