<script setup>
import { ref } from 'vue'
import Auth from './components/Auth.vue'
import Dashboard from './components/Dashboard.vue'
import Profile from './components/Profile.vue'

const isAuthenticated = ref(false)
const currentView = ref('dashboard') // 'dashboard' | 'profile'
const userAccount = ref(null)

const handleAuthenticated = (accountData) => {
  if (accountData) {
    userAccount.value = accountData
  }
  isAuthenticated.value = true
  currentView.value = 'dashboard'
}

const openProfileView = () => {
  currentView.value = 'profile'
}

const closeProfileView = () => {
  currentView.value = 'dashboard'
}

const handleLogout = () => {
  isAuthenticated.value = false
  userAccount.value = null
  currentView.value = 'dashboard'
}
</script>

<template>
  <div id="app">
    <!-- AUTH SCREEN -->
    <Auth 
      v-if="!isAuthenticated" 
      @authenticated="handleAuthenticated" 
    />

    <!-- AUTHENTICATED APP VIEWS -->
    <template v-else>
      <Dashboard 
        v-if="currentView === 'dashboard'" 
        :userAccount="userAccount" 
        @openProfile="openProfileView" 
      />
      
      <Profile 
        v-else-if="currentView === 'profile'" 
        :userAccount="userAccount" 
        @close="closeProfileView" 
        @logout="handleLogout" 
      />
    </template>
  </div>
</template>

<style>
/* GLOBAL DARK MODE OVERRIDES */
html.dark-theme {
  background-color: #090d16;
  color: #f8fafc;
}

html.dark-theme .modal-card,
html.dark-theme .receipt-card,
html.dark-theme .user-card,
html.dark-theme .settings-section,
html.dark-theme .tx-card,
html.dark-theme .action-item {
  background-color: #131c2e !important;
  color: #f8fafc !important;
  border-color: #1e293b !important;
}

html.dark-theme .form-input,
html.dark-theme .pin-box,
html.dark-theme .account-name-box {
  background-color: #1e293b !important;
  color: #ffffff !important;
  border-color: #334155 !important;
}

html.dark-theme .tx-title,
html.dark-theme .r-value,
html.dark-theme .user-name {
  color: #f8fafc !important;
}

html.dark-theme .form-label,
html.dark-theme .modal-sub,
html.dark-theme .r-label {
  color: #94a3b8 !important;
}
</style>
