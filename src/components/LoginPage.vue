<template>
  <div class="auth-page">
    <div class="aurora-bg">
      <div class="blob blob-1"></div>
      <div class="blob blob-2"></div>
    </div>

    <div class="container">
      <div class="left-section">
        <div class="branding">
          <h1 class="logo">Library<span>Sys</span></h1>
          <p class="tagline">Explore, borrow, and manage your literary world with ease.</p>
        </div>
      </div>

      <div class="right-section">
        <div class="glass-card">
          <div class="card-header">
            <h2>{{ isLogin ? 'LOGIN' : 'REGISTER' }}</h2>
            <p class="subtitle">
              {{ isLogin ? 'Welcome back! Please enter your details.' : 'Create an account to get started.' }}
            </p>
          </div>

          <div class="form-body">
            <div class="field" v-if="!isLogin">
              <input v-model="name" placeholder=" " required />
              <label>Full Name</label>
            </div>

            <div class="field">
              <input v-model="username" placeholder=" " required />
              <label>Username</label>
            </div>

            <div class="field">
              <input :type="show ? 'text' : 'password'" v-model="password" placeholder=" " required />
              <label>Password</label>
              <span class="eye-icon" @click="show = !show">{{ show ? '🙈' : '👁' }}</span>
            </div>

            <div v-if="!isLogin && password" class="strength-meter">
              <div class="bar" :style="{ width: passwordStrength + '%', backgroundColor: strengthColor }"></div>
            </div>

            <div class="options" v-if="isLogin">
              <label class="custom-checkbox">
                <input type="checkbox" v-model="remember" />
                <span class="checkmark"></span>
                Remember me
              </label>
            </div>

            <button class="primary-btn" @click="submit">
              {{ isLogin ? 'SIGN IN' : 'SIGN UP' }}
            </button>

            <transition name="fade">
              <div v-if="error" class="toast error">{{ error }}</div>
            </transition>
            <transition name="fade">
              <div v-if="success" class="toast success">{{ success }}</div>
            </transition>
          </div>

          <div class="switch-mode">
            {{ isLogin ? 'New user?' : 'Already a member?' }}
            <span @click="toggleMode">{{ isLogin ? 'Create account' : 'Login' }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const isLogin = ref(true)
const name = ref('')
const username = ref('')
const password = ref('')
const show = ref(false)
const remember = ref(false)
const error = ref('')
const success = ref('')

const users = ref(JSON.parse(localStorage.getItem('users') || '[]'))

const passwordStrength = computed(() => {
  let score = 0
  if (password.value.length >= 6) score += 30
  if (/[A-Z]/.test(password.value)) score += 20
  if (/[0-9]/.test(password.value)) score += 20
  if (/[\W]/.test(password.value)) score += 30
  return Math.min(score, 100)
})

const strengthColor = computed(() => {
  if (passwordStrength.value < 50) return '#f87171'
  if (passwordStrength.value < 80) return '#facc15'
  return '#22c55e'
})

watch(users, (val) => {
  localStorage.setItem('users', JSON.stringify(val))
}, { deep: true })

function toggleMode() {
  isLogin.value = !isLogin.value
  name.value = ''; username.value = ''; password.value = '';
  error.value = ''; success.value = '';
}

function submit() {
  error.value = ''; success.value = '';
  if (!username.value || !password.value || (!isLogin.value && !name.value)) {
    error.value = 'Please fill in all fields'; return;
  }
  if (isLogin.value) {
    const user = users.value.find(u => u.username === username.value && u.password === password.value)
    if (user) {
      success.value = `Welcome back, ${user.name}! Redirecting...`
      if (remember.value) localStorage.setItem('rememberedUser', JSON.stringify(user))
    } else { error.value = 'Invalid username or password' }
  } else {
    if (users.value.find(u => u.username === username.value)) {
      error.value = 'Username already exists'; return;
    }
    users.value.push({ name: name.value, username: username.value, password: password.value })
    success.value = 'Account successfully created!'; toggleMode();
  }
  password.value = ''
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;600;800&display=swap');

.auth-page {
  height: 100vh; width: 100vw;
  display: flex; position: relative;
  background: #020617; /* Deep Navy Base */
  font-family: 'Plus Jakarta Sans', sans-serif;
  overflow: hidden;
}

/* --- AURORA ANIMATION --- */
.aurora-bg {
  position: absolute; inset: 0; z-index: 0; overflow: hidden;
}
.blob {
  position: absolute; width: 600px; height: 600px;
  border-radius: 50%; filter: blur(100px);
  opacity: 0.15; animation: move 20s infinite alternate ease-in-out;
}
.blob-1 { background: #38bdf8; top: -10%; left: -10%; }
.blob-2 { background: #6366f1; bottom: -10%; right: -10%; animation-delay: -10s; }

@keyframes move {
  from { transform: translate(0,0) scale(1); }
  to { transform: translate(100px, 100px) scale(1.1); }
}

.container { display: flex; width: 100%; z-index: 1; }

/* --- LEFT PANEL --- */
.left-section {
  flex: 1.3; display: flex; align-items: center;
  padding-left: 8%;
  background: linear-gradient(to right, rgba(15, 23, 42, 0.8), transparent);
}
.logo { font-size: 5rem; font-weight: 800; margin: 0; letter-spacing: -3px; color: white; }
.logo span { color: #38bdf8; }
.tagline { color: #94a3b8; font-size: 1.3rem; margin-top: 15px; max-width: 400px; }

/* --- RIGHT PANEL --- */
.right-section { flex: 1; display: flex; align-items: center; justify-content: center; padding: 20px; }

.glass-card {
  width: 100%; max-width: 440px;
  padding: 50px;
  background: rgba(30, 41, 59, 0.6); /* Translucent Slate */
  backdrop-filter: blur(20px); /* Frosted Glass */
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 32px;
  box-shadow: 0 25px 50px -12px rgba(0,0,0,0.5);
}

.card-header h2 { text-align: center; font-size: 2rem; margin: 0; letter-spacing: 2px; color: white; }
.subtitle { text-align: center; color: #94a3b8; font-size: 14px; margin: 10px 0 35px; }

/* --- FLOATING LABELS --- */
.field { position: relative; margin-bottom: 25px; }
.field input {
  width: 100%; padding: 16px;
  background: rgba(15, 23, 42, 0.5);
  border: 1px solid rgba(255,255,255,0.1);
  border-radius: 16px; color: white;
  font-size: 1rem; outline: none; transition: 0.3s;
}
.field input:focus { border-color: #38bdf8; background: rgba(15, 23, 42, 0.8); }

.field label {
  position: absolute; left: 16px; top: 50%;
  transform: translateY(-50%); color: #64748b;
  pointer-events: none; transition: 0.3s; /* */
}
.field input:focus + label,
.field input:not(:placeholder-shown) + label {
  top: -12px; left: 12px;
  font-size: 12px; color: #38bdf8;
  font-weight: 700; background: #1e293b; /* Match card color */
  padding: 0 8px; border-radius: 4px;
}

.eye-icon { position: absolute; right: 16px; top: 50%; transform: translateY(-50%); cursor: pointer; }

/* --- PRIMARY BUTTON --- */
.primary-btn {
  width: 100%; padding: 16px;
  border: none; border-radius: 16px;
  background: linear-gradient(135deg, #38bdf8, #3b82f6);
  color: white; font-weight: 700; font-size: 1rem;
  cursor: pointer; transition: 0.3s;
  box-shadow: 0 10px 15px -3px rgba(59, 130, 246, 0.3);
  margin-top: 10px;
}
.primary-btn:hover { transform: translateY(-2px); box-shadow: 0 20px 25px -5px rgba(59, 130, 246, 0.4); }

.strength-meter { height: 4px; background: rgba(255,255,255,0.05); border-radius: 2px; margin: -15px 0 25px; overflow: hidden; }
.strength-meter .bar { height: 100%; transition: 0.4s; }

.switch-mode { margin-top: 25px; text-align: center; color: #94a3b8; font-size: 14px; }
.switch-mode span { color: #38bdf8; font-weight: 700; cursor: pointer; margin-left: 5px; }

.toast { margin-top: 20px; padding: 12px; border-radius: 12px; text-align: center; font-size: 13px; }
.error { background: rgba(239, 68, 68, 0.1); color: #f87171; border: 1px solid rgba(239,68,68,0.2); }
.success { background: rgba(34, 197, 94, 0.1); color: #4ade80; border: 1px solid rgba(34,197,94,0.2); }

/* Custom Checkbox */
.custom-checkbox { display: flex; align-items: center; gap: 10px; color: #94a3b8; font-size: 14px; cursor: pointer; margin-bottom: 20px; }
.custom-checkbox input { display: none; }
.checkmark { width: 18px; height: 18px; border: 1px solid #334155; border-radius: 4px; background: #0f172a; }
.custom-checkbox input:checked + .checkmark { background: #38bdf8; border-color: #38bdf8; }

.fade-enter-active, .fade-leave-active { transition: opacity 0.5s; }
.fade-enter-from, .fade-leave-to { opacity: 0; }

@media (max-width: 900px) {
  .left-section { display: none; }
  .right-section { background: #020617; width: 100%; }
}
</style>