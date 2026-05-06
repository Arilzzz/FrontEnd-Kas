<script setup>
import { ref, onMounted, computed } from 'vue'
import AdminLayout from '../../components/AdminLayout.vue'
import StudentKPIs from '../../components/DataStudent/StudentKPIs.vue'
import StudentRegistryTable from '../../components/DataStudent/StudentRegistryTable.vue'
import api from '../../services/api'
import { useRouter } from 'vue-router'

const router = useRouter()
const students = ref([])
const payments = ref([])
const expenses = ref([])
const loading = ref(true)

const WEEKLY_DUES = 2000; // Rp 2000 per week

const totalStudents = computed(() => students.value.length)

// Helper to format currency
const formatRupiah = (number) => {
  return new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 }).format(number);
}

// Calculate Treasury Balance
const treasuryBalanceRaw = computed(() => {
  const totalIn = payments.value.reduce((sum, p) => sum + Number(p.jumlah_pemasukkan), 0)
  const totalOut = expenses.value.reduce((sum, e) => sum + Number(e.jumlah_pengeluaran), 0)
  return totalIn - totalOut
})
const treasuryBalance = computed(() => formatRupiah(treasuryBalanceRaw.value))

// Determine which week of the month a date belongs to based on the First Monday
const getWeekOfMonth = (dateString) => {
  const date = new Date(dateString);
  const year = date.getFullYear();
  const month = date.getMonth();
  
  // Find first Monday
  let firstMonday = new Date(year, month, 1);
  while (firstMonday.getDay() !== 1) { // 1 = Monday
    firstMonday.setDate(firstMonday.getDate() + 1);
  }
  
  if (date < firstMonday) return 'w1';
  
  const diffDays = Math.floor((date - firstMonday) / (1000 * 60 * 60 * 24));
  if (diffDays < 7) return 'w1';
  if (diffDays < 14) return 'w2';
  if (diffDays < 21) return 'w3';
  return 'w4';
}

// Map students with their payment data
const enrichedStudents = computed(() => {
  const currentMonth = new Date().getMonth();
  const currentYear = new Date().getFullYear();

  return students.value.map(student => {
    // Filter payments for this student
    const studentPayments = payments.value.filter(p => Number(p.data_student_id) === Number(student.id))
    
    const totalPaid = studentPayments.reduce((sum, p) => sum + Number(p.jumlah_pemasukkan), 0)
    
    // Filter payments for CURRENT month
    const currentMonthPayments = studentPayments.filter(p => {
      const d = new Date(p.tanggal_pemasukkan);
      return d.getMonth() === currentMonth && d.getFullYear() === currentYear;
    });

    const progress = [];
    currentMonthPayments.forEach(p => {
      const week = getWeekOfMonth(p.tanggal_pemasukkan);
      if (!progress.includes(week)) progress.push(week);
    });

    // Assume 4 weeks per month target
    const targetThisMonth = 4 * WEEKLY_DUES;
    const paidThisMonth = currentMonthPayments.reduce((sum, p) => sum + Number(p.jumlah_pemasukkan), 0);
    const status = paidThisMonth >= targetThisMonth ? 'Lunas' : 'Belum Bayar';

    // Find last payment date
    let lastPaid = '-';
    if (studentPayments.length > 0) {
      const sorted = [...studentPayments].sort((a, b) => new Date(b.tanggal_pemasukkan) - new Date(a.tanggal_pemasukkan));
      const d = new Date(sorted[0].tanggal_pemasukkan);
      lastPaid = d.toLocaleDateString('en-US', { month: 'short', day: 'numeric' });
    }

    const isDue = paidThisMonth < targetThisMonth;
    const dueAmount = targetThisMonth - paidThisMonth;

    return {
      id: student.id,
      nisn: student.nis,
      name: student.nama_siswa || student.nama_lengkap || 'Unknown',
      status: status,
      progress: progress,
      total: formatRupiah(totalPaid),
      lastPaid: isDue ? `Due: ${formatRupiah(dueAmount)}` : lastPaid,
      isDue: isDue,
      avatar: (student.nama_siswa || 'U').substring(0, 2).toUpperCase()
    }
  })
})

const lunasCount = computed(() => enrichedStudents.value.filter(s => s.status === 'Lunas').length)
const belumBayarCount = computed(() => enrichedStudents.value.filter(s => s.status === 'Belum Bayar').length)

// Collection Rate and Outstanding for Current Month
const collectionRate = computed(() => {
  if (totalStudents.value === 0) return '0%';
  const targetTotal = totalStudents.value * 4 * WEEKLY_DUES;
  
  const currentMonth = new Date().getMonth();
  const currentYear = new Date().getFullYear();
  
  const paidThisMonth = payments.value
    .filter(p => {
      const d = new Date(p.tanggal_pemasukkan);
      return d.getMonth() === currentMonth && d.getFullYear() === currentYear;
    })
    .reduce((sum, p) => sum + Number(p.jumlah_pemasukkan), 0);
    
  if (targetTotal === 0) return '100%';
  const rate = Math.round((paidThisMonth / targetTotal) * 100);
  return `${rate > 100 ? 100 : rate}%`;
})

const outstanding = computed(() => {
  const targetTotal = totalStudents.value * 4 * WEEKLY_DUES;
  const currentMonth = new Date().getMonth();
  const currentYear = new Date().getFullYear();
  
  const paidThisMonth = payments.value
    .filter(p => {
      const d = new Date(p.tanggal_pemasukkan);
      return d.getMonth() === currentMonth && d.getFullYear() === currentYear;
    })
    .reduce((sum, p) => sum + Number(p.jumlah_pemasukkan), 0);
    
  const diff = targetTotal - paidThisMonth;
  return diff > 0 ? formatRupiah(diff) : 'Rp 0';
})

const fetchData = async () => {
  loading.value = true;
  try {
    const [studentsRes, paymentsRes, expensesRes] = await Promise.all([
      api.get('/student'),
      api.get('/pembayaran').catch(() => ({ data: { Data: [] } })), // Catch if endpoints don't exist yet
      api.get('/pengeluaran').catch(() => ({ data: { Data: [] } }))
    ]);

    students.value = studentsRes.data.Data || studentsRes.data || [];
    payments.value = paymentsRes.data.Data || paymentsRes.data || [];
    expenses.value = expensesRes.data.Data || expensesRes.data || [];
    
  } catch (error) {
    console.error("Error fetching data:", error);
    // Setup some dummy data if complete failure (e.g., backend not running)
    if (students.value.length === 0) {
      students.value = [
        { id: 1, nama_siswa: 'Jane Smith', nis: '202100142' },
        { id: 2, nama_siswa: 'Ahmad Maulana', nis: '202100145' }
      ]
      payments.value = [
        { id: 1, data_student_id: 1, tanggal_pemasukkan: new Date().toISOString(), jumlah_pemasukkan: 8000 }
      ]
    }
  } finally {
    loading.value = false;
  }
}

onMounted(() => {
  fetchData()
})

const navigateToAddStudent = () => {
  router.push('/admin/addstudent')
}
</script>

<template>
  <AdminLayout>
    <div class="mb-8 flex flex-col md:flex-row md:items-start md:justify-between gap-4">
      <div>
        <h1 class="text-3xl font-bold text-gray-900 tracking-tight">Student Contributions</h1>
        <p class="text-gray-500 mt-1">Manage class ledger database and track payment history per student.</p>
      </div>
      <div class="flex items-center gap-3">
        <div class="flex bg-white rounded-lg border border-gray-200 p-1">
          <button class="px-4 py-1.5 text-sm font-medium bg-gray-50 text-gray-800 rounded-md shadow-sm border border-gray-100">Table View</button>
          <button class="px-4 py-1.5 text-sm font-medium text-gray-500 hover:text-gray-700 transition-colors">Analytics</button>
        </div>
        <button class="flex items-center gap-2 bg-blue-600 hover:bg-blue-700 text-white px-5 py-2.5 rounded-xl font-medium shadow-sm shadow-blue-200 transition-all active:scale-95">
          <svg class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4"/></svg>
          New Transaction
        </button>
      </div>
    </div>

    <StudentKPIs 
      :collectionRate="collectionRate"
      :outstanding="outstanding"
      :totalStudents="totalStudents"
      :treasuryBalance="treasuryBalance"
    />

    <StudentRegistryTable 
      :enrichedStudents="enrichedStudents"
      :lunasCount="lunasCount"
      :belumBayarCount="belumBayarCount"
      :loading="loading"
    />

    <!-- Expand Class Roster CTA -->
    <div class="border border-dashed border-gray-300 bg-gray-50/50 rounded-3xl p-8 flex flex-col items-center justify-center text-center max-w-2xl mx-auto">
      <div class="w-16 h-16 bg-gray-200 rounded-full flex items-center justify-center mb-4 text-gray-500 shadow-inner">
        <svg class="w-8 h-8" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M18 9v3m0 0v3m0-3h3m-3 0h-3m-2-5a4 4 0 11-8 0 4 4 0 018 0zM3 20a6 6 0 0112 0v1H3v-1z"/></svg>
      </div>
      <h3 class="text-xl font-bold text-gray-900 mb-2">Expand Class Roster</h3>
      <p class="text-gray-500 mb-6 font-medium text-sm">New semester starting? You can import students from a CSV file or add them manually to the ledger.</p>
      <div class="flex gap-4">
        <button class="px-6 py-2.5 bg-white border border-gray-200 shadow-sm rounded-xl font-semibold text-gray-700 hover:bg-gray-50 hover:border-gray-300 transition-all">Import CSV</button>
        <button @click="navigateToAddStudent" class="px-6 py-2.5 bg-blue-600 shadow-sm shadow-blue-200 rounded-xl font-semibold text-white hover:bg-blue-700 transition-all active:scale-95">Add New Student</button>
      </div>
    </div>
  </AdminLayout>
</template>
