<script setup>
defineProps({
  selectedStudent: Object,
  totalPaid: Number,
  outstanding: Number,
  formatRupiah: Function,
  recentHistory: Array
})
</script>

<template>
  <div class="space-y-6">
    <!-- Student Profile -->
    <div class="bg-white rounded-3xl p-6 border-l-4 border-l-blue-600 border border-t-gray-100 border-r-gray-100 border-b-gray-100 shadow-sm relative overflow-hidden">
      <div v-if="!selectedStudent" class="absolute inset-0 bg-white/60 backdrop-blur-sm z-10 flex items-center justify-center">
        <p class="text-sm font-bold text-gray-400">Select a student first</p>
      </div>
      <div class="flex items-center gap-4 mb-6">
        <div class="w-14 h-14 bg-blue-100 text-blue-700 rounded-full flex items-center justify-center text-xl font-bold">
          {{ selectedStudent ? (selectedStudent.nama_siswa || 'U').substring(0,2).toUpperCase() : '?' }}
        </div>
        <div>
          <h3 class="text-lg font-bold text-gray-900">{{ selectedStudent ? selectedStudent.nama_siswa : 'Student Name' }}</h3>
          <p class="text-xs font-bold text-blue-600 uppercase tracking-wider">STUDENT ID: {{ selectedStudent ? selectedStudent.nis : '-----' }}</p>
        </div>
      </div>
      <div class="flex justify-between items-center bg-gray-50 rounded-xl p-3 border border-gray-100">
        <span class="text-sm font-medium text-gray-500">Status</span>
        <span class="bg-green-300/30 text-green-800 text-xs font-bold px-3 py-1 rounded-full">ACTIVE MEMBER</span>
      </div>
    </div>

    <!-- Financial Summary -->
    <div class="grid grid-cols-2 gap-4">
      <div class="bg-white rounded-3xl p-6 border border-gray-100 shadow-sm relative overflow-hidden">
        <div v-if="!selectedStudent" class="absolute inset-0 bg-white/60 backdrop-blur-sm z-10"></div>
        <p class="text-[10px] font-bold text-gray-500 uppercase tracking-widest mb-1">TOTAL PAID</p>
        <h4 class="text-2xl font-black text-green-600">{{ formatRupiah(totalPaid) }}</h4>
      </div>
      <div class="bg-white rounded-3xl p-6 border border-gray-100 shadow-sm relative overflow-hidden">
        <div v-if="!selectedStudent" class="absolute inset-0 bg-white/60 backdrop-blur-sm z-10"></div>
        <p class="text-[10px] font-bold text-gray-500 uppercase tracking-widest mb-1">OUTSTANDING</p>
        <h4 class="text-2xl font-black text-red-600">{{ outstanding > 0 ? formatRupiah(outstanding) : 'Rp 0' }}</h4>
      </div>
    </div>

    <!-- Recent History -->
    <div class="bg-white rounded-3xl p-6 border border-gray-100 shadow-sm relative overflow-hidden">
      <div v-if="!selectedStudent" class="absolute inset-0 bg-white/60 backdrop-blur-sm z-10"></div>
      <div class="flex justify-between items-center mb-6">
        <h3 class="font-bold text-gray-900 text-sm">Recent History</h3>
        <a href="#" class="text-xs font-bold text-blue-600 hover:underline">View Full</a>
      </div>

      <div class="space-y-4">
        <div v-if="recentHistory.length === 0" class="text-center py-4 text-sm font-medium text-gray-400">
          No recent payments.
        </div>
        <div v-for="item in recentHistory" :key="item.id" class="flex justify-between items-center">
          <div class="flex gap-3">
            <div class="w-1 h-10 bg-green-600 rounded-full"></div>
            <div>
              <p class="text-sm font-bold text-gray-900">{{ item.weekLabel }}</p>
              <p class="text-[10px] font-semibold text-gray-500 mt-0.5">{{ item.dateFormatted }} • Cash</p>
            </div>
          </div>
          <span class="text-sm font-bold text-gray-900">{{ item.amount }}</span>
        </div>

        <!-- Missing example if outstanding is high -->
        <div v-if="outstanding > 0" class="flex justify-between items-center opacity-80 pt-2">
          <div class="flex gap-3">
            <div class="w-1 h-10 bg-red-200 rounded-full"></div>
            <div>
              <p class="text-sm font-bold text-gray-900 line-through text-gray-400">Past Week</p>
              <p class="text-[10px] font-bold text-gray-400 mt-0.5 italic">Unpaid</p>
            </div>
          </div>
          <span class="text-sm font-bold text-red-600">Missing</span>
        </div>
      </div>
    </div>

    <!-- Pro Tip -->
    <div class="bg-blue-50 border border-blue-100 rounded-2xl p-5 flex gap-3">
      <div class="w-6 h-6 bg-blue-600 rounded-full flex items-center justify-center text-white font-bold text-xs flex-shrink-0">i</div>
      <p class="text-xs font-medium text-gray-600 leading-relaxed">
        <span class="font-bold text-gray-900">Pro-Tip:</span> Saving this payment will automatically notify the student via their registered school portal.
      </p>
    </div>
  </div>
</template>
