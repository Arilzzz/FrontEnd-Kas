<script setup>
import { ref } from "vue";
import { useRouter } from "vue-router";
import api from "../../services/api";

const email = ref('');
const password = ref('');
const errorMsg = ref('');
const loading = ref(false);

const router = useRouter();

async function loginAdmin() {
    errorMsg.value = '';
    if (!email.value || !password.value) {
        errorMsg.value = 'Email and password are required';
        return;
    }
    
    loading.value = true;
    try {
        const res = await api.post('/login-Admin', { 
            email: email.value,
            password: password.value
        });
        
        if (res.data && res.data.token) {
            localStorage.setItem('auth_token', res.data.token);
            localStorage.setItem('user_role', res.data.role);
            localStorage.setItem('user_data', JSON.stringify(res.data.user));
            router.push('/admin/dashboard');
        }
    } catch (err) {
        if (err.response && err.response.data && err.response.data.message) {
            errorMsg.value = err.response.data.message;
        } else {
            errorMsg.value = "Terjadi kesalahan server";
        }
        console.error(err);
    } finally {
        loading.value = false;
    }
}
</script>

<template>
  <div class="min-h-screen flex items-center justify-center bg-gray-900 bg-[url('https://www.transparenttextures.com/patterns/stardust.png')] relative overflow-hidden">
    <!-- Decorative blobs (Dark theme) -->
    <div class="absolute top-0 left-1/4 w-96 h-96 bg-blue-600/30 rounded-full mix-blend-screen filter blur-3xl opacity-50 animate-blob"></div>
    <div class="absolute top-0 right-1/4 w-96 h-96 bg-purple-600/30 rounded-full mix-blend-screen filter blur-3xl opacity-50 animate-blob animation-delay-2000"></div>
    <div class="absolute -bottom-32 left-1/2 w-96 h-96 bg-cyan-600/30 rounded-full mix-blend-screen filter blur-3xl opacity-50 animate-blob animation-delay-4000"></div>

    <div class="max-w-md w-full relative z-10 px-6">
      <div class="bg-gray-800/80 backdrop-blur-xl rounded-3xl shadow-2xl overflow-hidden border border-gray-700">
        <div class="px-8 pt-12 pb-8 text-center">
          <div class="inline-flex items-center justify-center w-16 h-16 rounded-2xl bg-gradient-to-br from-blue-500 to-purple-600 text-white shadow-lg shadow-purple-500/30 mb-6">
            <svg class="w-8 h-8" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m5.618-4.016A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016z"/></svg>
          </div>
          <h2 class="text-3xl font-black text-white mb-2">Admin Secure Login</h2>
          <p class="text-gray-400 font-medium text-sm">Authenticate to access the dashboard</p>
        </div>

        <div class="px-8 pb-12">
          <div class="space-y-5">
            <div>
              <label for="email" class="block text-sm font-bold text-gray-300 mb-2">Email Address</label>
              <div class="relative">
                <div class="absolute inset-y-0 left-0 pl-4 flex items-center pointer-events-none">
                  <svg class="h-5 w-5 text-gray-500" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 12a4 4 0 10-8 0 4 4 0 008 0zm0 0v1.5a2.5 2.5 0 005 0V12a9 9 0 10-9 9m4.5-1.206a8.959 8.959 0 01-4.5 1.207"/></svg>
                </div>
                <input id="email" type="email" v-model="email" @keyup.enter="loginAdmin"
                  class="block w-full pl-11 pr-4 py-3.5 border border-gray-600 rounded-xl text-sm font-medium text-white focus:ring-2 focus:ring-blue-500 focus:border-blue-500 bg-gray-900/50 outline-none transition-all placeholder-gray-500"
                  placeholder="admin@example.com" />
              </div>
            </div>

            <div>
              <label for="password" class="block text-sm font-bold text-gray-300 mb-2">Password</label>
              <div class="relative">
                <div class="absolute inset-y-0 left-0 pl-4 flex items-center pointer-events-none">
                  <svg class="h-5 w-5 text-gray-500" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z"/></svg>
                </div>
                <input id="password" type="password" v-model="password" @keyup.enter="loginAdmin"
                  class="block w-full pl-11 pr-4 py-3.5 border border-gray-600 rounded-xl text-sm font-medium text-white focus:ring-2 focus:ring-blue-500 focus:border-blue-500 bg-gray-900/50 outline-none transition-all placeholder-gray-500"
                  placeholder="••••••••" />
              </div>
            </div>

            <div v-if="errorMsg" class="bg-red-500/10 border border-red-500/50 text-red-400 text-sm font-medium rounded-xl p-4 flex items-center gap-3">
              <svg class="w-5 h-5 flex-shrink-0" fill="currentColor" viewBox="0 0 20 20"><path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd"/></svg>
              {{ errorMsg }}
            </div>

            <button @click="loginAdmin" :disabled="loading"
              class="w-full mt-2 flex justify-center items-center gap-2 py-3.5 px-4 border border-transparent rounded-xl shadow-lg shadow-blue-500/20 text-sm font-bold text-white bg-gradient-to-r from-blue-600 to-purple-600 hover:from-blue-500 hover:to-purple-500 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-offset-gray-900 focus:ring-blue-500 transition-all active:scale-[0.98] disabled:opacity-70 disabled:cursor-not-allowed">
              <span v-if="!loading">Authenticate</span>
              <span v-else class="flex items-center gap-2">
                <svg class="animate-spin h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"><circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle><path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path></svg>
                Processing...
              </span>
            </button>
          </div>
          
          <div class="mt-8 text-center flex flex-col gap-2 text-sm font-medium text-gray-400">
            <p>New admin? <router-link to="/admin/register" class="text-blue-400 font-bold hover:text-blue-300 transition-colors inline-block ml-1">Create Account</router-link></p>
            <router-link to="/login" class="text-gray-500 hover:text-gray-300 transition-colors inline-block mt-2">← Back to Student Portal</router-link>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
@keyframes blob {
  0% { transform: translate(0px, 0px) scale(1); }
  33% { transform: translate(30px, -50px) scale(1.1); }
  66% { transform: translate(-20px, 20px) scale(0.9); }
  100% { transform: translate(0px, 0px) scale(1); }
}
.animate-blob {
  animation: blob 7s infinite;
}
.animation-delay-2000 {
  animation-delay: 2s;
}
.animation-delay-4000 {
  animation-delay: 4s;
}
</style>
