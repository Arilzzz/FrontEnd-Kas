<script setup>
import { useRouter } from 'vue-router'

const router = useRouter()

const navigateTo = (path) => {
  router.push(path)
}
</script>

<template>
  <div class="bg-white rounded-3xl p-5 border border-gray-100 shadow-sm flex flex-col h-full">
    <div class="mb-4">
      <h3 class="text-lg font-bold text-gray-900">Aksi Cepat</h3>
      <p class="text-xs text-gray-500 mt-1">Pintasan untuk mengelola kas kelas.</p>
    </div>
    <div class="flex flex-col gap-3 flex-1 justify-center">
      <button @click="navigateTo('/admin/payment')" class="flex items-center gap-3 p-3 rounded-xl bg-blue-50 text-blue-700 hover:bg-blue-600 hover:text-white transition-all group border border-blue-100 font-semibold shadow-sm text-sm">
        <div class="w-8 h-8 rounded-full bg-blue-100 text-blue-600 group-hover:bg-blue-500 group-hover:text-white flex items-center justify-center transition-colors">
          <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4"/></svg>
        </div>
        Tambah Pembayaran
      </button>
      <button @click="navigateTo('/admin/expenditure')" class="flex items-center gap-3 p-3 rounded-xl bg-red-50 text-red-700 hover:bg-red-600 hover:text-white transition-all group border border-red-100 font-semibold shadow-sm text-sm">
        <div class="w-8 h-8 rounded-full bg-red-100 text-red-600 group-hover:bg-red-500 group-hover:text-white flex items-center justify-center transition-colors">
          <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20 12H4"/></svg>
        </div>
        Tambah Pengeluaran
      </button>
      <button @click="navigateTo('/admin/addstudent')" class="flex items-center gap-3 p-3 rounded-xl bg-green-50 text-green-700 hover:bg-green-600 hover:text-white transition-all group border border-green-100 font-semibold shadow-sm text-sm">
        <div class="w-8 h-8 rounded-full bg-green-100 text-green-600 group-hover:bg-green-500 group-hover:text-white flex items-center justify-center transition-colors">
          <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M18 9v3m0 0v3m0-3h3m-3 0h-3m-2-5a4 4 0 11-8 0 4 4 0 018 0zM3 20a6 6 0 0112 0v1H3v-1z"/></svg>
        </div>
        Tambah Siswa
      </button>
    </div>
  </div>
</template>
