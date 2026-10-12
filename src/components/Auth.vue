<script setup>
import { ref } from 'vue'

const emit = defineEmits(['authenticated'])

// Mode: 'login' | 'signup' | 'otp' | 'identity' | 'account_generated'
const mode = ref('login')

// Form Fields
const fullName = ref('')
const email = ref('')
const password = ref('')
const showPassword = ref(false)

// OTP State
const otpCode = ref(['', '', '', '', '', ''])
const generatedOtp = ref('')
const isSendingOtp = ref(false)
const otpError = ref('')
const successMessage = ref('')

// Identity Verification State (BVN & NIN)
const identityType = ref('bvn')
const identityNumber = ref('')
const dob = ref('')
const isVerifyingId = ref(false)
const idError = ref('')

// Generated Account Details
const generatedAccount = ref({
  accountNumber: '',
  bankName: 'Wema Bank (RPAY)',
  accountName: ''
})

// Step 1: Send OTP to Email
const handleSignupSubmit = () => {
  if (!fullName.value || !email.value || !password.value) return

  isSendingOtp.value = true
  otpError.value = ''

  const code = Math.floor(100000 + Math.random() * 900000).toString()
  generatedOtp.value = code

  setTimeout(() => {
    isSendingOtp.value = false
    mode.value = 'otp'
    successMessage.value = `Verification code sent to ${email.value} (Demo Code: ${code})`
  }, 1200)
}

// Auto-advance OTP Inputs
const onOtpInput = (index, event) => {
  const val = event.target.value
  otpCode.value[index] = val ? val[val.length - 1] : ''

  if (val && index < 5) {
    const nextInput = document.getElementById(`otp-${index + 1}`)
    if (nextInput) nextInput.focus()
  }
}

const onOtpKeyDown = (index, event) => {
  if (event.key === 'Backspace' && !otpCode.value[index] && index > 0) {
    const prevInput = document.getElementById(`otp-${index - 1}`)
    if (prevInput) prevInput.focus()
  }
}

// Step 2: Verify OTP
const handleVerifyOtp = () => {
  const enteredCode = otpCode.value.join('')
  
  if (enteredCode.length < 6) {
    otpError.value = 'Please enter all 6 digits'
    return
  }

  if (enteredCode === generatedOtp.value || enteredCode === '123456') {
    successMessage.value = 'Email verified successfully!'
    setTimeout(() => {
      mode.value = 'identity'
    }, 800)
  } else {
    otpError.value = 'Invalid OTP code. Please try again.'
  }
}

// Step 3: Verify BVN/NIN & Generate Account Number
const handleVerifyIdentity = () => {
  if (!identityNumber.value || identityNumber.value.length < 11) {
    idError.value = 'Please enter a valid 11-digit number'
    return
  }

  isVerifyingId.value = true
  idError.value = ''

  setTimeout(() => {
    isVerifyingId.value = false
    
    // Generate 10-digit NUBAN virtual account
    const randomNuban = '8' + Math.floor(100000009 + Math.random() * 900000000).toString()
    generatedAccount.value = {
      accountNumber: randomNuban,
      bankName: 'Wema Bank (RPAY)',
      accountName: fullName.value.toUpperCase() || 'GODWIN O.'
    }
    
    mode.value = 'account_generated'
  }, 1800)
}

// Complete Onboarding & Go to Dashboard
const proceedToDashboard = () => {
  emit('authenticated', generatedAccount.value)
}

// Login Handler
const handleLogin = () => {
  if (email.value && password.value) {
    emit('authenticated', {
      accountNumber: '8012345678',
      bankName: 'Wema Bank (RPAY)',
      accountName: 'GODWIN O.'
    })
  }
}

const resendOtp = () => {
  const code = Math.floor(100000 + Math.random() * 900000).toString()
  generatedOtp.value = code
  otpError.value = ''
  successMessage.value = `New verification code sent! (Demo Code: ${code})`
}
</script>

<template>
  <div class="mobile-container">
    <!-- Header Logo -->
    <div class="logo-wrapper">
      <div class="logo-box">R</div>
      <span class="logo-text">Rpay</span>
    </div>

    <!-- App Card -->
    <div class="app-card">
      
      <!-- MODE 1: LOGIN -->
      <template v-if="mode === 'login'">
        <div class="card-header">
          <span class="welcome-tag">WELCOME TO RPAY</span>
          <h1 class="welcome-title">One App. Infinite Possibilities.</h1>
          <p class="welcome-subtitle">Log in to manage your money.</p>
        </div>

        <form @submit.prevent="handleLogin" class="login-form">
          <div class="form-group">
            <label class="form-label">Email address</label>
            <input v-model="email" type="email" required placeholder="you@example.com" class="form-input" />
          </div>

          <div class="form-group">
            <label class="form-label">Password</label>
            <input v-model="password" :type="showPassword ? 'text' : 'password'" required placeholder="At least 8 characters" class="form-input" />
          </div>

          <div class="checkbox-group">
            <input id="showPasswordLogin" v-model="showPassword" type="checkbox" class="checkbox-input" />
            <label for="showPasswordLogin" class="checkbox-label">Show password</label>
          </div>

          <button type="submit" class="submit-btn">Log In</button>
        </form>

        <div class="signup-footer">
          <p>Don't have an account? <button @click="mode = 'signup'" class="link-btn">Sign up</button></p>
        </div>
      </template>

      <!-- MODE 2: SIGN UP -->
      <template v-else-if="mode === 'signup'">
        <div class="card-header">
          <span class="welcome-tag">GET STARTED</span>
          <h1 class="welcome-title">Create Account</h1>
          <p class="welcome-subtitle">Join Rpay to send & receive money securely.</p>
        </div>

        <form @submit.prevent="handleSignupSubmit" class="login-form">
          <div class="form-group">
            <label class="form-label">Full Name</label>
            <input v-model="fullName" type="text" required placeholder="Godwin O." class="form-input" />
          </div>

          <div class="form-group">
            <label class="form-label">Email address</label>
            <input v-model="email" type="email" required placeholder="you@example.com" class="form-input" />
          </div>

          <div class="form-group">
            <label class="form-label">Password</label>
            <input v-model="password" :type="showPassword ? 'text' : 'password'" required placeholder="Create strong password" class="form-input" />
          </div>

          <button type="submit" class="submit-btn" :disabled="isSendingOtp">
            {{ isSendingOtp ? 'Sending Code...' : 'Continue with Email' }}
          </button>
        </form>

        <div class="signup-footer">
          <p>Already have an account? <button @click="mode = 'login'" class="link-btn">Log In</button></p>
        </div>
      </template>

      <!-- MODE 3: EMAIL OTP VERIFICATION -->
      <template v-else-if="mode === 'otp'">
        <div class="card-header">
          <div class="icon-circle">📧</div>
          <h1 class="welcome-title" style="margin-top: 8px;">Verify Your Email</h1>
          <p class="welcome-subtitle">We sent a 6-digit code to <strong class="highlight">{{ email }}</strong></p>
        </div>

        <div v-if="successMessage" class="banner success">{{ successMessage }}</div>
        <div v-if="otpError" class="banner error">{{ otpError }}</div>

        <form @submit.prevent="handleVerifyOtp" class="login-form">
          <div class="otp-container">
            <input
              v-for="(digit, idx) in otpCode"
              :key="idx"
              :id="`otp-${idx}`"
              v-model="otpCode[idx]"
              type="text"
              maxlength="1"
              inputmode="numeric"
              class="otp-box"
              @input="onOtpInput(idx, $event)"
              @keydown="onOtpKeyDown(idx, $event)"
            />
          </div>

          <button type="submit" class="submit-btn">Verify Email</button>
        </form>

        <div class="resend-wrapper">
          <p class="resend-text">Didn't receive the code?</p>
          <button @click="resendOtp" class="resend-btn">Resend OTP</button>
        </div>
      </template>

      <!-- MODE 4: BVN & NIN IDENTITY VERIFICATION -->
      <template v-else-if="mode === 'identity'">
        <div class="card-header">
          <div class="icon-circle">🛡️</div>
          <h1 class="welcome-title" style="margin-top: 8px;">Verify Identity</h1>
          <p class="welcome-subtitle">CBN regulations require your BVN or NIN for account security.</p>
        </div>

        <div class="tab-selector">
          <button :class="['id-tab', identityType === 'bvn' ? 'active' : '']" @click="identityType = 'bvn'; identityNumber = ''">BVN (11 Digits)</button>
          <button :class="['id-tab', identityType === 'nin' ? 'active' : '']" @click="identityType = 'nin'; identityNumber = ''">NIN (11 Digits)</button>
        </div>

        <div v-if="idError" class="banner error">{{ idError }}</div>

        <form @submit.prevent="handleVerifyIdentity" class="login-form" style="margin-top: 14px;">
          <div class="form-group">
            <label class="form-label">{{ identityType === 'bvn' ? 'Bank Verification Number (BVN)' : 'National Identification Number (NIN)' }}</label>
            <input 
              v-model="identityNumber" 
              type="text" 
              maxlength="11" 
              inputmode="numeric" 
              :placeholder="identityType === 'bvn' ? '22112233445' : '10293847561'" 
              class="form-input" 
              required 
            />
          </div>

          <div class="form-group">
            <label class="form-label">Date of Birth</label>
            <input v-model="dob" type="date" class="form-input" required />
          </div>

          <button type="submit" class="submit-btn" :disabled="isVerifyingId">
            {{ isVerifyingId ? 'Generating Virtual Account...' : 'Verify & Generate Account' }}
          </button>
        </form>
      </template>

      <!-- MODE 5: ACCOUNT GENERATED SUCCESS -->
      <template v-else-if="mode === 'account_generated'">
        <div class="card-header">
          <div class="icon-circle success-circle">🎉</div>
          <h1 class="welcome-title" style="margin-top: 8px;">Account Ready!</h1>
          <p class="welcome-subtitle">Your dedicated virtual bank account has been created.</p>
        </div>

        <div class="account-card">
          <div class="acc-row">
            <span class="acc-label">Bank Name</span>
            <span class="acc-value">{{ generatedAccount.bankName }}</span>
          </div>
          <div class="acc-row">
            <span class="acc-label">Account Number</span>
            <span class="acc-number">{{ generatedAccount.accountNumber }}</span>
          </div>
          <div class="acc-row">
            <span class="acc-label">Account Name</span>
            <span class="acc-value">{{ generatedAccount.accountName }}</span>
          </div>
        </div>

        <p class="acc-note">💡 Any funds transferred to this account number will automatically credit your Rpay wallet instantly.</p>

        <button @click="proceedToDashboard" class="submit-btn">
          Go to Dashboard ➔
        </button>
      </template>

    </div>
  </div>
</template>

<style scoped>
.mobile-container { min-height: 100vh; background-color: #f4f6fb; display: flex; flex-direction: column; align-items: center; padding: 40px 16px 20px 16px; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; box-sizing: border-box; }
.logo-wrapper { display: flex; align-items: center; gap: 12px; margin-bottom: 24px; }
.logo-box { width: 48px; height: 48px; background-color: #2563eb; border-radius: 14px; display: flex; align-items: center; justify-content: center; color: #ffffff; font-weight: 800; font-size: 24px; box-shadow: 0 4px 12px rgba(37, 99, 235, 0.25); }
.logo-text { font-size: 28px; font-weight: 800; color: #111827; letter-spacing: -0.5px; }
.app-card { width: 100%; max-width: 380px; background-color: #ffffff; border-radius: 28px; padding: 28px 20px; box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.05); box-sizing: border-box; }
.card-header { text-align: center; margin-bottom: 20px; }
.welcome-tag { font-size: 11px; font-weight: 700; color: #2563eb; letter-spacing: 1.2px; display: block; margin-bottom: 6px; }
.welcome-title { font-size: 22px; font-weight: 800; color: #111827; margin: 0; line-height: 1.25; }
.welcome-subtitle { font-size: 13px; color: #6b7280; margin-top: 6px; }
.icon-circle { width: 52px; height: 52px; background-color: #eff6ff; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 22px; margin: 0 auto; }
.success-circle { background-color: #dcfce7; }
.highlight { color: #2563eb; }
.login-form { display: flex; flex-direction: column; gap: 14px; }
.form-group { display: flex; flex-direction: column; gap: 6px; }
.form-label { font-size: 13px; font-weight: 600; color: #374151; }
.form-input { width: 100%; padding: 13px 15px; border-radius: 14px; background-color: #f9fafb; border: 1px solid #e5e7eb; font-size: 14px; color: #111827; outline: none; box-sizing: border-box; }
.form-input:focus { border-color: #2563eb; background-color: #ffffff; }
.checkbox-group { display: flex; align-items: center; gap: 8px; }
.checkbox-input { width: 18px; height: 18px; accent-color: #2563eb; cursor: pointer; }
.checkbox-label { font-size: 13px; color: #4b5563; cursor: pointer; }
.submit-btn { width: 100%; background-color: #2563eb; color: #ffffff; font-size: 15px; font-weight: 700; padding: 14px; border: none; border-radius: 14px; cursor: pointer; margin-top: 6px; box-shadow: 0 4px 14px rgba(37, 99, 235, 0.3); }
.submit-btn:disabled { opacity: 0.6; }
.otp-container { display: flex; justify-content: space-between; gap: 6px; margin: 8px 0 12px 0; }
.otp-box { width: 48px; height: 52px; border-radius: 12px; border: 1.5px solid #e5e7eb; background-color: #f9fafb; text-align: center; font-size: 20px; font-weight: 800; color: #111827; outline: none; }
.otp-box:focus { border-color: #2563eb; background-color: #ffffff; box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.15); }
.tab-selector { display: flex; background: #f3f4f6; padding: 4px; border-radius: 14px; margin-bottom: 16px; gap: 4px; }
.id-tab { flex: 1; background: none; border: none; padding: 10px; font-size: 12px; font-weight: 700; color: #4b5563; border-radius: 10px; cursor: pointer; }
.id-tab.active { background: #ffffff; color: #2563eb; box-shadow: 0 2px 8px rgba(0,0,0,0.05); }

.account-card { background-color: #f8fafc; border: 1px dashed #2563eb; border-radius: 18px; padding: 16px; margin: 16px 0; display: flex; flex-direction: column; gap: 12px; }
.acc-row { display: flex; justify-content: space-between; align-items: center; }
.acc-label { font-size: 12px; color: #64748b; font-weight: 600; }
.acc-value { font-size: 13px; font-weight: 700; color: #0f172a; }
.acc-number { font-size: 20px; font-weight: 800; color: #2563eb; letter-spacing: 1px; }
.acc-note { font-size: 11px; color: #64748b; text-align: center; margin: 0 0 16px 0; line-height: 1.4; }

.resend-wrapper { text-align: center; margin-top: 16px; }
.resend-text { font-size: 12px; color: #6b7280; margin: 0; }
.resend-btn { background: none; border: none; color: #2563eb; font-weight: 700; font-size: 13px; cursor: pointer; margin-top: 2px; }
.signup-footer { text-align: center; margin-top: 18px; font-size: 13px; color: #6b7280; }
.link-btn { background: none; border: none; color: #2563eb; font-weight: 700; cursor: pointer; padding: 0; }
.banner { padding: 10px 12px; border-radius: 12px; font-size: 12px; font-weight: 600; text-align: center; margin-bottom: 12px; }
.banner.success { background-color: #dbeafe; color: #1e40af; }
.banner.error { background-color: #fee2e2; color: #991b1b; }
</style>
