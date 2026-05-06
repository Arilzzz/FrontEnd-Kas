<script setup>
defineProps({
  currentBalance: { type: String, required: true },
  balanceTrend: { type: String, required: true },
  totalStudents: { type: Number, required: true },
  unpaidThisWeek: { type: Number, required: true }
})
</script>

<template>
  <div class="grid grid-cols-1 xl:grid-cols-3 gap-6 mb-6">
    
    <!-- Top Left: Current Balance -->
    <div class="xl:col-span-1 bg-blue-600 rounded-3xl p-8 text-white relative overflow-hidden flex flex-col justify-between h-52 shadow-lg shadow-blue-200">
      <div class="absolute -right-6 -bottom-6 opacity-20 text-white">
        <svg class="w-48 h-48" fill="currentColor" viewBox="0 0 24 24">
          <path d="M21 18v1c0 1.1-.9 2-2 2H5c-1.11 0-2-.9-2-2V5c0-1.1.89-2 2-2h14c1.1 0 2 .9 2 2v1h-9c-1.11 0-2 .9-2 2v8c0 1.1.89 2 2 2h9zm-9-2h10V8H12v8zm4-2.5c-.83 0-1.5-.67-1.5-1.5s.67-1.5 1.5-1.5 1.5.67 1.5 1.5-.67 1.5-1.5 1.5z"/>
        </svg>
      </div>
      <div class="relative z-10">
        <p class="text-xs font-bold tracking-wider text-blue-100 uppercase mb-2">Current Balance</p>
        <h2 class="text-4xl md:text-5xl font-black mb-4 tracking-tight">{{ currentBalance }}</h2>
      </div>
      <div class="flex items-center gap-3 relative z-10">
        <span class="bg-white/20 text-white text-xs font-bold px-2.5 py-1 rounded-full flex items-center gap-1">
          <svg class="w-3 h-3" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M13 7h8m0 0v8m0-8l-8 8-4-4-6 6"/></svg>
          {{ balanceTrend }}
        </span>
        <span class="text-blue-100 text-xs font-medium">vs. last month</span>
      </div>
    </div>

    <!-- Top Middle: Total Students -->
    <div class="xl:col-span-1 bg-white rounded-3xl p-8 border border-gray-100 shadow-sm flex flex-col justify-center h-52">
      <p class="text-sm font-bold text-gray-500 mb-2">Total Students</p>
      <div class="flex items-baseline gap-2 mb-6">
        <h2 class="text-5xl font-black text-gray-900">{{ totalStudents }}</h2>
        <span class="text-sm font-medium text-gray-400">Registered</span>
      </div>
      <div class="flex -space-x-3">
        <img class="w-10 h-10 rounded-full border-2 border-white bg-blue-100 object-cover" src="https://ui-avatars.com/api/?name=J&background=random" />
        <img class="w-10 h-10 rounded-full border-2 border-white bg-green-100 object-cover" src="https://ui-avatars.com/api/?name=A&background=random" />
        <img class="w-10 h-10 rounded-full border-2 border-white bg-purple-100 object-cover" src="https://ui-avatars.com/api/?name=S&background=random" />
        <div class="w-10 h-10 rounded-full border-2 border-white bg-blue-50 flex items-center justify-center text-xs font-bold text-blue-600 z-10">
          +{{ totalStudents > 3 ? totalStudents - 3 : 0 }}
        </div>
      </div>
    </div>

    <!-- Top Right: Unpaid -->
    <div class="xl:col-span-1 bg-white rounded-3xl p-8 border border-gray-100 shadow-sm border-l-4 border-l-red-500 flex flex-col justify-center h-52">
      <p class="text-sm font-bold text-gray-500 mb-2">Unpaid This Week</p>
      <div class="flex items-baseline gap-2 mb-6">
        <h2 class="text-5xl font-black text-red-600">{{ unpaidThisWeek }}</h2>
        <span class="text-sm font-medium text-gray-400">Students</span>
      </div>
      <div class="bg-red-50 text-red-600 text-xs font-bold px-3 py-2 rounded-lg inline-flex items-center gap-2 self-start">
        <svg class="w-4 h-4" fill="currentColor" viewBox="0 0 20 20"><path fill-rule="evenodd" d="M18 10a8 8 0 11-16 0 8 8 0 0116 0zm-7 4a1 1 0 11-2 0 1 1 0 012 0zm-1-9a1 1 0 00-1 1v4a1 1 0 102 0V6a1 1 0 00-1-1z" clip-rule="evenodd"/></svg>
        URGENT ACTION
      </div>
    </div>

  </div>
</template>
