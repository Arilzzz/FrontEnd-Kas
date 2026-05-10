<script setup>
</script>

<template>
  <header class="bg-white border-b border-gray-100 h-20 flex items-center justify-between px-8 sticky top-0 z-10">
    <div class="flex-1 max-w-2xl">
      <!-- Removed search bar per user request -->
    </div>

    <div class="flex items-center gap-6 ml-6">
      <button class="relative p-2 text-gray-400 hover:text-gray-500 hover:bg-gray-50 rounded-full transition-all duration-200 focus:outline-none focus:ring-2 focus:ring-blue-100">
        <span class="absolute top-1.5 right-1.5 block h-2.5 w-2.5 rounded-full bg-red-500 ring-2 ring-white"></span>
        <svg class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 17h5l-1.405-1.405A2.032 2.032 0 0118 14.158V11a6.002 6.002 0 00-4-5.659V5a2 2 0 10-4 0v.341C7.67 6.165 6 8.388 6 11v3.159c0 .538-.214 1.055-.595 1.436L4 17h5m6 0v1a3 3 0 11-6 0v-1m6 0H9" />
        </svg>
      </button>

      <div class="flex items-center gap-3 pl-6 border-l border-gray-200">
        <div class="hidden text-right md:block">
          <p class="text-sm font-semibold text-gray-900 leading-none">Admin Kas</p>
          <p class="text-xs text-gray-500 mt-1">Class 12-A</p>
        </div>
        <button class="flex text-sm border-2 border-transparent rounded-full focus:outline-none focus:border-blue-100 transition duration-150 ease-in-out">
          <img class="h-10 w-10 rounded-full object-cover shadow-sm" src="https://ui-avatars.com/api/?name=Admin+Kas&background=0D8ABC&color=fff" alt="Admin avatar" />
        </button>
      </div>
    </div>
  </header>
</template>
