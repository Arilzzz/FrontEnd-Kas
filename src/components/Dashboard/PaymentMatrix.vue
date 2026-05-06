<script setup>
defineProps({
  matrixData: { type: Array, required: true },
  getMonthWeekText: { type: Function, required: true },
  loading: { type: Boolean, required: true },
  readonly: { type: Boolean, default: false }
})
</script>

<template>
  <div class="bg-white rounded-3xl border border-gray-100 shadow-sm overflow-hidden mb-8">
    <div class="p-8 pb-6 flex flex-col md:flex-row justify-between md:items-center gap-4 border-b border-gray-100">
      <div>
        <h2 class="text-xl font-bold text-gray-900">Payment Status Matrix</h2>
        <p class="text-sm font-medium text-gray-500 mt-1">{{ getMonthWeekText() }} &bull; Class XII-IPA-1</p>
      </div>
      <div class="flex bg-gray-100/80 p-1 rounded-lg self-start">
        <button class="px-5 py-2 text-xs font-bold text-gray-800 bg-white rounded-md shadow-sm">This Week</button>
        <button class="px-5 py-2 text-xs font-bold text-gray-500 hover:text-gray-800 transition-colors">Last Week</button>
      </div>
    </div>
    
    <div class="overflow-x-auto">
      <table class="min-w-full divide-y divide-gray-100">
        <thead class="bg-white">
          <tr>
            <th scope="col" class="px-8 py-5 text-left text-xs font-black text-gray-900 uppercase tracking-widest">Student Name</th>
            <th scope="col" class="px-8 py-5 text-left text-xs font-black text-gray-900 uppercase tracking-widest">Status</th>
            <th scope="col" class="px-8 py-5 text-left text-xs font-black text-gray-900 uppercase tracking-widest">Payment Date</th>
            <th scope="col" class="px-8 py-5 text-left text-xs font-black text-gray-900 uppercase tracking-widest">Amount</th>
            <th v-if="!readonly" scope="col" class="px-8 py-5 text-right text-xs font-black text-gray-900 uppercase tracking-widest">Action</th>
          </tr>
        </thead>
        <tbody class="divide-y divide-gray-50">
          <tr v-if="loading"><td :colspan="readonly ? 4 : 5" class="px-8 py-8 text-center text-gray-500 font-medium">Loading...</td></tr>
          <tr v-else-if="matrixData.length === 0"><td :colspan="readonly ? 4 : 5" class="px-8 py-8 text-center text-gray-500 font-medium">No students found.</td></tr>
          <tr v-for="student in matrixData.slice(0, 5)" :key="student.id" class="hover:bg-gray-50/50 transition-colors">
            <td class="px-8 py-5 whitespace-nowrap">
              <div class="flex items-center gap-4">
                <div class="w-10 h-10 rounded-full bg-blue-50 text-blue-600 font-bold flex items-center justify-center text-sm border border-blue-100">
                  {{ student.avatar }}
                </div>
                <span class="font-bold text-sm text-gray-900">{{ student.name }}</span>
              </div>
            </td>
            <td class="px-8 py-5 whitespace-nowrap">
              <span v-if="student.status === 'PAID'" class="px-3 py-1 bg-green-100 text-green-700 text-[10px] font-black rounded-full tracking-wider uppercase">Paid</span>
              <span v-else class="px-3 py-1 bg-red-100 text-red-600 text-[10px] font-black rounded-full tracking-wider uppercase">Unpaid</span>
            </td>
            <td class="px-8 py-5 whitespace-nowrap text-sm font-medium text-gray-500">
              {{ student.date }}
            </td>
            <td class="px-8 py-5 whitespace-nowrap text-sm font-bold text-gray-900">
              {{ student.amount }}
            </td>
            <td v-if="!readonly" class="px-8 py-5 whitespace-nowrap text-right">
              <button v-if="student.status === 'UNPAID'" class="px-4 py-1.5 bg-blue-50 hover:bg-blue-100 text-blue-600 text-xs font-bold rounded-md transition-colors uppercase tracking-wider">Remind</button>
              <button v-else class="text-gray-400 hover:text-gray-600 p-1">
                <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 20 20"><path d="M10 6a2 2 0 110-4 2 2 0 010 4zM10 12a2 2 0 110-4 2 2 0 010 4zM10 18a2 2 0 110-4 2 2 0 010 4z"/></svg>
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <div class="p-6 text-center border-t border-gray-100">
      <button class="px-6 py-2 bg-white border border-gray-200 text-gray-600 text-sm font-bold rounded-lg shadow-sm hover:bg-gray-50 transition-colors">
        Load Full Student Registry
      </button>
    </div>
  </div>
</template>
