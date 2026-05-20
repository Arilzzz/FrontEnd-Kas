<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  studentsMenunggak: { type: Array, required: true },
})

const currentPage = ref(1)
const itemsPerPage = 4

const paginatedStudents = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage
  return props.studentsMenunggak.slice(start, start + itemsPerPage)
})

const totalPages = computed(() => Math.ceil(props.studentsMenunggak.length / itemsPerPage))

const nextPage = () => {
  if (currentPage.value < totalPages.value) currentPage.value++
}

const prevPage = () => {
  if (currentPage.value > 1) currentPage.value--
}
</script>

<template>
  <div class="bg-white rounded-3xl border border-gray-100 shadow-sm p-5 xl:col-span-2">
    <div class="flex items-center justify-between mb-6">
      <div>
        <h3 class="text-lg font-bold text-gray-900">Siswa Menunggak</h3>
        <p class="text-xs text-gray-500 mt-1">Daftar siswa dengan tunggakan tertinggi bulan ini.</p>
      </div>
      <div class="bg-red-50 text-red-600 text-xs font-bold px-3 py-1.5 rounded-lg flex items-center gap-1.5">
        <span class="w-1.5 h-1.5 rounded-full bg-red-600 animate-pulse"></span>
        {{ studentsMenunggak.length }} Siswa
      </div>
    </div>

    <div class="space-y-4">
      <div v-if="studentsMenunggak.length === 0" class="text-center py-6 text-sm text-gray-500 font-medium">
        Tidak ada siswa yang menunggak. Bagus!
      </div>
      <div 
        v-for="student in paginatedStudents" 
        :key="student.id"
        class="flex items-center justify-between p-3 rounded-2xl bg-gray-50/50 hover:bg-gray-50 border border-gray-100/50 transition-colors"
      >
        <div class="flex items-center gap-3">
          <div class="w-8 h-8 rounded-full bg-red-100 text-red-600 font-bold flex items-center justify-center text-xs border border-red-200">
            {{ student.avatar }}
          </div>
          <div>
            <h4 class="text-sm font-bold text-gray-900">{{ student.name }}</h4>
            <p class="text-xs font-medium text-red-500 mt-0.5">Tunggakan: {{ student.tunggakan }}</p>
          </div>
        </div>
      </div>
    </div>

    <!-- Pagination -->
    <div v-if="studentsMenunggak.length > 5" class="mt-6 flex items-center justify-between border-t border-gray-100 pt-4">
      <div class="text-xs font-bold text-gray-400">
        {{ (currentPage - 1) * itemsPerPage + 1 }}-{{ Math.min(currentPage * itemsPerPage, studentsMenunggak.length) }} dari {{ studentsMenunggak.length }}
      </div>
      <div class="flex items-center gap-2">
        <button @click="prevPage" :disabled="currentPage === 1" class="p-1.5 rounded-lg border border-gray-200 text-gray-500 hover:bg-gray-100 disabled:opacity-50 disabled:cursor-not-allowed transition-colors">
          <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7"/></svg>
        </button>
        <button @click="nextPage" :disabled="currentPage === totalPages" class="p-1.5 rounded-lg border border-gray-200 text-gray-500 hover:bg-gray-100 disabled:opacity-50 disabled:cursor-not-allowed transition-colors">
          <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"/></svg>
        </button>
      </div>
    </div>
  </div>
</template>
