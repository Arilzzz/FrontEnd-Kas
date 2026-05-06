<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  enrichedStudents: { type: Array, required: true },
  lunasCount: { type: Number, required: true },
  belumBayarCount: { type: Number, required: true },
  loading: { type: Boolean, required: true },
  readonly: { type: Boolean, default: false }
})

const currentPage = ref(1)
const itemsPerPage = 5

const paginatedStudents = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage
  return props.enrichedStudents.slice(start, start + itemsPerPage)
})

const totalPages = computed(() => Math.ceil(props.enrichedStudents.length / itemsPerPage))

const nextPage = () => {
  if (currentPage.value < totalPages.value) currentPage.value++
}

const prevPage = () => {
  if (currentPage.value > 1) currentPage.value--
}
</script>

<template>
  <div class="bg-white rounded-2xl border border-gray-100 shadow-sm overflow-hidden mb-8">
    <div class="px-6 py-5 border-b border-gray-100 flex justify-between items-center">
      <h2 class="text-lg font-bold text-gray-900">Master Registry</h2>
      <div class="flex items-center gap-4 text-xs font-medium text-gray-500">
        <div class="flex items-center gap-1.5"><span class="w-2 h-2 rounded-full bg-green-500"></span> {{ lunasCount }} Lunas</div>
        <div class="flex items-center gap-1.5"><span class="w-2 h-2 rounded-full bg-red-500"></span> {{ belumBayarCount }} Belum Bayar</div>
      </div>
    </div>
    <div class="overflow-x-auto">
      <table class="min-w-full divide-y divide-gray-100">
        <thead class="bg-gray-50/50">
          <tr>
            <th scope="col" class="px-6 py-4 text-left text-xs font-bold text-gray-500 uppercase tracking-wider">Student Details</th>
            <th scope="col" class="px-6 py-4 text-left text-xs font-bold text-gray-500 uppercase tracking-wider">Status</th>
            <th scope="col" class="px-6 py-4 text-left text-xs font-bold text-gray-500 uppercase tracking-wider">Progress (Month)</th>
            <th scope="col" class="px-6 py-4 text-left text-xs font-bold text-gray-500 uppercase tracking-wider">Total Contributed</th>
            <th v-if="!readonly" scope="col" class="px-6 py-4 text-right text-xs font-bold text-gray-500 uppercase tracking-wider">Actions</th>
          </tr>
        </thead>
        <tbody class="bg-white divide-y divide-gray-50">
          <tr v-if="loading"><td :colspan="readonly ? 4 : 5" class="px-6 py-8 text-center text-gray-500 font-medium">Loading records...</td></tr>
          <tr v-else-if="enrichedStudents.length === 0"><td :colspan="readonly ? 4 : 5" class="px-6 py-8 text-center text-gray-500 font-medium">No students found.</td></tr>
          <tr v-for="student in paginatedStudents" :key="student.id" class="hover:bg-gray-50/50 transition-colors group">
            <td class="px-6 py-4 whitespace-nowrap">
              <div class="flex items-center">
                <div class="flex-shrink-0 h-10 w-10 bg-blue-100 text-blue-600 font-bold rounded-full flex items-center justify-center border border-blue-200">
                  {{ student.avatar }}
                </div>
                <div class="ml-4">
                  <div class="text-sm font-bold text-gray-900">{{ student.name }}</div>
                  <div class="text-xs font-medium text-gray-500 mt-0.5">NISN: {{ student.nisn }}</div>
                </div>
              </div>
            </td>
            <td class="px-6 py-4 whitespace-nowrap">
              <span class="px-3 py-1 inline-flex text-xs leading-5 font-bold rounded-full gap-1 items-center" :class="student.status === 'Lunas' ? 'bg-green-100 text-green-700' : 'bg-red-100 text-red-700'">
                <span class="w-1.5 h-1.5 rounded-full" :class="student.status === 'Lunas' ? 'bg-green-500' : 'bg-red-500'"></span>
                {{ student.status }}
              </span>
            </td>
            <td class="px-6 py-4 whitespace-nowrap">
              <div class="flex gap-1.5">
                <div v-for="week in ['w1', 'w2', 'w3', 'w4']" :key="week" class="w-6 h-6 rounded text-[10px] font-bold flex items-center justify-center uppercase" :class="student.progress.includes(week) ? 'bg-green-600 text-white shadow-sm' : 'bg-gray-100 text-gray-400'">
                  {{ week }}
                </div>
              </div>
            </td>
            <td class="px-6 py-4 whitespace-nowrap">
              <div class="text-sm font-bold text-gray-900">{{ student.total }}</div>
              <div class="text-xs mt-0.5 font-medium" :class="student.isDue ? 'text-red-500' : 'text-gray-500'">
                {{ student.lastPaid }}
              </div>
            </td>
            <td v-if="!readonly" class="px-6 py-4 whitespace-nowrap text-right text-sm font-medium">
              <button class="text-gray-400 hover:text-blue-600 transition-colors p-2 rounded-lg hover:bg-blue-50">
                <svg class="h-5 w-5" fill="currentColor" viewBox="0 0 20 20"><path d="M6 10a2 2 0 11-4 0 2 2 0 014 0zM12 10a2 2 0 11-4 0 2 2 0 014 0zM16 12a2 2 0 100-4 2 2 0 000 4z"/></svg>
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    
    <!-- Pagination -->
    <div v-if="!loading && enrichedStudents.length > 0" class="px-6 py-4 border-t border-gray-100 flex items-center justify-between bg-gray-50/30">
      <div class="text-xs font-bold text-gray-500">
        Showing {{ (currentPage - 1) * itemsPerPage + 1 }} to {{ Math.min(currentPage * itemsPerPage, enrichedStudents.length) }} of {{ enrichedStudents.length }} entries
      </div>
      <div class="flex items-center gap-2">
        <button @click="prevPage" :disabled="currentPage === 1" class="p-2 rounded-lg border border-gray-200 text-gray-500 hover:bg-gray-100 disabled:opacity-50 disabled:cursor-not-allowed transition-colors">
          <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7"/></svg>
        </button>
        <div class="flex items-center gap-1">
          <button v-for="p in totalPages" :key="p" @click="currentPage = p"
            :class="currentPage === p ? 'bg-blue-600 text-white shadow-sm' : 'text-gray-600 hover:bg-gray-100'"
            class="w-8 h-8 rounded-lg text-xs font-bold flex items-center justify-center transition-all">
            {{ p }}
          </button>
        </div>
        <button @click="nextPage" :disabled="currentPage === totalPages" class="p-2 rounded-lg border border-gray-200 text-gray-500 hover:bg-gray-100 disabled:opacity-50 disabled:cursor-not-allowed transition-colors">
          <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"/></svg>
        </button>
      </div>
    </div>
  </div>
</template>
