<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
  students: { type: Array, required: true },
  payments: { type: Array, required: true },
  weeklyDues: { type: Number, default: 2000 }
})

const filterMonth = ref(new Date().getMonth().toString())
const filterWeek = ref('')
const activeTab = ref('all')   // 'all' | 'lunas' | 'belum'
const currentPage = ref(1)
const itemsPerPage = 6

watch([filterMonth, filterWeek, activeTab], () => { currentPage.value = 1 })

const MONTH_NAMES = ['Januari','Februari','Maret','April','Mei','Juni','Juli','Agustus','September','Oktober','November','Desember']

const parseLocalDate = (dateStr) => {
  if (!dateStr) return new Date()
  const clean = dateStr.includes('T') ? dateStr.split('T')[0] : dateStr
  const parts = clean.split('-')
  if (parts.length === 3) return new Date(Number(parts[0]), Number(parts[1]) - 1, Number(parts[2]))
  return new Date(dateStr)
}

// Base = first Monday of the month of the student's first payment
// This is the KEY FIX: carry-over counts from first payment, not school year start
const getStudentBase = (studentPayments) => {
  if (!studentPayments.length) {
    // No payments: base is first Monday of the selected filter month
    const m = Number(filterMonth.value)
    const y = new Date().getFullYear()
    let d = new Date(y, m, 1)
    while (d.getDay() !== 1) d.setDate(d.getDate() + 1)
    return d
  }
  const sorted = [...studentPayments].sort((a, b) =>
    parseLocalDate(a.tanggal_pemasukkan) - parseLocalDate(b.tanggal_pemasukkan)
  )
  const first = parseLocalDate(sorted[0].tanggal_pemasukkan)
  let d = new Date(first.getFullYear(), first.getMonth(), 1)
  while (d.getDay() !== 1) d.setDate(d.getDate() + 1)
  return d
}

// Get the Nth Monday in a given month/year
const getNthMonday = (year, month, n) => {
  let d = new Date(year, month, 1)
  while (d.getDay() !== 1) d.setDate(d.getDate() + 1)
  return new Date(d.getTime() + (n - 1) * 7 * 86400000)
}

// Returns ['w1','w2',...] of weeks in month that are covered by totalPaid
const getCoveredWeeks = (totalPaid, year, month, base) => {
  const covered = Math.floor(totalPaid / props.weeklyDues)
  const result = []
  for (let n = 1; n <= 4; n++) {
    const monday = getNthMonday(year, month, n)
    const ord = Math.floor((monday.getTime() - base.getTime()) / (7 * 86400000)) + 1
    if (ord >= 1 && ord <= covered) result.push(`w${n}`)
  }
  return result
}

const formatRupiah = (num) =>
  new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 }).format(num)

const targetMonth = computed(() => Number(filterMonth.value))
const targetYear = computed(() => new Date().getFullYear())

const allStudentsEnriched = computed(() => {
  return props.students.map(student => {
    const sp = props.payments.filter(p => Number(p.data_student_id) === Number(student.id))
    const totalPaid = sp.reduce((sum, p) => sum + Number(p.jumlah_pemasukkan), 0)

    const base = getStudentBase(sp)
    const progress = getCoveredWeeks(totalPaid, targetYear.value, targetMonth.value, base)
    const status = progress.length >= 4 ? 'Lunas' : 'Belum Bayar'
    const tunggakanRaw = Math.max(0, (4 - progress.length)) * props.weeklyDues

    return {
      id: student.id,
      name: student.nama_siswa || student.nama_lengkap || 'Unknown',
      nisn: student.nis,
      avatar: (student.nama_siswa || 'U').substring(0, 2).toUpperCase(),
      progress,
      status,
      totalPaid: formatRupiah(totalPaid),
      totalPaidRaw: totalPaid,
      tunggakan: tunggakanRaw > 0 ? formatRupiah(tunggakanRaw) : null,
      tunggakanRaw,
    }
  }).sort((a, b) => b.tunggakanRaw - a.tunggakanRaw)
})

const filteredStudents = computed(() => {
  if (activeTab.value === 'lunas') return allStudentsEnriched.value.filter(s => s.status === 'Lunas')
  if (activeTab.value === 'belum') return allStudentsEnriched.value.filter(s => s.status === 'Belum Bayar')
  return allStudentsEnriched.value
})

const paginatedStudents = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage
  return filteredStudents.value.slice(start, start + itemsPerPage)
})

const totalPages = computed(() => Math.ceil(filteredStudents.value.length / itemsPerPage))
const lunasCount = computed(() => allStudentsEnriched.value.filter(s => s.status === 'Lunas').length)
const belumCount = computed(() => allStudentsEnriched.value.filter(s => s.status === 'Belum Bayar').length)

const prevPage = () => { if (currentPage.value > 1) currentPage.value-- }
const nextPage = () => { if (currentPage.value < totalPages.value) currentPage.value++ }
</script>

<template>
  <div class="bg-white rounded-3xl border border-gray-100 shadow-sm p-5 h-full flex flex-col">
    <!-- Header -->
    <div class="flex items-center justify-between mb-4">
      <div>
        <h3 class="text-lg font-bold text-gray-900">Status Iuran Siswa</h3>
        <p class="text-xs text-gray-500 mt-0.5">{{ MONTH_NAMES[targetMonth] }} · {{ students.length }} siswa</p>
      </div>
      <div class="flex items-center gap-1.5">
        <span class="bg-green-50 text-green-700 text-xs font-bold px-2.5 py-1 rounded-full flex items-center gap-1">
          <span class="w-1.5 h-1.5 rounded-full bg-green-500"></span>{{ lunasCount }} Lunas
        </span>
        <span class="bg-red-50 text-red-600 text-xs font-bold px-2.5 py-1 rounded-full flex items-center gap-1">
          <span class="w-1.5 h-1.5 rounded-full bg-red-500 animate-pulse"></span>{{ belumCount }} Tunggak
        </span>
      </div>
    </div>

    <!-- Filters row -->
    <div class="flex gap-2 mb-3">
      <div class="relative flex-1">
        <select v-model="filterMonth"
          class="w-full bg-gray-50 border border-gray-200 rounded-xl pl-3 pr-7 py-2.5 text-xs font-bold text-gray-700 focus:outline-none appearance-none cursor-pointer">
          <option v-for="(name, i) in MONTH_NAMES" :key="i" :value="String(i)">{{ name }}</option>
        </select>
        <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center px-2 text-gray-400">
          <svg class="w-3 h-3" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M19 9l-7 7-7-7"/></svg>
        </div>
      </div>
      <div class="relative flex-1">
        <select v-model="filterWeek"
          class="w-full bg-gray-50 border border-gray-200 rounded-xl pl-3 pr-7 py-2.5 text-xs font-bold text-gray-700 focus:outline-none appearance-none cursor-pointer">
          <option value="">Semua Minggu</option>
          <option value="Minggu 1">Minggu 1</option>
          <option value="Minggu 2">Minggu 2</option>
          <option value="Minggu 3">Minggu 3</option>
          <option value="Minggu 4">Minggu 4</option>
        </select>
        <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center px-2 text-gray-400">
          <svg class="w-3 h-3" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M19 9l-7 7-7-7"/></svg>
        </div>
      </div>
    </div>

    <!-- Tab switcher -->
    <div class="flex gap-1 bg-gray-100/70 rounded-xl p-1 mb-4">
      <button v-for="[key, label] in [['all','Semua'],['lunas','Lunas'],['belum','Belum Bayar']]" :key="key"
        @click="activeTab = key"
        :class="activeTab === key ? 'bg-white shadow-sm text-gray-900 font-black' : 'text-gray-500 hover:text-gray-700 font-bold'"
        class="flex-1 py-1.5 text-xs rounded-lg transition-all">
        {{ label }}
      </button>
    </div>

    <!-- Student list -->
    <div class="flex-1 overflow-y-auto min-h-0">
      <div v-if="filteredStudents.length === 0" class="text-center py-10">
        <div class="text-3xl mb-2">🎉</div>
        <p class="text-sm text-gray-500 font-medium">Tidak ada siswa di kategori ini</p>
      </div>

      <TransitionGroup name="slist" tag="div" class="space-y-2">
        <div v-for="student in paginatedStudents" :key="student.id"
          class="flex items-center gap-3 p-3 rounded-2xl border transition-all group"
          :class="student.status === 'Lunas'
            ? 'bg-green-50/40 border-green-100/70 hover:bg-green-50'
            : 'bg-gray-50/50 border-gray-100 hover:bg-red-50/20 hover:border-red-100/60'">

          <!-- Avatar -->
          <div class="w-8 h-8 rounded-full flex items-center justify-center text-xs font-black flex-shrink-0 border"
            :class="student.status === 'Lunas'
              ? 'bg-green-100 text-green-700 border-green-200'
              : 'bg-red-100 text-red-600 border-red-200'">
            {{ student.avatar }}
          </div>

          <!-- Name + NIS -->
          <div class="flex-1 min-w-0">
            <p class="text-xs font-bold text-gray-900 truncate leading-tight">{{ student.name }}</p>
            <p class="text-[10px] text-gray-400 font-medium">NIS: {{ student.nisn }}</p>
          </div>

          <!-- Week progress w1–w4 -->
          <div class="flex gap-0.5">
            <div v-for="w in ['w1','w2','w3','w4']" :key="w"
              class="w-5 h-5 rounded text-[9px] font-bold flex items-center justify-center transition-colors"
              :class="student.progress.includes(w) ? 'bg-green-600 text-white' : 'bg-gray-200 text-gray-400'">
              {{ w.slice(1) }}
            </div>
          </div>

          <!-- Total paid + status -->
          <div class="text-right flex-shrink-0 min-w-[60px]">
            <p class="text-xs font-black leading-tight"
              :class="student.status === 'Lunas' ? 'text-green-600' : 'text-red-500'">
              {{ student.totalPaid }}
            </p>
            <p class="text-[9px] font-bold mt-0.5"
              :class="student.status === 'Lunas' ? 'text-green-500' : 'text-red-400'">
              {{ student.status }}
            </p>
          </div>
        </div>
      </TransitionGroup>
    </div>

    <!-- Pagination -->
    <div v-if="totalPages > 1" class="mt-3 pt-3 border-t border-gray-100 flex items-center justify-between">
      <p class="text-xs text-gray-400 font-bold">
        {{ (currentPage - 1) * itemsPerPage + 1 }}–{{ Math.min(currentPage * itemsPerPage, filteredStudents.length) }} / {{ filteredStudents.length }}
      </p>
      <div class="flex items-center gap-2">
        <button @click="prevPage" :disabled="currentPage === 1"
          class="p-1.5 rounded-lg border border-gray-200 text-gray-500 hover:bg-gray-100 disabled:opacity-40 disabled:cursor-not-allowed transition-colors">
          <svg class="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7"/></svg>
        </button>
        <span class="text-xs font-bold text-gray-600">{{ currentPage }} / {{ totalPages }}</span>
        <button @click="nextPage" :disabled="currentPage === totalPages"
          class="p-1.5 rounded-lg border border-gray-200 text-gray-500 hover:bg-gray-100 disabled:opacity-40 disabled:cursor-not-allowed transition-colors">
          <svg class="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"/></svg>
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.slist-enter-active { transition: all 0.25s ease; }
.slist-leave-active { transition: all 0.2s ease; position: absolute; width: 100%; }
.slist-enter-from { opacity: 0; transform: translateY(-6px); }
.slist-leave-to { opacity: 0; }
.slist-move { transition: transform 0.25s ease; }
</style>
