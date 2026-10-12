<script setup>
import { ref, computed, onMounted } from 'vue'

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

const emit = defineEmits(['openProfile'])

// Theme State
const isDarkMode = ref(false)

const toggleTheme = () => {
  isDarkMode.value = !isDarkMode.value
  document.documentElement.classList.toggle('dark-theme', isDarkMode.value)
  localStorage.setItem('rpay-theme', isDarkMode.value ? 'dark' : 'light')
}

onMounted(() => {
  const savedTheme = localStorage.getItem('rpay-theme')
  if (savedTheme === 'dark' || (!savedTheme && window.matchMedia('(prefers-color-scheme: dark)').matches)) {
    isDarkMode.value = true
    document.documentElement.classList.add('dark-theme')
  }
})

// Core State
const walletBalance = ref(50000.00)
const isBalanceHidden = ref(false)
const copied = ref(false)
const selectedTx = ref(null)

// Transaction Search & Filter State
const searchQuery = ref('')
const activeFilter = ref('ALL') // 'ALL' | 'CREDIT' | 'DEBIT'

// Modals: null | 'fund' | 'transfer' | 'airtime' | 'bills' | 'pin'
const activeModal = ref(null)
const pendingAction = ref(null)

// PIN Verification State
const pinCode = ref(['', '', '', ''])
const pinError = ref('')

// Fund Wallet State
const fundAmount = ref('')
const isProcessingFund = ref(false)

// Transfer State
const transferData = ref({ bank: 'Access Bank', accountNumber: '', accountName: '', amount: '', note: '' })
const isResolvingAccount = ref(false)

// Saved Beneficiaries
const beneficiaries = ref([
  { id: 1, name: 'John Doe', bank: 'Access Bank', accountNumber: '0123456789' },
  { id: 2, name: 'Sarah Connor', bank: 'GTBank', accountNumber: '0987654321' },
  { id: 3, name: 'Alex Smith', bank: 'Zenith Bank', accountNumber: '5544332211' }
])

const selectBeneficiary = (b) => {
  transferData.value.bank = b.bank
  transferData.value.accountNumber = b.accountNumber
  transferData.value.accountName = b.name
}

// Airtime & Bills State
const airtimeData = ref({ network: 'MTN', phone: '', amount: '' })
const billData = ref({ category: 'Electricity', service: 'IKEDC Postpaid', meterNo: '', amount: '' })

// Transactions History Data
const transactions = ref([
  { id: 'TX-1001', title: 'Wallet Funding', type: 'Credit', amount: 20000, date: '12 Oct 2026, 02:15 AM', status: 'Successful', ref: 'PAY-9823101' },
  { id: 'TX-1002', title: 'Airtime Recharge (MTN)', type: 'Debit', amount: 1000, date: '11 Oct 2026, 08:30 PM', status: 'Successful', ref: 'PAY-4810293' },
  { id: 'TX-1003', title: 'Transfer to John Doe', type: 'Debit', amount: 5000, date: '10 Oct 2026, 01:12 PM', status: 'Successful', ref: 'PAY-7718293' }
])

// Computed Filtered Transactions
const filteredTransactions = computed(() => {
  return transactions.value.filter(tx => {
    const matchesFilter = 
      activeFilter.value === 'ALL' ||
      (activeFilter.value === 'CREDIT' && tx.type === 'Credit') ||
      (activeFilter.value === 'DEBIT' && tx.type === 'Debit')

    const q = searchQuery.value.toLowerCase().trim()
    const matchesSearch = 
      !q ||
      tx.title.toLowerCase().includes(q) ||
      tx.ref.toLowerCase().includes(q) ||
      tx.amount.toString().includes(q)

    return matchesFilter && matchesSearch
  })
})

const copyAccountNumber = () => {
  navigator.clipboard.writeText(props.userAccount.accountNumber)
  copied.value = true
  setTimeout(() => { copied.value = false }, 2000)
}

const openReceipt = (tx) => { selectedTx.value = tx }
const closeReceipt = () => { selectedTx.value = null }

// Download / Print Receipt Handler
const downloadReceipt = () => {
  window.print()
}

// Share Receipt Handler
const shareReceipt = async () => {
  if (!selectedTx.value) return
  const text = `Rpay Transaction Receipt:\nDescription: ${selectedTx.value.title}\nAmount: ₦${selectedTx.value.amount.toLocaleString()}\nRef: ${selectedTx.value.ref}\nStatus: ${selectedTx.value.status}`
  
  if (navigator.share) {
    try {
      await navigator.share({ title: 'Transaction Receipt', text })
    } catch (e) {
      console.log('Share canceled', e)
    }
  } else {
    navigator.clipboard.writeText(text)
    alert('Receipt summary copied to clipboard!')
  }
}

const recordTransaction = (title, type, amount, reference) => {
  transactions.value.unshift({
    id: `TX-${Math.floor(1000 + Math.random() * 9000)}`,
    title,
    type,
    amount: parseFloat(amount),
    date: new Date().toLocaleString('en-NG', { dateStyle: 'medium', timeStyle: 'short' }),
    status: 'Successful',
    ref: reference || 'RPAY-' + Math.floor(Math.random() * 1000000000)
  })
}

// Open PIN Verification Modal Before Processing
const promptPinForAction = (actionType) => {
  pendingAction.value = actionType
  activeModal.value = 'pin'
  pinCode.value = ['', '', '', '']
  pinError.value = ''
}

// Auto-advance PIN Inputs
const onPinInput = (index, event) => {
  const val = event.target.value
  pinCode.value[index] = val ? val[val.length - 1] : ''
  if (val && index < 3) {
    const next = document.getElementById(`pin-${index + 1}`)
    if (next) next.focus()
  }
}

// Validate PIN and execute transaction
const handleVerifyPin = () => {
  const enteredPin = pinCode.value.join('')
  if (enteredPin.length < 4) {
    pinError.value = 'Enter 4-digit PIN'
    return
  }

  if (enteredPin !== '1234') {
    pinError.value = 'Incorrect PIN (Default: 1234)'
    return
  }

  // PIN Verified: Execute Pending Transaction
  const action = pendingAction.value
  activeModal.value = null

  if (action === 'transfer') {
    const amt = parseFloat(transferData.value.amount)
    walletBalance.value -= amt
    const recipient = transferData.value.accountName || transferData.value.bank
    recordTransaction(`Transfer to ${recipient}`, 'Debit', amt)
    transferData.value = { bank: 'Access Bank', accountNumber: '', accountName: '', amount: '', note: '' }
    alert(`Transfer of ₦${amt.toLocaleString()} successful!`)
  } else if (action === 'airtime') {
    const amt = parseFloat(airtimeData.value.amount)
    walletBalance.value -= amt
    recordTransaction(`Airtime Recharge (${airtimeData.value.network})`, 'Debit', amt)
    airtimeData.value = { network: 'MTN', phone: '', amount: '' }
    alert(`Airtime recharge of ₦${amt.toLocaleString()} successful!`)
  } else if (action === 'bills') {
    const amt = parseFloat(billData.value.amount)
    walletBalance.value -= amt
    recordTransaction(`${billData.value.category} (${billData.value.service})`, 'Debit', amt)
    billData.value = { category: 'Electricity', service: 'IKEDC Postpaid', meterNo: '', amount: '' }
    alert(`Bill payment of ₦${amt.toLocaleString()} successful!`)
  }
}

// Account Name Resolution
const handleAccountNumberInput = () => {
  const num = transferData.value.accountNumber
  if (num.length === 10) {
    isResolvingAccount.value = true
    transferData.value.accountName = ''
    setTimeout(() => {
      isResolvingAccount.value = false
      transferData.value.accountName = 'GODWIN J. O.'
    }, 800)
  } else {
    transferData.value.accountName = ''
  }
}

// Form Handlers
const submitTransfer = () => {
  const amt = parseFloat(transferData.value.amount)
  if (!amt || amt <= 0) return alert('Enter valid amount')
  if (amt > walletBalance.value) return alert('Insufficient balance')
  if (transferData.value.accountNumber.length < 10) return alert('Enter 10-digit account number')
  promptPinForAction('transfer')
}

const submitAirtime = () => {
  const amt = parseFloat(airtimeData.value.amount)
  if (!amt || amt <= 0) return alert('Enter valid amount')
  if (amt > walletBalance.value) return alert('Insufficient balance')
  if (airtimeData.value.phone.length < 11) return alert('Enter 11-digit phone number')
  promptPinForAction('airtime')
}

const submitBills = () => {
  const amt = parseFloat(billData.value.amount)
  if (!amt || amt <= 0) return alert('Enter valid amount')
  if (amt > walletBalance.value) return alert('Insufficient balance')
  promptPinForAction('bills')
}

const handlePaystackPayment = () => {
  const amt = parseFloat(fundAmount.value)
  if (!amt || amt < 100) return alert('Min amount is ₦100')
  isProcessingFund.value = true
  const handler = window.PaystackPop.setup({
    key: 'pk_test_a9f8cdc52070923cf4d69',
    email: 'godwin@example.com',
    amount: amt * 100,
    currency: 'NGN',
    ref: 'RPAY-' + Math.floor(Math.random() * 1000000000 + 1),
    onClose: () => { isProcessingFund.value = false },
    callback: (res) => {
      isProcessingFund.value = false
      walletBalance.value += amt
      recordTransaction('Card Wallet Funding', 'Credit', amt, res.reference)
      fundAmount.value = ''
      activeModal.value = null
      alert(`Wallet credited with ₦${amt.toLocaleString()}`)
    }
  })
  handler.openIframe()
}
</script>

<template>
  <div :class="['dashboard-wrapper', { 'dark-mode': isDarkMode }]">
    <!-- Header Card -->
    <div class="header-card">
      <div class="user-greeting">
        <div>
          <span class="greeting-sub">Welcome back 👋</span>
          <h2 class="greeting-name">{{ props.userAccount.accountName }}</h2>
        </div>
        
        <div class="header-actions">
          <button @click="toggleTheme" class="theme-toggle-btn" :title="isDarkMode ? 'Switch to Light Mode' : 'Switch to Dark Mode'">
            {{ isDarkMode ? '☀️' : '🌙' }}
          </button>
          <div @click="emit('openProfile')" class="avatar" title="Open Profile">G</div>
        </div>
      </div>

      <!-- Balance Box -->
      <div class="balance-card">
        <div class="balance-header">
          <span class="balance-label">Total Wallet Balance</span>
          <button @click="isBalanceHidden = !isBalanceHidden" class="eye-btn">
            {{ isBalanceHidden ? '👁️ Show' : '🙈 Hide' }}
          </button>
        </div>
        <h1 class="balance-amount">
          {{ isBalanceHidden ? '••••••••' : `₦${walletBalance.toLocaleString('en-NG', { minimumFractionDigits: 2 })}` }}
        </h1>
      </div>

      <!-- Virtual Account Card -->
      <div class="virtual-account-box">
        <div class="acc-info">
          <span class="bank-tag">{{ props.userAccount.bankName }}</span>
          <div class="acc-num-row">
            <span class="acc-num">{{ props.userAccount.accountNumber }}</span>
            <button @click="copyAccountNumber" class="copy-btn">
              {{ copied ? '✓ Copied' : 'Copy' }}
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Quick Actions -->
    <div class="actions-grid">
      <div @click="activeModal = 'transfer'" class="action-item"><div class="action-icon">💸</div><span>Transfer</span></div>
      <div @click="activeModal = 'airtime'" class="action-item"><div class="action-icon">📱</div><span>Airtime</span></div>
      <div @click="activeModal = 'bills'" class="action-item"><div class="action-icon">💡</div><span>Bills</span></div>
      <div @click="activeModal = 'fund'" class="action-item highlight-action"><div class="action-icon">💳</div><span>Fund</span></div>
    </div>

    <!-- TRANSACTION HISTORY WITH SEARCH & FILTER -->
    <div class="tx-section">
      <div class="tx-header">
        <h3>Recent Transactions</h3>
      </div>

      <!-- Search Bar -->
      <div class="search-box">
        <span class="search-icon">🔍</span>
        <input 
          v-model="searchQuery" 
          type="text" 
          placeholder="Search by title, ref or amount..." 
          class="search-input"
        />
        <button v-if="searchQuery" @click="searchQuery = ''" class="clear-search">✕</button>
      </div>

      <!-- Category Filter Tabs -->
      <div class="filter-tabs">
        <button 
          :class="['filter-tab', { active: activeFilter === 'ALL' }]" 
          @click="activeFilter = 'ALL'"
        >
          All
        </button>
        <button 
          :class="['filter-tab', { active: activeFilter === 'CREDIT' }]" 
          @click="activeFilter = 'CREDIT'"
        >
          ↓ Credits
        </button>
        <button 
          :class="['filter-tab', { active: activeFilter === 'DEBIT' }]" 
          @click="activeFilter = 'DEBIT'"
        >
          ↑ Debits
        </button>
      </div>

      <!-- Transaction List -->
      <div class="tx-list">
        <div v-if="filteredTransactions.length === 0" class="no-tx-box">
          <p>No matching transactions found.</p>
        </div>

        <div 
          v-for="tx in filteredTransactions" 
          :key="tx.id" 
          class="tx-card" 
          @click="openReceipt(tx)"
        >
          <div class="tx-left">
            <div :class="['tx-badge', tx.type === 'Credit' ? 'credit' : 'debit']">
              {{ tx.type === 'Credit' ? '↓' : '↑' }}
            </div>
            <div>
              <div class="tx-title">{{ tx.title }}</div>
              <div class="tx-date">{{ tx.date }}</div>
            </div>
          </div>
          <div :class="['tx-amount', tx.type === 'Credit' ? 'credit' : 'debit']">
            {{ tx.type === 'Credit' ? '+' : '-' }}₦{{ tx.amount.toLocaleString() }}
          </div>
        </div>
      </div>
    </div>

    <!-- PIN SECURITY MODAL -->
    <div v-if="activeModal === 'pin'" class="modal-overlay" @click.self="activeModal = null">
      <div class="modal-card text-center">
        <div class="modal-header">
          <h3>Transaction PIN</h3>
          <button @click="activeModal = null" class="close-x">✕</button>
        </div>
        <p class="modal-sub">Enter your 4-digit PIN to authorize payment.</p>

        <div v-if="pinError" class="pin-error">{{ pinError }}</div>

        <div class="pin-inputs">
          <input
            v-for="(digit, idx) in pinCode"
            :key="idx"
            :id="`pin-${idx}`"
            v-model="pinCode[idx]"
            type="password"
            maxlength="1"
            inputmode="numeric"
            class="pin-box"
            @input="onPinInput(idx, $event)"
          />
        </div>

        <button @click="handleVerifyPin" class="submit-btn" style="margin-top: 16px;">
          Authorize Payment 🔒
        </button>
        <p class="demo-pin-hint">Default PIN: <strong>1234</strong></p>
      </div>
    </div>

    <!-- TRANSFER MODAL -->
    <div v-if="activeModal === 'transfer'" class="modal-overlay" @click.self="activeModal = null">
      <div class="modal-card">
        <div class="modal-header">
          <h3>Send Money</h3>
          <button @click="activeModal = null" class="close-x">✕</button>
        </div>
        <p class="modal-sub">Transfer to any Nigerian bank account.</p>

        <!-- SAVED BENEFICIARIES ROW -->
        <div class="saved-beneficiaries">
          <span class="beneficiary-label">Saved Beneficiaries:</span>
          <div class="beneficiaries-list">
            <button 
              v-for="b in beneficiaries" 
              :key="b.id" 
              type="button" 
              class="beneficiary-chip"
              @click="selectBeneficiary(b)"
            >
              👤 {{ b.name.split(' ')[0] }}
            </button>
          </div>
        </div>

        <form @submit.prevent="submitTransfer" class="modal-form">
          <div class="form-group">
            <label class="form-label">Select Bank</label>
            <select v-model="transferData.bank" class="form-input">
              <option>Access Bank</option>
              <option>GTBank</option>
              <option>First Bank</option>
              <option>UBA</option>
              <option>Zenith Bank</option>
              <option>Kuda Bank</option>
              <option>Opay</option>
              <option>Palmpay</option>
            </select>
          </div>
          <div class="form-group">
            <label class="form-label">Account Number</label>
            <input v-model="transferData.accountNumber" @input="handleAccountNumberInput" type="text" maxlength="10" placeholder="10-digit number" class="form-input" required />
          </div>
          <div class="form-group">
            <label class="form-label">Account Name</label>
            <div class="account-name-box">
              <span v-if="isResolvingAccount" class="resolving-text">🔍 Verifying account details...</span>
              <span v-else-if="transferData.accountName" class="verified-name">✓ {{ transferData.accountName }}</span>
              <span v-else class="placeholder-text">Enter 10-digit account number</span>
            </div>
          </div>
          <div class="form-group">
            <label class="form-label">Amount (₦)</label>
            <input v-model="transferData.amount" type="number" placeholder="0.00" class="form-input" required />
          </div>
          <button type="submit" class="submit-btn" :disabled="isResolvingAccount">Continue</button>
        </form>
      </div>
    </div>

    <!-- AIRTIME MODAL -->
    <div v-if="activeModal === 'airtime'" class="modal-overlay" @click.self="activeModal = null">
      <div class="modal-card">
        <div class="modal-header">
          <h3>Buy Airtime</h3>
          <button @click="activeModal = null" class="close-x">✕</button>
        </div>
        <form @submit.prevent="submitAirtime" class="modal-form" style="margin-top: 10px;">
          <div class="form-group">
            <label class="form-label">Network</label>
            <select v-model="airtimeData.network" class="form-input">
              <option>MTN</option>
              <option>Airtel</option>
              <option>Glo</option>
              <option>9mobile</option>
            </select>
          </div>
          <div class="form-group">
            <label class="form-label">Phone Number</label>
            <input v-model="airtimeData.phone" type="tel" maxlength="11" placeholder="08012345678" class="form-input" required />
          </div>
          <div class="form-group">
            <label class="form-label">Amount (₦)</label>
            <input v-model="airtimeData.amount" type="number" placeholder="100 - 50000" class="form-input" required />
          </div>
          <button type="submit" class="submit-btn">Continue</button>
        </form>
      </div>
    </div>

    <!-- BILLS MODAL -->
    <div v-if="activeModal === 'bills'" class="modal-overlay" @click.self="activeModal = null">
      <div class="modal-card">
        <div class="modal-header">
          <h3>Pay Utility Bills</h3>
          <button @click="activeModal = null" class="close-x">✕</button>
        </div>
        <form @submit.prevent="submitBills" class="modal-form" style="margin-top: 10px;">
          <div class="form-group">
            <label class="form-label">Category</label>
            <select v-model="billData.category" class="form-input">
              <option>Electricity</option>
              <option>Cable TV</option>
              <option>Internet</option>
            </select>
          </div>
          <div class="form-group">
            <label class="form-label">Smartcard / Meter / Account No.</label>
            <input v-model="billData.meterNo" type="text" placeholder="Enter number" class="form-input" required />
          </div>
          <div class="form-group">
            <label class="form-label">Amount (₦)</label>
            <input v-model="billData.amount" type="number" placeholder="Amount" class="form-input" required />
          </div>
          <button type="submit" class="submit-btn">Continue</button>
        </form>
      </div>
    </div>

    <!-- FUND WALLET MODAL -->
    <div v-if="activeModal === 'fund'" class="modal-overlay" @click.self="activeModal = null">
      <div class="modal-card">
        <div class="modal-header">
          <h3>Fund Wallet</h3>
          <button @click="activeModal = null" class="close-x">✕</button>
        </div>
        <p class="modal-sub">Add funds instantly using Paystack Debit Card.</p>
        <div class="form-group">
          <label class="form-label">Amount (₦)</label>
          <input v-model="fundAmount" type="number" placeholder="e.g. 5000" class="form-input" />
        </div>
        <div class="quick-amounts">
          <button @click="fundAmount = '1000'">+₦1,000</button>
          <button @click="fundAmount = '5000'">+₦5,000</button>
          <button @click="fundAmount = '10000'">+₦10,000</button>
        </div>
        <button @click="handlePaystackPayment" class="submit-btn" :disabled="isProcessingFund">
          {{ isProcessingFund ? 'Initializing Paystack...' : 'Pay with Paystack' }}
        </button>
      </div>
    </div>

    <!-- RECEIPT MODAL WITH DOWNLOAD & SHARE -->
    <div v-if="selectedTx" class="modal-overlay" @click.self="closeReceipt">
      <div class="receipt-card printable-area">
        <div class="receipt-header">
          <div class="receipt-icon">🧾</div>
          <h2>Transaction Receipt</h2>
          <span class="receipt-status">{{ selectedTx.status }}</span>
        </div>
        <div class="receipt-body">
          <div class="receipt-amount">₦{{ selectedTx.amount.toLocaleString('en-NG', { minimumFractionDigits: 2 }) }}</div>
          <div class="receipt-divider"></div>
          <div class="receipt-row"><span class="r-label">Type</span><span class="r-value">{{ selectedTx.type }}</span></div>
          <div class="receipt-row"><span class="r-label">Description</span><span class="r-value">{{ selectedTx.title }}</span></div>
          <div class="receipt-row"><span class="r-label">Date & Time</span><span class="r-value">{{ selectedTx.date }}</span></div>
          <div class="receipt-row"><span class="r-label">Reference</span><span class="r-value font-mono">{{ selectedTx.ref }}</span></div>
        </div>

        <!-- RECEIPT ACTION BUTTONS -->
        <div class="receipt-actions no-print">
          <button @click="downloadReceipt" class="receipt-btn secondary">📥 Download / Print</button>
          <button @click="shareReceipt" class="receipt-btn primary">📤 Share</button>
        </div>

        <button @click="closeReceipt" class="close-receipt-btn no-print">Close Receipt</button>
      </div>
    </div>

  </div>
</template>

<style scoped>
.dashboard-wrapper {
  --bg-color: #f8fafc;
  --card-bg: #ffffff;
  --text-primary: #0f172a;
  --text-secondary: #64748b;
  --border-color: #e2e8f0;
  --header-bg: linear-gradient(135deg, #1e40af, #2563eb);

  max-width: 420px;
  margin: 0 auto;
  background-color: var(--bg-color);
  min-height: 100vh;
  padding-bottom: 30px;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
  transition: background-color 0.3s ease, color 0.3s ease;
}

.dashboard-wrapper.dark-mode {
  --bg-color: #090d16;
  --card-bg: #131c2e;
  --text-primary: #f8fafc;
  --text-secondary: #94a3b8;
  --border-color: #1e293b;
  --header-bg: linear-gradient(135deg, #0f172a, #1e3a8a);
}

.header-card {
  background: var(--header-bg);
  padding: 24px 20px 20px 20px;
  border-radius: 0 0 28px 28px;
  color: #ffffff;
}

.user-greeting { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
.greeting-sub { font-size: 12px; color: #93c5fd; }
.greeting-name { font-size: 18px; font-weight: 800; margin: 2px 0 0 0; }
.header-actions { display: flex; align-items: center; gap: 10px; }
.theme-toggle-btn { background: rgba(255, 255, 255, 0.2); border: none; width: 36px; height: 36px; border-radius: 50%; font-size: 16px; cursor: pointer; display: flex; align-items: center; justify-content: center; backdrop-filter: blur(4px); }
.avatar { width: 40px; height: 40px; background: rgba(255,255,255,0.2); border-radius: 50%; display: flex; align-items: center; justify-content: center; font-weight: 800; cursor: pointer; }

.balance-card { margin-bottom: 16px; }
.balance-header { display: flex; justify-content: space-between; align-items: center; }
.balance-label { font-size: 12px; color: #bfdbfe; }
.eye-btn { background: rgba(255,255,255,0.15); border: none; color: #ffffff; font-size: 11px; padding: 4px 10px; border-radius: 12px; cursor: pointer; font-weight: 600; }
.balance-amount { font-size: 28px; font-weight: 800; margin: 4px 0 0 0; }

.virtual-account-box { background: rgba(255,255,255,0.12); border: 1px solid rgba(255,255,255,0.2); backdrop-filter: blur(8px); padding: 12px 14px; border-radius: 16px; }
.bank-tag { font-size: 11px; color: #93c5fd; font-weight: 700; display: block; margin-bottom: 4px; }
.acc-num-row { display: flex; justify-content: space-between; align-items: center; }
.acc-num { font-size: 18px; font-weight: 800; letter-spacing: 1px; }
.copy-btn { background: #ffffff; color: #2563eb; border: none; font-size: 11px; font-weight: 700; padding: 6px 12px; border-radius: 8px; cursor: pointer; }

/* ACTION BUTTONS GRID */
.actions-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 12px; padding: 20px; }
.action-item { background: var(--card-bg); padding: 14px 8px; border-radius: 18px; display: flex; flex-direction: column; align-items: center; gap: 6px; box-shadow: 0 4px 12px rgba(0,0,0,0.03); cursor: pointer; font-size: 12px; font-weight: 600; color: var(--text-primary); border: 1px solid var(--border-color); }
.action-icon { font-size: 20px; }
.highlight-action { border: 1.5px solid #2563eb; background: #eff6ff; }
.dark-mode .highlight-action { background: #1e293b; border-color: #3b82f6; }

/* SEARCH & FILTER STYLES */
.tx-section { padding: 0 20px; }
.tx-header h3 { font-size: 16px; font-weight: 800; color: var(--text-primary); margin-bottom: 12px; }

.search-box { position: relative; display: flex; align-items: center; margin-bottom: 12px; }
.search-icon { position: absolute; left: 12px; font-size: 12px; color: var(--text-secondary); }
.search-input { width: 100%; padding: 10px 32px 10px 34px; border-radius: 12px; border: 1px solid var(--border-color); background: var(--card-bg); color: var(--text-primary); font-size: 12px; outline: none; box-sizing: border-box; }
.clear-search { position: absolute; right: 10px; background: none; border: none; font-size: 12px; color: var(--text-secondary); cursor: pointer; }

.filter-tabs { display: flex; gap: 8px; margin-bottom: 14px; }
.filter-tab { flex: 1; padding: 6px; border-radius: 10px; border: 1px solid var(--border-color); background: var(--card-bg); color: var(--text-secondary); font-size: 11px; font-weight: 700; cursor: pointer; text-align: center; }
.filter-tab.active { background: #2563eb; color: #ffffff; border-color: #2563eb; }

.tx-list { display: flex; flex-direction: column; gap: 10px; }
.no-tx-box { padding: 20px; text-align: center; color: var(--text-secondary); font-size: 12px; font-weight: 600; }
.tx-card { background: var(--card-bg); padding: 14px; border-radius: 16px; display: flex; justify-content: space-between; align-items: center; box-shadow: 0 2px 8px rgba(0,0,0,0.02); cursor: pointer; border: 1px solid var(--border-color); }
.tx-left { display: flex; align-items: center; gap: 12px; }
.tx-badge { width: 36px; height: 36px; border-radius: 10px; display: flex; align-items: center; justify-content: center; font-weight: 800; }
.tx-badge.credit { background: #dcfce7; color: #166534; }
.tx-badge.debit { background: #fee2e2; color: #991b1b; }
.tx-title { font-size: 13px; font-weight: 700; color: var(--text-primary); }
.tx-date { font-size: 11px; color: var(--text-secondary); }
.tx-amount { font-size: 14px; font-weight: 800; }
.tx-amount.credit { color: #166534; }
.tx-amount.debit { color: var(--text-primary); }

/* SAVED BENEFICIARIES */
.saved-beneficiaries { margin-bottom: 12px; }
.beneficiary-label { font-size: 11px; font-weight: 700; color: var(--text-secondary); display: block; margin-bottom: 6px; }
.beneficiaries-list { display: flex; gap: 8px; overflow-x: auto; padding-bottom: 4px; }
.beneficiary-chip { background: #f1f5f9; border: 1px solid #cbd5e1; border-radius: 12px; padding: 6px 10px; font-size: 11px; font-weight: 700; color: #2563eb; cursor: pointer; white-space: nowrap; }

/* MODALS */
.modal-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.5); backdrop-filter: blur(4px); display: flex; align-items: center; justify-content: center; z-index: 100; padding: 16px; }
.modal-card { background: var(--card-bg); border-radius: 24px; padding: 20px; width: 100%; max-width: 340px; box-shadow: 0 20px 25px -5px rgba(0,0,0,0.1); border: 1px solid var(--border-color); }
.modal-header { display: flex; justify-content: space-between; align-items: center; }
.modal-header h3 { font-size: 18px; font-weight: 800; color: var(--text-primary); margin: 0; }
.close-x { background: none; border: none; font-size: 18px; color: var(--text-secondary); cursor: pointer; }
.modal-sub { font-size: 12px; color: var(--text-secondary); margin: 6px 0 16px 0; }
.modal-form { display: flex; flex-direction: column; gap: 12px; }
.form-group { display: flex; flex-direction: column; gap: 6px; }
.form-label { font-size: 12px; font-weight: 700; color: var(--text-primary); }
.form-input { width: 100%; padding: 12px; border-radius: 12px; border: 1px solid var(--border-color); outline: none; font-size: 14px; box-sizing: border-box; background-color: var(--card-bg); color: var(--text-primary); }

.account-name-box { width: 100%; padding: 12px; border-radius: 12px; background-color: var(--bg-color); border: 1px solid var(--border-color); font-size: 13px; font-weight: 700; box-sizing: border-box; min-height: 42px; display: flex; align-items: center; }
.verified-name { color: #166534; font-weight: 800; }
.resolving-text { color: #2563eb; font-weight: 600; font-size: 12px; }
.placeholder-text { color: var(--text-secondary); font-weight: 500; font-size: 12px; }

/* PIN STYLES */
.pin-inputs { display: flex; justify-content: center; gap: 12px; margin: 16px 0; }
.pin-box { width: 48px; height: 52px; border-radius: 12px; border: 1.5px solid var(--border-color); background: var(--bg-color); color: var(--text-primary); text-align: center; font-size: 22px; font-weight: 800; outline: none; }
.pin-box:focus { border-color: #2563eb; background: var(--card-bg); box-shadow: 0 0 0 3px rgba(37,99,235,0.15); }
.pin-error { font-size: 12px; color: #dc2626; font-weight: 700; margin-bottom: 8px; text-align: center; }
.demo-pin-hint { font-size: 11px; color: var(--text-secondary); text-align: center; margin-top: 10px; }

.quick-amounts { display: flex; gap: 8px; margin-bottom: 16px; }
.quick-amounts button { flex: 1; background: var(--bg-color); border: 1px solid var(--border-color); padding: 8px; border-radius: 10px; font-size: 11px; font-weight: 700; color: #2563eb; cursor: pointer; }
.submit-btn { width: 100%; background: #2563eb; color: #ffffff; border: none; padding: 14px; border-radius: 12px; font-weight: 700; cursor: pointer; margin-top: 6px; }
.submit-btn:disabled { opacity: 0.6; }

/* RECEIPT STYLES */
.receipt-card { background: var(--card-bg); border-radius: 24px; padding: 24px; width: 100%; max-width: 340px; box-shadow: 0 20px 25px -5px rgba(0,0,0,0.1); text-align: center; border: 1px solid var(--border-color); }
.receipt-icon { font-size: 32px; margin-bottom: 6px; }
.receipt-header h2 { font-size: 18px; font-weight: 800; margin: 0; color: var(--text-primary); }
.receipt-status { font-size: 11px; font-weight: 700; color: #166534; background: #dcfce7; padding: 3px 10px; border-radius: 12px; display: inline-block; margin-top: 6px; }
.receipt-amount { font-size: 26px; font-weight: 800; color: var(--text-primary); margin: 18px 0 14px 0; }
.receipt-divider { border-top: 1px dashed var(--border-color); margin-bottom: 14px; }
.receipt-row { display: flex; justify-content: space-between; margin-bottom: 10px; font-size: 12px; }
.r-label { color: var(--text-secondary); font-weight: 500; }
.r-value { color: var(--text-primary); font-weight: 700; }
.font-mono { font-family: monospace; }

.receipt-actions { display: flex; gap: 10px; margin-top: 16px; }
.receipt-btn { flex: 1; padding: 10px; border-radius: 12px; font-size: 11px; font-weight: 700; border: none; cursor: pointer; }
.receipt-btn.secondary { background: var(--bg-color); color: var(--text-primary); border: 1px solid var(--border-color); }
.receipt-btn.primary { background: #2563eb; color: #ffffff; }

.close-receipt-btn { width: 100%; background: #f1f5f9; color: #334155; border: none; padding: 12px; border-radius: 12px; font-weight: 700; margin-top: 10px; cursor: pointer; }
.dark-mode .close-receipt-btn { background: #1e293b; color: #f8fafc; }
.text-center { text-align: center; }

/* PRINT STYLES */
@media print {
  .no-print { display: none !important; }
  body { background: #ffffff !important; }
  .modal-overlay { position: static; background: none; }
  .receipt-card { box-shadow: none; border: 1px solid #000; width: 100%; max-width: 100%; }
}
</style>
