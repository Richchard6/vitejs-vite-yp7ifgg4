<script setup>
import { ref } from 'vue'

const props = defineProps({
  userAccount: {
    type: Object,
    default: () => ({
      accountNumber: '8012345678',
      bankName: 'Wema Bank (RPAY)',
      accountName: 'GODWIN O.'
    })
  }
})

const emit = defineEmits(['close', 'logout'])

const biometricEnabled = ref(true)
const notificationsEnabled = ref(true)

const handleLogout = () => {
  if (confirm('Are you sure you want to log out?')) {
    emit('logout')
  }
}
</script>

<template>
  <div class="profile-wrapper">
    <!-- Header -->
    <div class="profile-header">
      <button @click="emit('close')" class="back-btn">← Back</button>
      <h2>Profile & Settings</h2>
      <div style="width: 48px;"></div>
    </div>

    <!-- User Info Card -->
    <div class="user-card">
      <div class="profile-avatar">G</div>
      <h3 class="user-name">{{ props.userAccount.accountName }}</h3>
      <span class="user-email">godwin@example.com</span>
      <div class="kyc-badge">✓ Tier 2 Verified (BVN/NIN)</div>
    </div>

    <!-- Account Details Section -->
    <div class="settings-section">
      <h4 class="section-title">Account Information</h4>
      <div class="info-row">
        <span>Virtual Account</span>
        <strong>{{ props.userAccount.accountNumber }}</strong>
      </div>
      <div class="info-row">
        <span>Bank Provider</span>
        <strong>{{ props.userAccount.bankName }}</strong>
      </div>
    </div>

    <!-- Security Preferences -->
    <div class="settings-section">
      <h4 class="section-title">Security & Preferences</h4>
      
      <div class="toggle-row">
        <div>
          <strong>Biometric Authentication</strong>
          <p>Use Fingerprint or Face ID for transactions</p>
        </div>
        <input type="checkbox" v-model="biometricEnabled" class="toggle-switch" />
      </div>

      <div class="toggle-row">
        <div>
          <strong>Push Notifications</strong>
          <p>Get instant alerts for transactions</p>
        </div>
        <input type="checkbox" v-model="notificationsEnabled" class="toggle-switch" />
      </div>

      <button class="settings-btn">🔒 Change 4-Digit Transaction PIN</button>
    </div>

    <!-- Logout Button -->
    <button @click="handleLogout" class="logout-btn">
      Log Out
    </button>
  </div>
</template>

<style scoped>
.profile-wrapper { max-width: 420px; margin: 0 auto; background: #f8fafc; min-height: 100vh; padding: 20px 16px; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; box-sizing: border-box; }
.profile-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
.profile-header h2 { font-size: 18px; font-weight: 800; color: #0f172a; margin: 0; }
.back-btn { background: none; border: none; font-size: 14px; font-weight: 700; color: #2563eb; cursor: pointer; }

.user-card { background: #ffffff; border-radius: 20px; padding: 20px; text-align: center; box-shadow: 0 4px 12px rgba(0,0,0,0.03); margin-bottom: 20px; }
.profile-avatar { width: 64px; height: 64px; background: #2563eb; color: #ffffff; font-size: 26px; font-weight: 800; border-radius: 50%; display: flex; align-items: center; justify-content: center; margin: 0 auto 12px auto; }
.user-name { font-size: 18px; font-weight: 800; color: #0f172a; margin: 0; }
.user-email { font-size: 12px; color: #64748b; display: block; margin-top: 2px; }
.kyc-badge { display: inline-block; background: #dcfce7; color: #166534; font-size: 11px; font-weight: 700; padding: 4px 12px; border-radius: 12px; margin-top: 10px; }

.settings-section { background: #ffffff; border-radius: 20px; padding: 16px; margin-bottom: 16px; box-shadow: 0 4px 12px rgba(0,0,0,0.03); }
.section-title { font-size: 12px; font-weight: 800; color: #94a3b8; text-transform: uppercase; letter-spacing: 0.5px; margin: 0 0 12px 0; }
.info-row { display: flex; justify-content: space-between; font-size: 13px; padding: 8px 0; border-bottom: 1px solid #f1f5f9; color: #334155; }
.info-row:last-child { border-bottom: none; }

.toggle-row { display: flex; justify-content: space-between; align-items: center; padding: 10px 0; border-bottom: 1px solid #f1f5f9; }
.toggle-row:last-of-type { border-bottom: none; }
.toggle-row strong { font-size: 13px; color: #0f172a; display: block; }
.toggle-row p { font-size: 11px; color: #64748b; margin: 2px 0 0 0; }
.toggle-switch { width: 20px; height: 20px; accent-color: #2563eb; cursor: pointer; }

.settings-btn { width: 100%; background: #f1f5f9; border: none; padding: 12px; border-radius: 12px; font-size: 13px; font-weight: 700; color: #1e293b; margin-top: 10px; cursor: pointer; text-align: left; }
.logout-btn { width: 100%; background: #fee2e2; color: #991b1b; border: none; padding: 14px; border-radius: 16px; font-size: 14px; font-weight: 800; cursor: pointer; margin-top: 10px; }
</style>
