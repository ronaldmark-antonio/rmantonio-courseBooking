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

      <h1 class="text-dark mb-4">Profile Details</h1>

      <div class="mb-3">
        <input
          type="text"
          class="form-control rounded-0"
          placeholder="First Name"
          :value="firstName"
          disabled
        />
      </div>

      <div class="mb-3">
        <input
          type="text"
          class="form-control rounded-0"
          placeholder="Last Name"
          :value="lastName"
          disabled
        />
      </div>

      <div class="mb-3">
        <input
          type="text"
          class="form-control rounded-0"
          placeholder="Mobile Number"
          :value="mobileNo"
          disabled
        />
      </div>

      <div class="mb-4">
        <input
          type="email"
          class="form-control rounded-0"
          placeholder="Email Address"
          :value="email"
          disabled
        />
      </div>

      <h4 class="mb-3">Reset Password</h4>

      <form @submit.prevent="handleReset">

        <!-- NEW PASSWORD -->
        <div class="mb-3 position-relative">
          <input
            :type="showPassword ? 'text' : 'password'"
            class="form-control rounded-0 pe-5"
            placeholder="New Password"
            v-model="newPassword"
          />

          <i
            :class="showPassword ? 'bi bi-eye-slash' : 'bi bi-eye'"
            class="position-absolute top-50 end-0 translate-middle-y me-3"
            style="cursor:pointer;font-size:1.2rem"
            @click="showPassword = !showPassword"
          ></i>
        </div>

        <!-- CONFIRM PASSWORD -->
        <div class="mb-3 position-relative">
          <input
            :type="showConfirmPassword ? 'text' : 'password'"
            class="form-control rounded-0 pe-5"
            placeholder="Confirm Password"
            v-model="confirmPassword"
          />

          <i
            :class="showConfirmPassword ? 'bi bi-eye-slash' : 'bi bi-eye'"
            class="position-absolute top-50 end-0 translate-middle-y me-3"
            style="cursor:pointer;font-size:1.2rem"
            @click="showConfirmPassword = !showConfirmPassword"
          ></i>
        </div>

        <div class="d-grid mt-3">
          <button
            type="submit"
            class="btn btn-primary btn-block rounded-0"
            :disabled="isSubmitDisabled || loading"
          >
            <span v-if="loading">Submitting...</span>
            <span v-else>Reset Password</span>
          </button>
        </div>

      </form>

    </div>
  </div>
</div>
</template>