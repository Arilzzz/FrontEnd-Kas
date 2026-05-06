<script setup>
defineProps({
  recentLedger: { type: Array, required: true },
  formatDate: { type: Function, required: true },
  formatShortRupiah: { type: Function, required: true },
  formatRupiah: { type: Function, required: true },
  weeklyTotalRaw: { type: Number, required: true }
})
</script>

<template>
  <div class="xl:col-span-1 bg-slate-50/50 rounded-3xl p-8 border border-gray-100 shadow-sm flex flex-col">
    <div class="flex justify-between items-center mb-6">
      <h2 class="text-lg font-bold text-gray-900">Recent Ledger</h2>
      <a href="#" class="text-xs font-bold text-blue-600 hover:text-blue-800">View All</a>
    </div>

    <div class="flex flex-col gap-4 flex-1">
      <div v-if="recentLedger.length === 0" class="text-center text-sm text-gray-500 py-4">No recent transactions</div>
      <div v-for="item in recentLedger" :key="item.id" class="bg-white p-4 rounded-2xl border border-gray-100 flex items-center justify-between shadow-sm">
        <div class="flex items-center gap-3">
          <div class="w-10 h-10 rounded-xl flex items-center justify-center flex-shrink-0" :class="item.type === 'income' ? 'bg-green-50 text-green-600 border border-green-100' : 'bg-red-50 text-red-600 border border-red-100'">
            <!-- Income Icon -->
            <svg v-if="item.type === 'income'" class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 14l-7 7m0 0l-7-7m7 7V3"/></svg>
            <!-- Expense Icon -->
            <svg v-else class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 3h2l.4 2M7 13h10l4-8H5.4M7 13L5.4 5M7 13l-2.293 2.293c-.63.63-.184 1.707.707 1.707H17m0 0a2 2 0 100 4 2 2 0 000-4zm-8 2a2 2 0 11-4 0 2 2 0 014 0z"/></svg>
          </div>
          <div>
            <p class="text-sm font-bold text-gray-900 line-clamp-1" :title="item.title">{{ item.title }}</p>
            <p class="text-xs font-medium text-gray-500 mt-0.5">{{ formatDate(item.date) }}</p>
          </div>
        </div>
        <span class="text-sm font-black whitespace-nowrap" :class="item.type === 'income' ? 'text-green-600' : 'text-red-600'">
          {{ item.type === 'income' ? '+' : '-' }}{{ formatShortRupiah(item.amount) }}
        </span>
      </div>
    </div>

    <div class="mt-6">
      <button class="w-full bg-blue-600 hover:bg-blue-700 text-white rounded-xl py-3.5 px-4 font-bold flex justify-between items-center transition-colors shadow-sm shadow-blue-200">
        <span class="flex items-center gap-2">
          <svg class="w-5 h-5 opacity-70" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 17v-2m3 2v-4m3 4v-6m2 10H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"/></svg>
          Weekly Total
        </span>
        <span class="text-lg">+{{ formatRupiah(weeklyTotalRaw) }}</span>
      </button>
    </div>
  </div>
</template>
