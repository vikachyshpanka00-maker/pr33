<template>
  <div class="form-page">
    <h1>Регистрация</h1>

    <form @submit.prevent="handleSubmit" class="registration-form">
      <!-- Имя -->
      <div class="form-group">
        <label for="name">Имя</label>
        <input
          id="name"
          v-model="form.name"
          type="text"
          class="form-input"
          :class="{ 'input-error': errors.name }"
          @blur="validateField('name')"
        />
        <span v-if="errors.name" class="error-message">{{ errors.name }}</span>
      </div>

      <!-- Email -->
      <div class="form-group">
        <label for="email">Email</label>
        <input
          id="email"
          v-model="form.email"
          type="email"
          class="form-input"
          :class="{ 'input-error': errors.email }"
          @blur="validateField('email')"
        />
        <span v-if="errors.email" class="error-message">{{ errors.email }}</span>
      </div>

      <!-- Пароль -->
      <div class="form-group">
        <label for="password">Пароль</label>
        <input
          id="password"
          v-model="form.password"
          type="password"
          class="form-input"
          :class="{ 'input-error': errors.password }"
          @blur="validateField('password')"
        />
        <span v-if="errors.password" class="error-message">{{ errors.password }}</span>
      </div>

      <!-- Подтверждение пароля -->
      <div class="form-group">
        <label for="confirmPassword">Подтверждение пароля</label>
        <input
          id="confirmPassword"
          v-model="form.confirmPassword"
          type="password"
          class="form-input"
          :class="{ 'input-error': errors.confirmPassword }"
          @blur="validateField('confirmPassword')"
        />
        <span v-if="errors.confirmPassword" class="error-message">{{ errors.confirmPassword }}</span>
      </div>

      <!-- Общее сообщение об ошибке -->
      <div v-if="generalError" class="general-error">{{ generalError }}</div>

      <!-- Кнопка отправки -->
      <button type="submit" class="btn-submit" :disabled="isLoading">
        {{ isLoading ? 'Отправка...' : 'Зарегистрироваться' }}
      </button>
    </form>

    <!-- Сообщение об успехе -->
    <div v-if="successMessage" class="success-message">
      {{ successMessage }}
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import axios from 'axios'

const isLoading = ref(false)
const successMessage = ref('')
const generalError = ref(null)

// Данные формы
const form = reactive({
  name: '',
  email: '',
  password: '',
  confirmPassword: ''
})

// Ошибки валидации
const errors = reactive({
  name: null,
  email: null,
  password: null,
  confirmPassword: null
})

// Функция валидации одного поля (живая валидация)
const validateField = (field) => {
  if (field === 'name') {
    errors.name = !form.name || form.name.trim().length < 2 
      ? 'Имя должно быть не короче 2 символов.' 
      : null
  }
  
  if (field === 'email') {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
    errors.email = !form.email || !emailRegex.test(form.email) 
      ? 'Введите корректный email.' 
      : null
  }

  if (field === 'password') {
    // Проверка: 8 символов, строчные, прописные, цифры и спецсимволы
    const passwordRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[!@#$%^&*])[A-Za-z\d!@#$%^&*]{8,}$/
    if (!form.password) {
      errors.password = 'Пароль обязателен.'
    } else if (!passwordRegex.test(form.password)) {
      errors.password = 'Пароль должен содержать минимум 8 символов, включая заглавные, строчные буквы, цифры и спецсимволы.'
    } else {
      errors.password = null
    }
    if (form.confirmPassword) validateField('confirmPassword')
  }

  if (field === 'confirmPassword') {
    if (form.password !== form.confirmPassword) {
      errors.confirmPassword = 'Пароли не совпадают.'
    } else if (!form.confirmPassword && form.password) {
      errors.confirmPassword = 'Подтвердите пароль.'
    } else {
      errors.confirmPassword = null
    }
  }
}

// Общая валидация перед отправкой
const validate = () => {
  validateField('name')
  validateField('email')
  validateField('password')
  validateField('confirmPassword')
  return !Object.values(errors).some(err => err !== null)
}

// Отправка формы
const handleSubmit = async () => {
  successMessage.value = ''
  generalError.value = null

  if (!validate()) return

  isLoading.value = true

  try {
    const payload = {
      name: form.name.trim(),
      email: form.email.trim(),
      password: form.password,
    }

    // Отправка через Axios
    const response = await axios.post('https://jsonplaceholder.typicode.com/users', payload)
    successMessage.value = 'Регистрация успешно отправлена (демо).'
  } catch (err) {
    // Обработка ошибок Axios
    if (err.response) {
      generalError.value = `Ошибка сервера: ${err.response.status}`
    } else if (err.request) {
      generalError.value = 'Ошибка сети: Сервер не отвечает.'
    } else {
      generalError.value = 'Произошла ошибка при отправке.'
    }
  } finally {
    isLoading.value = false
  }
}
</script>

<style scoped>
.form-page {
  max-width: 420px;
  margin: 60px auto;
  padding: 40px;
  background-color: #fafafa;
  border-radius: 16px;
  border: 1px solid #e0e0e0;
  font-family: 'Inter', 'Segoe UI', sans-serif;
}

h1 {
  text-align: center;
  font-size: 24px;
  font-weight: 500;
  color: #1a1a1a;
  margin-bottom: 30px;
  letter-spacing: -0.5px;
}

.registration-form {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.form-group {
  display: flex;
  flex-direction: column;
}

label {
  font-size: 13px;
  color: #666;
  margin-bottom: 8px;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.form-input {
  padding: 10px 0;
  border: none;
  border-bottom: 1.5px solid #ccc;
  background: transparent;
  font-size: 15px;
  color: #333;
  transition: border-color 0.3s ease;
}

.form-input:focus {
  outline: none;
  border-bottom-color: #4a90e2;
}

.form-input.input-error {
  border-bottom-color: #e74c3c;
  background-color: transparent;
}

.error-message {
  color: #e74c3c;
  font-size: 12px;
  margin-top: 6px;
}

.general-error {
  color: #e74c3c;
  font-size: 14px;
  text-align: center;
  margin-top: -10px;
}

.btn-submit {
  width: 100%;
  padding: 14px;
  background-color: #1a1a1a;
  color: #ffffff;
  border: none;
  border-radius: 8px;
  font-size: 15px;
  font-weight: 500;
  cursor: pointer;
  transition: background-color 0.2s ease, opacity 0.2s ease;
  margin-top: 10px;
}

.btn-submit:hover:not(:disabled) {
  background-color: #333;
}

.btn-submit:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.success-message {
  margin-top: 30px;
  color: #2e7d32;
  font-size: 14px;
  text-align: center;
  padding: 10px;
  border-top: 1px solid #c8e6c9;
}
</style>