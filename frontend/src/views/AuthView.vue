<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()

const isLogin = ref(true)
const message = ref('')
const isError = ref(false)
const isDarkMode = ref(true) // Default to Dark Mode

// Login Data
const loginData = ref({
  username: '',
  password: ''
})

// Register Data
const registerData = ref({
  name: '',
  email: '',
  username: '',
  password: ''
})

const toggleMode = () => {
  isLogin.value = !isLogin.value
  message.value = ''
  isError.value = false
}

const toggleTheme = () => {
  isDarkMode.value = !isDarkMode.value
}

const handleLogin = async () => {
  try {
    const response = await fetch(`${import.meta.env.VITE_API_URL}/api/login`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(loginData.value)
    })
    if (response.ok) {
        const userData = await response.json()
        localStorage.setItem('user', JSON.stringify(userData))
        
        isError.value = false
        message.value = 'Login successful! Redirecting...'
        
        setTimeout(() => {
            router.push('/profile')
        }, 1000)
    } else {
        const text = await response.text()
        isError.value = true
        message.value = text || 'Login failed'
    }
  } catch (error) {
    isError.value = true
    message.value = 'Error: ' + error.message
  }
}

const handleRegister = async () => {
  try {
    const response = await fetch(`${import.meta.env.VITE_API_URL}/api/users`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(registerData.value)
    })
    
    if (response.ok) {
        isError.value = false
        message.value = 'User created successfully! Redirecting to login...'
        registerData.value = { name: '', email: '', username: '', password: '' }
        setTimeout(() => {
            isLogin.value = true
            message.value = ''
        }, 1500)
    } else {
        isError.value = true
        message.value = 'Registration failed'
    }
  } catch (error) {
    isError.value = true
    message.value = 'Error: ' + error.message
  }
}
</script>

<template>
  <div :class="['app-container', isDarkMode ? 'dark' : 'light']">
    <button class="theme-toggle" @click="toggleTheme">
      {{ isDarkMode ? '☀️ Light Mode' : '🌙 Dark Mode' }}
    </button>

    <div class="card">
      <div class="header">
        <h1>{{ isLogin ? 'Welcome Back' : 'Create Account' }}</h1>
        <p>{{ isLogin ? 'Please login to continue' : 'Sign up to get started' }}</p>
      </div>

      <div class="form-container">
        <!-- Login Form -->
        <form v-if="isLogin" @submit.prevent="handleLogin" class="auth-form">
          <div class="input-group">
            <label>Username</label>
            <input type="text" v-model="loginData.username" placeholder="Enter your username" required />
          </div>
          <div class="input-group">
            <label>Password</label>
            <input type="password" v-model="loginData.password" placeholder="Enter your password" required />
          </div>
          <button type="submit" class="primary-btn">Login</button>
        </form>

        <!-- Register Form -->
        <form v-else @submit.prevent="handleRegister" class="auth-form">
          <div class="input-group">
            <label>Full Name</label>
            <input type="text" v-model="registerData.name" placeholder="John Doe" required />
          </div>
          <div class="input-group">
            <label>Email</label>
            <input type="email" v-model="registerData.email" placeholder="john@example.com" required />
          </div>
          <div class="input-group">
            <label>Username</label>
            <input type="text" v-model="registerData.username" placeholder="Choose a username" required />
          </div>
          <div class="input-group">
            <label>Password</label>
            <input type="password" v-model="registerData.password" placeholder="Choose a password" required />
          </div>
          <button type="submit" class="primary-btn">Sign Up</button>
        </form>
      </div>

      <div class="footer">
        <p>
          {{ isLogin ? "Don't have an account?" : "Already have an account?" }}
          <a href="#" @click.prevent="toggleMode">{{ isLogin ? 'Sign up' : 'Login' }}</a>
        </p>
      </div>

      <div v-if="message" :class="['message', isError ? 'error' : 'success']">
        {{ message }}
      </div>
    </div>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');

/* Base Styles */
.app-container {
  min-height: 100vh;
  display: flex; /* Flexbox for centering */
  flex-direction: column;
  align-items: center;
  justify-content: center;
  font-family: 'Inter', sans-serif;
  padding: 20px;
  transition: background-color 0.3s ease, color 0.3s ease;
  position: relative;
}

.card {
  display: flex; /* Flexbox for card layout */
  flex-direction: column;
  padding: 2.5rem;
  border-radius: 16px;
  width: 90%; /* Responsive width */
  max-width: 400px; /* Maximum width */
  transition: background-color 0.3s ease, box-shadow 0.3s ease, transform 0.3s ease;
}

/* Form Flex Layout */
.auth-form {
  display: flex;
  flex-direction: column;
  gap: 1.25rem; /* Space between form elements */
}

.input-group {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

/* Light Mode */
.light {
  background-color: #f3f4f6;
  color: #1f2937;
}

.light .card {
  background-color: #ffffff;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
  border: 1px solid #e5e7eb;
}

.light .header h1 { color: #111827; }
.light .header p { color: #6b7280; }
.light label { color: #374151; }
.light input { 
  background-color: #ffffff; 
  border: 1px solid #d1d5db; 
  color: #1f2937;
}
.light .footer { color: #6b7280; }
.light .footer a { color: #2563eb; }

/* Dark Mode */
.dark {
  background-color: #111827;
  color: #f9fafb;
}

.dark .card {
  background-color: #1f2937;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.5);
  border: 1px solid #374151;
}

.dark .header h1 { color: #f9fafb; }
.dark .header p { color: #9ca3af; }
.dark label { color: #d1d5db; }
.dark input { 
  background-color: #374151; 
  border: 1px solid #4b5563; 
  color: #f9fafb;
}
.dark input:focus {
  border-color: #60a5fa;
  outline: none;
}
.dark .footer { color: #9ca3af; }
.dark .footer a { color: #60a5fa; }

/* Common Elements */
.theme-toggle {
  position: absolute;
  top: 20px;
  right: 20px;
  background: none;
  border: 1px solid currentColor;
  padding: 8px 16px;
  border-radius: 20px;
  cursor: pointer;
  font-weight: 500;
  opacity: 0.7;
  transition: opacity 0.2s;
  color: inherit;
}

.theme-toggle:hover {
  opacity: 1;
}

.header {
  text-align: center;
  margin-bottom: 2rem;
}

.header h1 {
  font-size: 1.8rem;
  font-weight: 700;
  margin-bottom: 0.5rem;
}

.header p {
  font-size: 0.95rem;
}

.input-group label {
  font-size: 0.9rem;
  font-weight: 500;
}

.input-group input {
  width: 100%;
  padding: 0.75rem 1rem;
  border-radius: 8px;
  font-size: 1rem;
  transition: all 0.2s;
  box-sizing: border-box;
}

.primary-btn {
  width: 100%;
  padding: 0.85rem;
  background-color: #2563eb; /* Blue-600 */
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.2s;
}

.primary-btn:hover {
  background-color: #1d4ed8; /* Blue-700 */
}

.footer {
  margin-top: 1.5rem;
  text-align: center;
  font-size: 0.9rem;
}

.footer a {
  text-decoration: none;
  font-weight: 600;
}

.footer a:hover {
  text-decoration: underline;
}

.message {
  margin-top: 1.5rem;
  padding: 0.75rem;
  border-radius: 8px;
  text-align: center;
  font-size: 0.9rem;
  font-weight: 500;
}

.success {
  background-color: rgba(16, 185, 129, 0.1);
  color: #059669;
  border: 1px solid #059669;
}

.error {
  background-color: rgba(239, 68, 68, 0.1);
  color: #dc2626;
  border: 1px solid #dc2626;
}

/* Responsive Adjustments */
@media (max-width: 480px) {
  .card {
    padding: 1.5rem;
    width: 95%;
  }
  
  .header h1 {
    font-size: 1.5rem;
  }
}
</style>
