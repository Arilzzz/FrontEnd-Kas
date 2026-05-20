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
const searchQuery = ref('')

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

const filteredStudents = computed(() => {
  let result = enrichedStudents.value;
  if (searchQuery.value) {
    const q = searchQuery.value.toLowerCase();
    result = result.filter(s => 
      s.name.toLowerCase().includes(q) || s.nisn.toLowerCase().includes(q)
    );
  }
  
  return [...result].sort((a, b) => {
    // Sort by status: Lunas comes first
    if (a.status === 'Lunas' && b.status !== 'Lunas') return -1;
    if (a.status !== 'Lunas' && b.status === 'Lunas') return 1;
    
    // Then sort by NISN ascending
    if (a.nisn < b.nisn) return -1;
    if (a.nisn > b.nisn) return 1;
    
    return 0;
  });
})

const lunasCount = computed(() => filteredStudents.value.filter(s => s.status === 'Lunas').length)
const belumBayarCount = computed(() => filteredStudents.value.filter(s => s.status === 'Belum Bayar').length)

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

const navigateToAddPayment = () => {
  router.push('/admin/payment')
}

const handleEdit = (id) => {
  router.push({ path: '/admin/addstudent', query: { editId: id } })
}

const handleDelete = async (id) => {
  try {
    await api.delete(`/student/${id}`)
    await fetchData()
  } catch (error) {
    console.error('Failed to delete student:', error)
    alert('Gagal menghapus siswa.')
  }
}
</script>

<template>
  <AdminLayout>
    <div class="mb-8 flex flex-col md:flex-row md:items-start md:justify-between gap-4">
      <div>
        <h1 class="text-3xl font-bold text-gray-900 tracking-tight">Kontribusi Siswa</h1>
        <p class="text-gray-500 mt-1">Kelola database kas kelas dan pantau riwayat pembayaran per siswa.</p>
      </div>
      <div class="flex items-center gap-3">
        <button type="button" @click="navigateToAddPayment" class="flex items-center gap-2 bg-blue-600 hover:bg-blue-700 text-white px-5 py-2.5 rounded-xl font-medium shadow-sm shadow-blue-200 transition-all active:scale-95">
          Transaksi Baru
        </button>
      </div>
    </div>

    <StudentKPIs 
      :collectionRate="collectionRate"
      :outstanding="outstanding"
      :totalStudents="totalStudents"
      :treasuryBalance="treasuryBalance"
    />  

    <!-- Search Bar -->
    <div class="mb-6 relative max-w-md">
      <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
        <svg class="h-5 w-5 text-gray-400 group-focus-within:text-blue-500 transition-colors" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
        </svg>
      </div>
      <input
        v-model="searchQuery"
        type="text"
        class="block w-full pl-10 pr-3 py-3 border border-gray-200 rounded-xl leading-5 bg-white placeholder-gray-400 focus:outline-none focus:bg-white focus:ring-2 focus:ring-blue-100 focus:border-blue-300 sm:text-sm transition-all duration-200 shadow-sm"
        placeholder="Cari nama siswa atau NIS..."
      />
    </div>

    <StudentRegistryTable 
      :enrichedStudents="filteredStudents"
      :lunasCount="lunasCount"
      :belumBayarCount="belumBayarCount"
      :loading="loading"
      @edit="handleEdit"
      @delete="handleDelete"
    />

    <!-- Expand Class Roster CTA -->
    <div class="border border-dashed border-gray-300 bg-gray-50/50 rounded-3xl p-8 flex flex-col items-center justify-center text-center max-w-2xl mx-auto">
      <div class="w-16 h-16 bg-gray-200 rounded-full flex items-center justify-center mb-4 text-gray-500 shadow-inner">
        <svg class="w-8 h-8" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M18 9v3m0 0v3m0-3h3m-3 0h-3m-2-5a4 4 0 11-8 0 4 4 0 018 0zM3 20a6 6 0 0112 0v1H3v-1z"/></svg>
      </div>
      <h3 class="text-xl font-bold text-gray-900 mb-2">Kelola Data Kelas</h3>
      <p class="text-gray-500 mb-6 font-medium text-sm">Semester baru dimulai? Anda dapat mengimpor siswa dari file CSV atau menambahkannya secara manual.</p>
      <div class="flex gap-4">
        <button class="px-6 py-2.5 bg-white border border-gray-200 shadow-sm rounded-xl font-semibold text-gray-700 hover:bg-gray-50 hover:border-gray-300 transition-all">Impor CSV</button>
        <button @click="navigateToAddStudent" class="px-6 py-2.5 bg-blue-600 shadow-sm shadow-blue-200 rounded-xl font-semibold text-white hover:bg-blue-700 transition-all active:scale-95">Tambah Siswa Baru</button>
      </div>
    </div>
  </AdminLayout>
</template>
