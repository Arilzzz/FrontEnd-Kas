<script setup>
import StudentSidebar from './StudentSidebar.vue'
import Topbar from './Topbar.vue'
</script>

<template>
  <div class="flex h-screen bg-gray-50/30 overflow-hidden font-sans">
    <StudentSidebar />
    <div class="flex-1 flex flex-col overflow-hidden">
      <Topbar />
      <main class="flex-1 overflow-x-hidden overflow-y-auto bg-[#fafafa]">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 md:px-8 py-8">
          <slot></slot>
        </div>
      </main>
    </div>
  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
body {
  font-family: 'Inter', sans-serif;
}
</style>
