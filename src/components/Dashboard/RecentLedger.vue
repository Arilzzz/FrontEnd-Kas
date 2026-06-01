<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  recentLedger: { type: Array, required: true },
  formatDate: { type: Function, required: true },
  formatShortRupiah: { type: Function, required: true },
  formatRupiah: { type: Function, required: true },
  weeklyTotalRaw: { type: Number, required: true },
  isExpanded: { type: Boolean, default: false },
  filterMonth: { type: String, default: '' },
  filterWeek: { type: String, default: '' }
})

const emit = defineEmits(['toggle', 'update:filterMonth', 'update:filterWeek'])

// Slice internally so TransitionGroup can animate entering items
const displayedItems = computed(() =>
  props.isExpanded ? props.recentLedger : props.recentLedger.slice(0, 4)
)
</script>

<template>
  <div class="bg-slate-50/50 rounded-3xl p-5 border border-gray-100 shadow-sm flex flex-col h-full">
    <div class="flex justify-between items-center mb-4">
      <h2 class="text-lg font-bold text-gray-900">Aktivitas Terbaru</h2>
      <button @click="emit('toggle')"
        class="text-xs font-bold text-blue-600 hover:text-blue-800 transition-colors px-3 py-1.5 bg-blue-50 hover:bg-blue-100 rounded-lg">
        {{ isExpanded ? '↑ Tutup' : 'Lihat Semua →' }}
      </button>
    </div>

    <!-- Filters -->
    <div class="grid grid-cols-2 gap-2 mb-4">
      <div class="relative">
        <select :value="filterMonth" @change="emit('update:filterMonth', $event.target.value)"
          class="w-full bg-white border border-gray-200 rounded-xl pl-3 pr-8 py-2.5 text-xs font-bold text-gray-700 focus:outline-none focus:ring-2 focus:ring-blue-100 appearance-none cursor-pointer transition-all">
          <option value="">Semua Bulan</option>
          <option value="0">Januari</option><option value="1">Februari</option><option value="2">Maret</option>
          <option value="3">April</option><option value="4">Mei</option><option value="5">Juni</option>
          <option value="6">Juli</option><option value="7">Agustus</option><option value="8">September</option>
          <option value="9">Oktober</option><option value="10">November</option><option value="11">Desember</option>
        </select>
        <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center px-3 text-gray-400">
          <svg class="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M19 9l-7 7-7-7"/></svg>
        </div>
      </div>
      <div class="relative">
        <select :value="filterWeek" @change="emit('update:filterWeek', $event.target.value)"
          class="w-full bg-white border border-gray-200 rounded-xl pl-3 pr-8 py-2.5 text-xs font-bold text-gray-700 focus:outline-none focus:ring-2 focus:ring-blue-100 appearance-none cursor-pointer transition-all">
          <option value="">Semua Minggu</option>
          <option value="Minggu 1">Minggu 1</option><option value="Minggu 2">Minggu 2</option>
          <option value="Minggu 3">Minggu 3</option><option value="Minggu 4">Minggu 4</option>
        </select>
        <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center px-3 text-gray-400">
          <svg class="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M19 9l-7 7-7-7"/></svg>
        </div>
      </div>
    </div>

    <!-- List with TransitionGroup for smooth animation -->
    <div class="flex-1" :class="isExpanded ? 'overflow-y-auto pr-1' : ''">
      <div v-if="recentLedger.length === 0" class="text-center text-sm text-gray-500 py-8 font-medium">
        Tidak ada aktivitas pada periode ini
      </div>

      <TransitionGroup name="ledger-slide" tag="div" class="flex flex-col gap-3">
        <div v-for="item in displayedItems" :key="item.id"
          class="bg-white p-4 rounded-2xl border border-gray-100 flex items-center justify-between shadow-sm gap-3">
          <div class="flex items-center gap-3 min-w-0">
            <div class="w-9 h-9 rounded-xl flex items-center justify-center flex-shrink-0"
              :class="item.type === 'income' ? 'bg-green-50 text-green-600 border border-green-100' : 'bg-red-50 text-red-600 border border-red-100'">
              <svg v-if="item.type === 'income'" class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 14l-7 7m0 0l-7-7m7 7V3"/></svg>
              <svg v-else class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 10l7-7m0 0l7 7m-7-7v18"/></svg>
            </div>
            <div class="min-w-0">
              <p class="text-sm font-bold text-gray-900 truncate">{{ item.title }}</p>
              <p class="text-xs font-medium text-gray-500 mt-0.5">{{ formatDate(item.date) }}</p>
            </div>
          </div>
          <span class="text-sm font-black whitespace-nowrap flex-shrink-0"
            :class="item.type === 'income' ? 'text-green-600' : 'text-red-600'">
            {{ item.type === 'income' ? '+' : '-' }}{{ formatShortRupiah(item.amount) }}
          </span>
        </div>
      </TransitionGroup>

      <!-- Show more hint when collapsed -->
      <div v-if="!isExpanded && recentLedger.length > 4" class="text-center mt-3">
        <button @click="emit('toggle')" class="text-xs text-gray-400 hover:text-blue-600 font-bold transition-colors">
          +{{ recentLedger.length - 4 }} transaksi lainnya →
        </button>
      </div>
    </div>

    <div class="mt-5 pt-4 border-t border-gray-100">
      <div class="w-full bg-blue-600 rounded-xl py-3.5 px-4 font-bold flex justify-between items-center shadow-sm shadow-blue-200">
        <span class="flex items-center gap-2 text-white/80 text-sm">
          <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 17v-2m3 2v-4m3 4v-6m2 10H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"/></svg>
          Total Periode
        </span>
        <span class="text-white text-base font-black">+{{ formatRupiah(weeklyTotalRaw) }}</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
.ledger-slide-enter-active {
  transition: all 0.4s cubic-bezier(0.25, 0.46, 0.45, 0.94);
}
.ledger-slide-leave-active {
  transition: all 0.25s ease-in;
  position: absolute;
  width: 100%;
}
.ledger-slide-enter-from {
  opacity: 0;
  transform: translateY(-16px) scale(0.97);
}
.ledger-slide-leave-to {
  opacity: 0;
  transform: translateY(-8px);
}
.ledger-slide-move {
  transition: transform 0.35s ease;
}
</style>
