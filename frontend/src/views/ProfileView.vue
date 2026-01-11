<script setup>
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()
const isDarkMode = ref(true) // Default to Dark Mode
const message = ref('')
const isError = ref(false)

const user = ref({
  id: null,
  name: '',
  email: '',
  username: '',
  password: '' // Only if changing
})

const toggleTheme = () => {
  isDarkMode.value = !isDarkMode.value
}

onMounted(() => {
  const storedUser = localStorage.getItem('user')
  if (storedUser) {
    const parsed = JSON.parse(storedUser)
    user.value = { ...parsed, password: '' }
  } else {
    // Not logged in, redirect
    router.push('/')
  }
})

const handleUpdate = async () => {
  try {
    const updatePayload = { ...user.value }
    // If password is empty, don't send it or handle in backend (backend checks empty)
    
    const response = await fetch(`${import.meta.env.VITE_API_URL}/api/users/${user.value.id}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(updatePayload)
    })

    if (response.ok) {
        const updatedUser = await response.json()
        // Update local storage (without password)
        const toStore = { ...updatedUser }
        delete toStore.password
        localStorage.setItem('user', JSON.stringify(toStore))
        
        user.value = { ...toStore, password: '' }
        
        isError.value = false
        message.value = 'Profile updated successfully!'
    } else {
        isError.value = true
        message.value = 'Update failed'
    }
  } catch (error) {
    isError.value = true
    message.value = 'Error: ' + error.message
  }
}

const handleLogout = () => {
  localStorage.removeItem('user')
  router.push('/')
}
</script>

<template>
  <div :class="['app-container', isDarkMode ? 'dark' : 'light']">
    <button class="theme-toggle" @click="toggleTheme">
      {{ isDarkMode ? '☀️ Light Mode' : '🌙 Dark Mode' }}
    </button>

    <div class="card">
      <div class="header">
        <h1>Profile Settings</h1>
        <p>Manage your account details</p>
      </div>

      <div class="form-container">
        <form @submit.prevent="handleUpdate" class="auth-form">
          <div class="input-group">
            <label>Full Name</label>
            <input type="text" v-model="user.name" required />
          </div>
          <div class="input-group">
            <label>Email</label>
            <input type="email" v-model="user.email" required />
          </div>
          <div class="input-group">
            <label>Username</label>
            <input type="text" v-model="user.username" required />
          </div>
          <div class="input-group">
            <label>New Password (leave blank to keep current)</label>
            <input type="password" v-model="user.password" placeholder="New Password" />
          </div>
          
          <button type="submit" class="primary-btn">Save Changes</button>
          <button type="button" class="secondary-btn" @click="handleLogout">Logout</button>
        </form>
      </div>

      <div v-if="message" :class="['message', isError ? 'error' : 'success']">
        {{ message }}
      </div>
    </div>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');

/* Reusing styles from AuthView for consistency */
.app-container {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  font-family: 'Inter', sans-serif;
  padding: 20px;
  transition: background-color 0.3s ease, color 0.3s ease;
  position: relative;
}

.card {
  display: flex;
  flex-direction: column;
  padding: 2.5rem;
  border-radius: 16px;
  width: 90%;
  max-width: 400px;
  transition: background-color 0.3s ease, box-shadow 0.3s ease;
}

.auth-form {
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
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
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
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
  box-sizing: border-box;
}

.primary-btn {
  width: 100%;
  padding: 0.85rem;
  background-color: #2563eb;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
}

.primary-btn:hover {
  background-color: #1d4ed8;
}

.secondary-btn {
  width: 100%;
  padding: 0.85rem;
  background-color: transparent;
  color: currentColor;
  border: 1px solid currentColor;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  opacity: 0.8;
}

.secondary-btn:hover {
  opacity: 1;
  background-color: rgba(128, 128, 128, 0.1);
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
</style>
