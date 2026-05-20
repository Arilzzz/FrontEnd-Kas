<script setup>
import { inject, ref, onMounted } from 'vue'

const toggleSidebar = inject('toggleSidebar')
const adminName = ref('Admin Kas')
const initials = ref('A')

onMounted(() => {
  const userDataStr = localStorage.getItem('user_data')
  if (userDataStr) {
    try {
      const user = JSON.parse(userDataStr)
      if (user && user.name) {
        adminName.value = user.name
        initials.value = user.name.substring(0, 2).toUpperCase()
      }
    } catch (e) {
      console.error(e)
    }
  }
})
</script>

<template>
  <header class="bg-white border-b border-gray-100 h-20 flex items-center justify-between px-8 sticky top-0 z-10">
    <div class="flex-1 max-w-2xl flex items-center">
      <button @click="toggleSidebar" class="p-2 mr-4 text-gray-500 hover:text-blue-600 hover:bg-blue-50 rounded-lg transition-colors duration-200 focus:outline-none focus:ring-2 focus:ring-blue-100 hidden md:block">
        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16"></path>
        </svg>
      </button>
    </div>

    <div class="flex items-center gap-6 ml-6">


      <div class="flex items-center gap-3 pl-6 border-l border-gray-200">
        <div class="hidden text-right md:block">
          <p class="text-sm font-semibold text-gray-900 leading-none">{{ adminName }}</p>
          <p class="text-xs text-gray-500 mt-1">XI TJKT 1</p>
        </div>
        <button class="flex text-sm border-2 border-transparent rounded-full focus:outline-none focus:border-blue-100 transition duration-150 ease-in-out">
          <div class="h-10 w-10 rounded-full bg-gradient-to-br from-blue-500 to-purple-600 text-white flex items-center justify-center font-bold text-lg shadow-sm">
            {{ initials }}
          </div>
        </button>
      </div>
    </div>
  </header>
</template>
