<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import AdminLayout from '../../components/AdminLayout.vue'
import StudentPaymentProfile from '../../components/Payment/StudentPaymentProfile.vue'
import api from '../../services/api'

const router = useRouter()
const students = ref([])
const payments = ref([])
const loading = ref(true)
const isSubmitting = ref(false)

// Form State
const selectedStudentId = ref('')
const amount = ref('')
const transactionDate = ref(new Date().toISOString().split('T')[0])
const selectedWeek = ref('Week 1')
const selectedMonth = ref(new Date().toISOString().slice(0, 7)) // YYYY-MM
const showSuccess = ref(false)

const WEEKLY_DUES = 2000;

// Fetch Data
const fetchData = async () => {
  loading.value = true;
  try {
    const [studentsRes, paymentsRes] = await Promise.all([
      api.get('/student'),
      api.get('/pembayaran').catch(() => ({ data: { Data: [] } }))
    ]);
    students.value = studentsRes.data.Data || studentsRes.data || [];
    payments.value = paymentsRes.data.Data || paymentsRes.data || [];
  } catch (error) {
    console.error("Error fetching data:", error);
  } finally {
    loading.value = false;
  }
}

onMounted(() => {
  fetchData()
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
  const monthsActive = Math.max(1, currentMonth - 5) // roughly assume started in July
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
  if (date < firstMonday) return 'Week 1';
  const diffDays = Math.floor((date - firstMonday) / (1000 * 60 * 60 * 24));
  if (diffDays < 7) return 'Week 1';
  if (diffDays < 14) return 'Week 2';
  if (diffDays < 21) return 'Week 3';
  return 'Week 4';
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

const savePayment = async () => {
  if (!selectedStudentId.value || !amount.value || !transactionDate.value) return
  
  isSubmitting.value = true
  try {
    const payload = {
      data_student_id: selectedStudentId.value,
      jumlah_pemasukkan: amount.value,
      tanggal_pemasukkan: transactionDate.value
    }
    
    await api.post('/pembayaran', payload)
    
    const paymentsRes = await api.get('/pembayaran')
    payments.value = paymentsRes.data.Data || paymentsRes.data || []
    
    amount.value = ''
    showSuccess.value = true
    setTimeout(() => {
      showSuccess.value = false
    }, 3000)
    
  } catch (error) {
    console.error("Failed to save payment:", error)
    if (error.response) {
      alert("Error: " + JSON.stringify(error.response.data))
    }
  } finally {
    isSubmitting.value = false
  }
}
</script>

<template>
  <AdminLayout>
    <div class="mb-8">
      <router-link to="/admin/dashboard" class="flex items-center text-blue-600 text-sm font-medium hover:underline mb-2">
        <svg class="w-4 h-4 mr-1" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 19l-7-7m0 0l7-7m-7 7h18"/></svg>
        Back to Dashboard
      </router-link>
      <h1 class="text-4xl font-black text-gray-900 tracking-tight">Record New Payment</h1>
      <p class="text-gray-500 font-medium mt-2">Manage student contributions with academic precision.</p>
    </div>

    <div class="grid grid-cols-1 lg:grid-cols-12 gap-8">
      
      <!-- Left: Form Pencatatan -->
      <div class="lg:col-span-7 bg-gray-50 rounded-3xl p-8 border border-gray-100">
        <div class="space-y-6">
          
          <div>
            <label class="block text-sm font-bold text-gray-900 mb-2">Select Student</label>
            <div class="relative">
              <div class="absolute inset-y-0 left-0 pl-4 flex items-center pointer-events-none">
                <svg class="h-5 w-5 text-gray-400" fill="currentColor" viewBox="0 0 20 20"><path fill-rule="evenodd" d="M10 9a3 3 0 100-6 3 3 0 000 6zm-7 9a7 7 0 1114 0H3z" clip-rule="evenodd"/></svg>
              </div>
              <select v-model="selectedStudentId" class="block w-full pl-11 pr-10 py-3.5 bg-gray-200/50 border-transparent rounded-xl text-gray-900 text-sm font-medium focus:bg-white focus:ring-2 focus:ring-blue-500 outline-none appearance-none cursor-pointer transition-all">
                <option value="" disabled>Search or select a student name...</option>
                <option v-for="s in students" :key="s.id" :value="s.id">{{ s.nama_siswa || s.nama_lengkap }} (NIS: {{ s.nis }})</option>
              </select>
              <div class="absolute inset-y-0 right-0 pr-4 flex items-center pointer-events-none">
                <svg class="h-5 w-5 text-gray-400" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"/></svg>
              </div>
            </div>
          </div>

          <div class="grid grid-cols-2 gap-4">
            <div>
              <label class="block text-sm font-bold text-gray-900 mb-2">Amount</label>
              <div class="relative">
                <div class="absolute inset-y-0 left-0 pl-4 flex items-center pointer-events-none">
                  <span class="text-gray-500 font-bold">Rp</span>
                </div>
                <input v-model="amount" type="number" placeholder="0.00" class="block w-full pl-12 pr-4 py-3.5 bg-gray-200/50 border-transparent rounded-xl text-gray-900 text-sm font-medium focus:bg-white focus:ring-2 focus:ring-blue-500 outline-none transition-all" />
              </div>
            </div>
            <div>
              <label class="block text-sm font-bold text-gray-900 mb-2">Transaction Date</label>
              <input v-model="transactionDate" type="date" class="block w-full px-4 py-3.5 bg-gray-200/50 border-transparent rounded-xl text-gray-900 text-sm font-medium focus:bg-white focus:ring-2 focus:ring-blue-500 outline-none transition-all" />
            </div>
          </div>

          <div>
            <label class="block text-sm font-bold text-gray-900 mb-2">Payment Period (Week/Month)</label>
            <div class="flex gap-2 mb-3">
              <button v-for="w in ['Week 1', 'Week 2', 'Week 3', 'Week 4']" :key="w" @click="selectedWeek = w" type="button"
                :class="[selectedWeek === w ? 'bg-blue-100 text-blue-700 border-blue-200 shadow-sm' : 'bg-gray-200/50 text-gray-600 border-transparent hover:bg-gray-200']"
                class="flex-1 py-3 text-sm font-bold border rounded-xl transition-all">
                {{ w }}
              </button>
            </div>
            <input v-model="selectedMonth" type="month" class="block w-full px-4 py-3.5 bg-gray-200/50 border-transparent rounded-xl text-gray-900 text-sm font-medium focus:bg-white focus:ring-2 focus:ring-blue-500 outline-none transition-all" />
          </div>

          <div>
            <label class="block text-sm font-bold text-gray-900 mb-2">Internal Notes (Optional)</label>
            <textarea placeholder="Add details about this transaction..." rows="3" class="block w-full px-4 py-3.5 bg-gray-200/50 border-transparent rounded-xl text-gray-900 text-sm font-medium focus:bg-white focus:ring-2 focus:ring-blue-500 outline-none transition-all resize-none"></textarea>
            <p class="text-[10px] text-gray-400 mt-1">* Note: Keterangan field was removed from DB. Notes won't be saved.</p>
          </div>

          <button @click="savePayment" :disabled="!selectedStudentId || !amount || isSubmitting" 
            class="w-full mt-2 flex justify-center items-center gap-2 py-4 px-4 border border-transparent rounded-xl shadow-lg shadow-blue-500/20 text-sm font-bold text-white bg-blue-600 hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 transition-all active:scale-[0.98] disabled:opacity-70 disabled:cursor-not-allowed">
            <svg v-if="!isSubmitting" class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 7H5a2 2 0 00-2 2v9a2 2 0 002 2h14a2 2 0 002-2V9a2 2 0 00-2-2h-3m-1 4l-3 3m0 0l-3-3m3 3V4"/></svg>
            <svg v-else class="animate-spin h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"><circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle><path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path></svg>
            {{ isSubmitting ? 'Processing...' : 'Save Payment Record' }}
          </button>

          <div v-if="showSuccess" class="p-4 bg-green-50 text-green-700 rounded-xl text-sm font-bold flex items-center gap-3 mt-4">
            <svg class="w-5 h-5 text-green-500" fill="currentColor" viewBox="0 0 20 20"><path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd"/></svg>
            Payment saved successfully!
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
        />
      </div>
    </div>
  </AdminLayout>
</template>
