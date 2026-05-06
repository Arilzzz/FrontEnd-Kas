<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import AdminLayout from '../../components/AdminLayout.vue'
import api from '../../services/api'

const router = useRouter()
const studentForm = ref({ nis: '', nama_siswa: '' })
const isSubmitting = ref(false)
const submitSuccess = ref(false)
const recentlyAdded = ref([
  { id: 1, name: 'Sarah J. Miller', nis: '2024042', time: 'Added 2 mins ago' },
  { id: 2, name: 'David Chen', nis: '2024041', time: 'Added 1 hour ago' }
])

const goBack = () => {
  router.push('/admin/datastudent')
}

const registerStudent = async () => {
  if (!studentForm.value.nis || !studentForm.value.nama_siswa) return
  
  isSubmitting.value = true
  try {
    const response = await api.post('/student', studentForm.value)
    
    recentlyAdded.value.unshift({
      id: Date.now(),
      name: studentForm.value.nama_siswa,
      nis: studentForm.value.nis,
      time: 'Added just now'
    })
    
    studentForm.value.nis = ''
    studentForm.value.nama_siswa = ''
    submitSuccess.value = true
    
    setTimeout(() => {
      submitSuccess.value = false
    }, 3000)
    
  } catch (error) {
    console.error("Failed to add student:", error)
    alert("Failed to add student. Please check console.")
  } finally {
    isSubmitting.value = false
  }
}

// CSV Upload logic stub
const handleFileUpload = (event) => {
  const file = event.target.files[0]
  if (file && file.type === "text/csv") {
    alert(`File ${file.name} selected. Upload logic goes here.`)
    // Normally we'd use FormData and api.post('/student/import', formData)
  } else {
    alert("Please upload a valid CSV file.")
  }
}
</script>

<template>
  <AdminLayout>
    <div class="mb-8">
      <div class="flex items-center gap-3 mb-2">
        <button @click="goBack" class="p-2 rounded-full hover:bg-gray-200 text-gray-500 hover:text-gray-900 transition-colors">
          <svg class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 19l-7-7m0 0l7-7m-7 7h18"/></svg>
        </button>
        <span class="text-sm font-bold text-blue-600 uppercase tracking-widest">Student Registry</span>
      </div>
      <div class="flex items-center justify-between">
        <h1 class="text-4xl font-black text-gray-900 tracking-tight">Add New Students</h1>
        <div class="bg-gray-100 rounded-full px-4 py-1.5 flex items-center gap-3">
          <div class="text-right">
            <p class="text-xs font-bold text-gray-900 leading-none">Class 12-A</p>
            <p class="text-[10px] font-semibold text-gray-500 mt-0.5">Total 32 Students</p>
          </div>
          <img class="w-8 h-8 rounded-full shadow-sm" src="https://ui-avatars.com/api/?name=Admin&background=0D8ABC&color=fff" alt="Admin" />
        </div>
      </div>
    </div>

    <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
      
      <!-- Left Column: Manual Registration -->
      <div class="flex flex-col gap-6">
        <div class="bg-gray-50 rounded-3xl p-8 border border-gray-100">
          <div class="flex items-center gap-3 mb-6">
            <div class="w-1 h-6 bg-blue-600 rounded-full"></div>
            <h2 class="text-xl font-bold text-gray-900">Manual Registration</h2>
          </div>
          
          <form @submit.prevent="registerStudent" class="space-y-4">
            <div class="grid grid-cols-2 gap-4">
              <div>
                <label class="block text-sm font-bold text-gray-700 mb-1.5">NIS (Student ID Number)</label>
                <input v-model="studentForm.nis" type="text" required class="w-full bg-gray-200/50 border-transparent focus:bg-white focus:border-blue-500 focus:ring-2 focus:ring-blue-200 rounded-xl px-4 py-3 text-sm font-medium transition-all" placeholder="e.g. 2024001" />
              </div>
              <div>
                <label class="block text-sm font-bold text-gray-700 mb-1.5">Nama Lengkap</label>
                <input v-model="studentForm.nama_siswa" type="text" required class="w-full bg-gray-200/50 border-transparent focus:bg-white focus:border-blue-500 focus:ring-2 focus:ring-blue-200 rounded-xl px-4 py-3 text-sm font-medium transition-all" placeholder="e.g. Alexander Graham" />
              </div>
            </div>
            <div class="flex justify-end pt-2">
              <button type="submit" :disabled="isSubmitting" class="flex items-center gap-2 bg-blue-600 hover:bg-blue-700 text-white px-6 py-3 rounded-xl font-bold shadow-sm shadow-blue-200 transition-all active:scale-95 disabled:opacity-70">
                <svg v-if="isSubmitting" class="animate-spin -ml-1 mr-2 h-5 w-5 text-white" fill="none" viewBox="0 0 24 24"><circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle><path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path></svg>
                <svg v-else class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M18 9v3m0 0v3m0-3h3m-3 0h-3m-2-5a4 4 0 11-8 0 4 4 0 018 0zM3 20a6 6 0 0112 0v1H3v-1z"/></svg>
                Register Student
              </button>
            </div>
          </form>
          <div v-if="submitSuccess" class="mt-4 p-3 bg-green-50 text-green-700 rounded-lg text-sm font-semibold flex items-center gap-2">
            <svg class="w-5 h-5 text-green-500" fill="currentColor" viewBox="0 0 20 20"><path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd"/></svg>
            Student successfully registered!
          </div>
        </div>

        <div>
          <h3 class="text-xs font-bold text-gray-500 uppercase tracking-widest mb-4">Recently Added</h3>
          <div class="space-y-3">
            <div v-for="student in recentlyAdded" :key="student.id" class="bg-white rounded-xl p-4 border border-gray-100 flex items-center justify-between shadow-sm">
              <div class="flex items-center gap-4">
                <div class="w-1 h-8 bg-green-500 rounded-full"></div>
                <div>
                  <p class="font-bold text-gray-900 text-sm">{{ student.name }}</p>
                  <p class="text-xs font-medium text-gray-500">NIS: {{ student.nis }} • {{ student.time }}</p>
                </div>
              </div>
              <div class="flex items-center gap-3">
                <span class="bg-green-100 text-green-700 text-[10px] font-bold px-2 py-1 rounded-full uppercase">Success</span>
                <button class="text-gray-400 hover:text-gray-600"><svg class="w-5 h-5" fill="currentColor" viewBox="0 0 20 20"><path d="M10 6a2 2 0 110-4 2 2 0 010 4zM10 12a2 2 0 110-4 2 2 0 010 4zM10 18a2 2 0 110-4 2 2 0 010 4z"/></svg></button>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Right Column: Bulk Import -->
      <div class="bg-slate-50/80 rounded-3xl p-8 border border-gray-100 flex flex-col h-full">
        <div class="flex items-center gap-3 mb-4">
          <div class="w-1 h-6 bg-red-600 rounded-full"></div>
          <h2 class="text-xl font-bold text-gray-900">Bulk Import</h2>
        </div>
        <p class="text-gray-600 text-sm font-medium mb-8">Have a long list of students? Skip the manual work and upload a CSV file with your student data.</p>
        
        <div class="border-2 border-dashed border-gray-300 rounded-2xl p-10 flex flex-col items-center justify-center text-center mb-6 bg-white transition-all hover:border-blue-400 hover:bg-blue-50/30 group relative">
          <input type="file" accept=".csv" @change="handleFileUpload" class="absolute inset-0 w-full h-full opacity-0 cursor-pointer" />
          <div class="w-16 h-16 bg-blue-50 text-blue-600 rounded-full flex items-center justify-center mb-4 group-hover:scale-110 transition-transform">
            <svg class="w-8 h-8" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M15 13l-3-3m0 0l-3 3m3-3v12"/></svg>
          </div>
          <h3 class="font-bold text-gray-900 text-lg mb-1">Drop your CSV here</h3>
          <p class="text-gray-500 text-sm font-medium mb-4">or click to browse from device</p>
          <span class="bg-gray-200 text-gray-600 text-[10px] font-bold px-3 py-1 rounded-full uppercase tracking-wider">Max file size: 5MB</span>
        </div>

        <button class="w-full py-3.5 bg-gray-200 hover:bg-gray-300 text-gray-800 font-bold rounded-xl transition-colors flex items-center justify-center gap-2 mb-4">
          <svg class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"/></svg>
          Download CSV Template
        </button>

        <div class="bg-blue-50 border border-blue-100 rounded-xl p-4 flex gap-3 text-sm mt-auto">
          <svg class="w-5 h-5 text-blue-500 flex-shrink-0 mt-0.5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"/></svg>
          <div>
            <p class="font-bold text-blue-900 mb-1">Pro Tip: Ensure your CSV follows the header format:</p>
            <code class="text-xs bg-white text-blue-700 px-2 py-1 rounded font-mono border border-blue-100 shadow-sm">nis, full_name, guardian_contact</code>
          </div>
        </div>

      </div>
    </div>
  </AdminLayout>
</template>
