<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import AdminLayout from '../../components/AdminLayout.vue'
import StudentPaymentProfile from '../../components/Payment/StudentPaymentProfile.vue'
import api from '../../services/api'

const router = useRouter()
const route = useRoute()

// State
const loading = ref(true)
const isSubmitting = ref(false)
const students = ref([])
const payments = ref([])
const expenses = ref([])

const WEEKLY_DUES = 2000;

// Form State
const paymentId = ref(route.query.id || null)
const selectedStudentId = ref('')
const amount = ref('')
const transactionDate = ref('')
const showSuccess = ref(false)

const fetchData = async () => {
  loading.value = true
  try {
    const [studentsRes, paymentsRes, expensesRes] = await Promise.all([
      api.get('/student'),
      api.get('/pembayaran').catch(() => ({ data: { Data: [] } })),
      api.get('/pengeluaran').catch(() => ({ data: { Data: [] } }))
    ])
    
    students.value = studentsRes.data.Data || studentsRes.data || []
    payments.value = paymentsRes.data.Data || paymentsRes.data || []
    expenses.value = expensesRes.data.Data || expensesRes.data || []

    if (paymentId.value) {
      loadPaymentToForm(paymentId.value)
    }
  } catch (error) {
    console.error("Error fetching data:", error)
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  fetchData()
})

const loadPaymentToForm = (id) => {
  const p = payments.value.find(pay => Number(pay.id) === Number(id))
  if (p) {
    paymentId.value = p.id
    selectedStudentId.value = p.data_student_id
    amount.value = p.jumlah_pemasukkan
    transactionDate.value = p.tanggal_pemasukkan ? p.tanggal_pemasukkan.split('T')[0] : ''
  }
}

const treasuryBalance = computed(() => {
  const totalIn = payments.value.reduce((sum, p) => sum + Number(p.jumlah_pemasukkan), 0)
  const totalOut = expenses.value.reduce((sum, e) => sum + Number(e.jumlah_pengeluaran), 0)
  return totalIn - totalOut
})

// Computed Properties for selected student
const selectedStudent = computed(() => {
  if (!selectedStudentId.value) return null
  return students.value.find(s => Number(s.id) === Number(selectedStudentId.value))
})

const studentPayments = computed(() => {
  if (!selectedStudent.value) return []
  return payments.value.filter(p => Number(p.data_student_id) === Number(selectedStudent.value.id))
    .sort((a, b) => new Date(b.tanggal_pemasukkan) - new Date(a.tanggal_pemasukkan))
})

const totalPaid = computed(() => {
  return studentPayments.value.reduce((sum, p) => sum + Number(p.jumlah_pemasukkan), 0)
})

const outstanding = computed(() => {
  const currentMonth = new Date().getMonth()
  const monthsActive = Math.max(1, currentMonth - 5)
  const target = monthsActive * 4 * WEEKLY_DUES
  const diff = target - totalPaid.value
  return diff > 0 ? diff : 0
})

const formatRupiah = (num) => {
  if (num >= 1000) return `Rp ${(num / 1000).toFixed(0)}k`
  return `Rp ${num}`
}

const getWeekOfMonth = (dateString) => {
  const date = new Date(dateString);
  const year = date.getFullYear();
  const month = date.getMonth();
  let firstMonday = new Date(year, month, 1);
  while (firstMonday.getDay() !== 1) {
    firstMonday.setDate(firstMonday.getDate() + 1);
  }
  if (date < firstMonday) return 'Minggu 1';
  const diffDays = Math.floor((date - firstMonday) / (1000 * 60 * 60 * 24));
  if (diffDays < 7) return 'Minggu 1';
  if (diffDays < 14) return 'Minggu 2';
  if (diffDays < 21) return 'Minggu 3';
  return 'Minggu 4';
}

const recentHistory = computed(() => {
  return studentPayments.value.slice(0, 3).map(p => {
    const d = new Date(p.tanggal_pemasukkan)
    return {
      id: p.id,
      dateFormatted: d.toLocaleDateString('en-GB', { day: 'numeric', month: 'short', year: 'numeric' }),
      weekLabel: `${d.toLocaleString('en-US', { month: 'short' })} ${getWeekOfMonth(p.tanggal_pemasukkan)}`,
      amount: formatRupiah(p.jumlah_pemasukkan)
    }
  })
})

const submitPayment = async () => {
  if (!paymentId.value || !selectedStudentId.value || !amount.value || !transactionDate.value) return
  
  isSubmitting.value = true
  try {
    const payload = {
      data_student_id: selectedStudentId.value,
      jumlah_pemasukkan: Math.round(Number(amount.value)),
      tanggal_pemasukkan: transactionDate.value
    }
    
    await api.put(`/pembayaran/${paymentId.value}`, payload)
    
    const paymentsRes = await api.get('/pembayaran')
    payments.value = paymentsRes.data.Data || paymentsRes.data || []
    
    paymentId.value = null
    amount.value = ''
    showSuccess.value = true
    setTimeout(() => { showSuccess.value = false }, 3000)
  } catch (error) {
    console.error("Failed to update payment:", error)
    if (error.response) {
      alert("Error: " + JSON.stringify(error.response.data))
    }
  } finally {
    isSubmitting.value = false
  }
}

const editPayment = (id) => {
  loadPaymentToForm(id)
}

const deletePayment = async (id) => {
  if (!confirm('Hapus pembayaran ini?')) return
  try {
    await api.delete(`/pembayaran/${id}`)
    const paymentsRes = await api.get('/pembayaran')
    payments.value = paymentsRes.data.Data || paymentsRes.data || []
    if (paymentId.value === id) {
      paymentId.value = null
      amount.value = ''
    }
  } catch (error) {
    console.error('Failed to delete payment:', error)
    alert('Gagal menghapus pembayaran.')
  }
}
</script>

<template>
  <AdminLayout>
    <div class="mb-8 flex flex-col md:flex-row md:items-end justify-between gap-4">
      <div>
        <router-link to="/admin/dashboard" class="flex items-center text-blue-600 text-sm font-medium hover:underline mb-2">
          <svg class="w-4 h-4 mr-1" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 19l-7-7m0 0l7-7m-7 7h18"/></svg>
          Kembali ke Dashboard
        </router-link>
        <h1 class="text-4xl font-black text-gray-900 tracking-tight">Edit Pembayaran</h1>
        <p class="text-gray-500 font-medium mt-2 mb-6">Perbarui catatan pembayaran iuran siswa.</p>

        <!-- Mode Toggle -->
        <div class="flex bg-gray-200 rounded-full p-1 max-w-fit relative overflow-hidden">
          <div class="absolute inset-y-1 left-1 w-[calc(50%-4px)] bg-white rounded-full shadow-sm transition-transform duration-300 ease-in-out translate-x-[100%]"></div>
          <button @click="router.push('/admin/payment')" class="relative z-10 px-6 py-2 rounded-full text-sm font-bold text-gray-500 hover:text-gray-700 transition-colors w-32">
            Catat Baru
          </button>
          <button class="relative z-10 px-6 py-2 rounded-full text-sm font-bold text-gray-900 transition-colors w-32 cursor-default">
            Edit Data
          </button>
        </div>
      </div>
      
      <!-- Treasury Balance Card -->
      <div class="bg-white rounded-2xl px-6 py-4 border border-gray-100 shadow-sm flex items-center gap-6">
        <div>
          <p class="text-[10px] font-bold text-gray-400 uppercase tracking-widest mb-1">TOTAL SALDO KAS</p>
          <h3 class="text-2xl font-black text-green-600">{{ formatRupiah(treasuryBalance) }}</h3>
        </div>
        <svg class="w-8 h-8 text-green-500" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8c-1.657 0-3 .895-3 2s1.343 2 3 2 3 .895 3 2-1.343 2-3 2m0-8c1.11 0 2.08.402 2.599 1M12 8V7m0 1v8m0 0v1m0-1c-1.11 0-2.08-.402-2.599-1M21 12a9 9 0 11-18 0 9 9 0 0118 0z"/></svg>
      </div>
    </div>

    <div v-if="loading" class="text-center py-12 text-gray-500 font-medium">Memuat data...</div>
    <div v-else class="grid grid-cols-1 lg:grid-cols-12 gap-8">
      
      <!-- Left: Form Pencatatan -->
      <div class="lg:col-span-7 bg-gray-50 rounded-3xl p-8 border border-gray-100">
        <div class="space-y-6">
          
          <div>
            <label class="block text-sm font-bold text-gray-900 mb-2">Pilih Siswa</label>
            <div class="relative">
              <div class="absolute inset-y-0 left-0 pl-4 flex items-center pointer-events-none">
                <svg class="h-5 w-5 text-gray-400" fill="currentColor" viewBox="0 0 20 20"><path fill-rule="evenodd" d="M10 9a3 3 0 100-6 3 3 0 000 6zm-7 9a7 7 0 1114 0H3z" clip-rule="evenodd"/></svg>
              </div>
              <select v-model="selectedStudentId" @change="paymentId = null; amount = ''" class="block w-full pl-11 pr-10 py-3.5 bg-gray-200/50 border-transparent rounded-xl text-gray-900 text-sm font-medium focus:bg-white focus:ring-2 focus:ring-blue-500 outline-none appearance-none cursor-pointer transition-all">
                <option value="" disabled>Pilih nama siswa...</option>
                <option v-for="s in students" :key="s.id" :value="s.id">{{ s.nama_siswa || s.nama_lengkap }} (NIS: {{ s.nis }})</option>
              </select>
              <div class="absolute inset-y-0 right-0 pr-4 flex items-center pointer-events-none">
                <svg class="h-5 w-5 text-gray-400" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"/></svg>
              </div>
            </div>
            <p v-if="selectedStudentId && !paymentId" class="text-xs text-blue-600 mt-2 font-medium bg-blue-50 p-2 rounded-lg inline-block">
              Pilih transaksi di panel profil (sebelah kanan) untuk mengedit.
            </p>
          </div>

          <div v-if="paymentId" class="grid grid-cols-2 gap-4">
            <div>
              <label class="block text-sm font-bold text-gray-900 mb-2">Jumlah</label>
              <div class="relative">
                <div class="absolute inset-y-0 left-0 pl-4 flex items-center pointer-events-none">
                  <span class="text-gray-500 font-bold">Rp</span>
                </div>
                <input v-model="amount" type="number" @wheel.prevent placeholder="0.00" class="block w-full pl-12 pr-4 py-3.5 bg-gray-200/50 border-transparent rounded-xl text-gray-900 text-sm font-medium focus:bg-white focus:ring-2 focus:ring-blue-500 outline-none transition-all" />
              </div>
            </div>
            <div>
              <label class="block text-sm font-bold text-gray-900 mb-2">Tanggal Transaksi</label>
              <input v-model="transactionDate" type="date" class="block w-full px-4 py-3.5 bg-gray-200/50 border-transparent rounded-xl text-gray-900 text-sm font-medium focus:bg-white focus:ring-2 focus:ring-blue-500 outline-none transition-all" />
            </div>
          </div>

          <button v-if="paymentId" @click="submitPayment" :disabled="!selectedStudentId || !amount || isSubmitting" 
            class="w-full mt-2 flex justify-center items-center gap-2 py-4 px-4 border border-transparent rounded-xl shadow-lg shadow-blue-500/20 text-sm font-bold text-white bg-blue-600 hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 transition-all active:scale-[0.98] disabled:opacity-70 disabled:cursor-not-allowed">
            <svg v-if="!isSubmitting" class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 7H5a2 2 0 00-2 2v9a2 2 0 002 2h14a2 2 0 002-2V9a2 2 0 00-2-2h-3m-1 4l-3 3m0 0l-3-3m3 3V4"/></svg>
            <svg v-else class="animate-spin h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"><circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle><path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path></svg>
            {{ isSubmitting ? 'Memproses...' : 'Simpan Perubahan' }}
          </button>

          <div v-if="showSuccess" class="p-4 bg-green-50 text-green-700 rounded-xl text-sm font-bold flex items-center gap-3 mt-4">
            <svg class="w-5 h-5 text-green-500" fill="currentColor" viewBox="0 0 20 20"><path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd"/></svg>
            Pembayaran berhasil diperbarui!
          </div>

        </div>
      </div>

      <!-- Right: Dynamic Cards -->
      <div class="lg:col-span-5">
          <StudentPaymentProfile 
            :selectedStudent="selectedStudent"
            :totalPaid="totalPaid"
            :outstanding="outstanding"
            :formatRupiah="formatRupiah"
            :recentHistory="recentHistory"
            @edit="editPayment"
            @delete="deletePayment"
          />
      </div>
    </div>
  </AdminLayout>
</template>
