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
    const response = await api.patch(
      'https://rmantonio-coursebookingapi.onrender.com/users/update-password',
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
      <div class="col-md-5 register-card apple-shadow p-4">
        <h1 class="text-center mb-5">
          <i class="fas fa-user-circle text-success me-1"></i> Profile Details
        </h1>

        <div class="mb-2">
          <label class="form-label">First Name:</label>
          <div class="input-group">
            <input type="text" class="form-control" :value="firstName" disabled />
          </div>
        </div>

        <div class="mb-2">
          <label class="form-label">Last Name:</label>
          <div class="input-group">
            <input type="text" class="form-control" :value="lastName" disabled />
          </div>
        </div>

        <div class="mb-2">
          <label class="form-label">Mobile Number:</label>
          <div class="input-group">
            <input type="text" class="form-control" :value="mobileNo" disabled />
          </div>
        </div>

        <div class="mb-2">
          <label class="form-label">Email Address:</label>
          <div class="input-group">
            <input type="email" class="form-control" :value="email" disabled />
          </div>
        </div>

        <div class="mt-5">
          <h3 class="card-title mb-3">
            <i class="fas fa-key me-2"></i> Reset Password
          </h3>
          <form @submit.prevent="handleReset">
            <div class="mb-3">
              <label for="newPassword" class="form-label">New Password</label>
              <div class="input-group">
                <span class="input-group-text"><i class="fas fa-lock"></i></span>
                <input
                  :type="showPassword ? 'text' : 'password'"
                  id="newPassword"
                  placeholder="Enter your new password"
                  v-model="newPassword"
                  class="form-control"
                  required
                />

                <span
                  class="input-group-text password-eye"
                  @click="showPassword = !showPassword"
                >
                  <i :class="showPassword ? 'fas fa-eye-slash' : 'fas fa-eye'"></i>
                </span>
              </div>
            </div>

            <div class="mb-3">
              <label for="confirmPassword" class="form-label">Confirm Password</label>
              <div class="input-group">
                <span class="input-group-text"><i class="fas fa-lock"></i></span>
                <input
                  :type="showConfirmPassword ? 'text' : 'password'"
                  id="confirmPassword"
                  placeholder="Confirm your new password"
                  v-model="confirmPassword"
                  class="form-control"
                  required
                />

                <span
                  class="input-group-text password-eye"
                  @click="showConfirmPassword = !showConfirmPassword"
                >
                  <i :class="showConfirmPassword ? 'fas fa-eye-slash' : 'fas fa-eye'"></i>
                </span>
              </div>
            </div>

            <button
              type="submit"
              class="btn btn-success w-100"
              :disabled="isSubmitDisabled || loading"
            >
              <span v-if="!loading">
                <i class="fas fa-check-circle me-2"></i> Submit
              </span>
              <span v-else class="d-flex align-items-center justify-content-center gap-2">
                <span class="spinner-border spinner-border-sm" role="status" aria-hidden="true"></span>
                Submitting...
              </span>
            </button>

          </form>
        </div>

      </div>
    </div>
  </div>
</template>