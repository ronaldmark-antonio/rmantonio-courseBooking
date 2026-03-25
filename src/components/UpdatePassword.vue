<script setup>
import { onBeforeMount, ref, computed } from 'vue'
import { useRouter } from 'vue-router'
import { Notyf } from 'notyf'
import api from '../api'
import { useGlobalStore } from '../stores/global'

const router = useRouter()
const notyf = new Notyf()
const { user } = useGlobalStore()

const firstName = ref('')
const lastName = ref('')
const mobileNo = ref('')
const email = ref('')
const isAdmin = ref(false) // boolean for admin

const newPassword = ref('')
const confirmPassword = ref('')

const showPassword = ref(false);
const showConfirmPassword = ref(false);

const isSubmitDisabled = computed(() => {
  return !newPassword.value || !confirmPassword.value
})

const loading = ref(false)

const handleReset = async () => {
  if (newPassword.value !== confirmPassword.value) {
    notyf.error("Passwords do not match")
    return
  }

  const token = user.token || localStorage.getItem('token')
  if (!token) {
    notyf.error('You are not authorized')
    return
  }

  loading.value = true
  try {
    const response = await api.post(
      '/users/reset-password',
      { newPassword: newPassword.value },
      { headers: { Authorization: `Bearer ${token}` } }
    )

    notyf.success(response.data.message || 'Password reset successfully')

    newPassword.value = ''
    confirmPassword.value = ''

    localStorage.removeItem('token')
    user.token = null
    user.email = null
    user.firstName = null
    user.lastName = null

    router.push('/login')

  } catch (err) {
      const msg = err.response?.data?.message || 'Password must be at least 8 characters'
      notyf.error(msg)
  } finally {
      loading.value = false
  }
}

onBeforeMount(async () => {
  const token = user.token || localStorage.getItem('token')

  if (!token) {
    router.push('/login')
    return
  }

  loading.value = true

  try {
    const { data } = await api.get('/users/details', {
      headers: { Authorization: `Bearer ${token}` },
    })

    firstName.value = data.firstName || ''
    lastName.value = data.lastName || ''
    mobileNo.value = data.mobileNo || ''
    email.value = data.email || ''
    isAdmin.value = data.isAdmin || false

  } catch (err) {
    console.error(err)
    notyf.error(err.response?.data?.message || 'Failed to load profile info.')
  } finally {
    loading.value = false
  }
})
</script>

<template>
<div class="container-fluid">
  <div class="row d-flex justify-content-center p-5">

    <div class="col-md-5 border border rounded-0 mx-auto p-5">

      <!-- TITLE -->
      <h1 class="text-dark mb-4 d-flex align-items-center gap-2">
        Profile Details
        <span 
          class="ms-2 badge" 
          :class="isAdmin ? 'bg-dark' : 'bg-success'"
        >
          {{ isAdmin ? 'Admin' : 'User' }}
        </span>
      </h1>

      <!-- FIRST NAME -->
      <div class="mb-3 position-relative">
        <i class="bi bi-person position-absolute top-50 start-0 translate-middle-y ms-3 text-muted"></i>
        <input
          type="text"
          class="form-control rounded-0 ps-5"
          :value="firstName"
          disabled
        />
      </div>

      <!-- LAST NAME -->
      <div class="mb-3 position-relative">
        <i class="bi bi-person-badge position-absolute top-50 start-0 translate-middle-y ms-3 text-muted"></i>
        <input
          type="text"
          class="form-control rounded-0 ps-5"
          :value="lastName"
          disabled
        />
      </div>

      <!-- MOBILE NUMBER -->
      <div class="mb-3 position-relative">
        <i class="bi bi-phone position-absolute top-50 start-0 translate-middle-y ms-3 text-muted"></i>
        <input
          type="text"
          class="form-control rounded-0 ps-5"
          :value="mobileNo"
          disabled
        />
      </div>

      <!-- EMAIL -->
      <div class="mb-4 position-relative">
        <i class="bi bi-envelope position-absolute top-50 start-0 translate-middle-y ms-3 text-muted"></i>
        <input
          type="email"
          class="form-control rounded-0 ps-5"
          :value="email"
          disabled
        />
      </div>

      <!-- RESET PASSWORD TITLE -->
      <h4 class="mb-3 d-flex align-items-center gap-2">
        <i class="bi bi-shield-lock"></i> Reset Password
      </h4>

      <form @submit.prevent="handleReset">

        <!-- NEW PASSWORD -->
        <div class="mb-3 position-relative">
          <i class="bi bi-lock position-absolute top-50 start-0 translate-middle-y ms-3 text-muted"></i>
          <input
            :type="showPassword ? 'text' : 'password'"
            class="form-control rounded-0 ps-5 pe-5"
            placeholder="New Password"
            v-model="newPassword"
          />
          <i
            :class="showPassword ? 'bi bi-eye-slash' : 'bi bi-eye'"
            class="position-absolute top-50 end-0 translate-middle-y me-3"
            style="cursor:pointer"
            @click="showPassword = !showPassword"
          ></i>
        </div>

        <!-- CONFIRM PASSWORD -->
        <div class="mb-3 position-relative">
          <i class="bi bi-shield-lock position-absolute top-50 start-0 translate-middle-y ms-3 text-muted"></i>
          <input
            :type="showConfirmPassword ? 'text' : 'password'"
            class="form-control rounded-0 ps-5 pe-5"
            placeholder="Confirm Password"
            v-model="confirmPassword"
          />
          <i
            :class="showConfirmPassword ? 'bi bi-eye-slash' : 'bi bi-eye'"
            class="position-absolute top-50 end-0 translate-middle-y me-3"
            style="cursor:pointer"
            @click="showConfirmPassword = !showConfirmPassword"
          ></i>
        </div>

        <!-- RESET BUTTON -->
        <div class="d-grid mt-3">
          <button
            type="submit"
            class="btn btn-primary rounded-0 d-flex align-items-center justify-content-center gap-2"
            :disabled="isSubmitDisabled || loading"
          >
            <span v-if="loading">
              <i class="bi bi-arrow-repeat spin"></i> Resetting...
            </span>
            <span v-else>
              <i class="bi bi-key"></i> Reset Password
            </span>
          </button>
        </div>

      </form>

    </div>
  </div>
</div>
</template>

<style>
.spin {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}
</style>