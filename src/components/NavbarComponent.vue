<script setup>
import {onBeforeMount, ref, watch} from "vue";
import { useGlobalStore } from "../stores/global";

const {user} = useGlobalStore();

console.log(user.email);

const email = ref("")

</script>

<template>
  <nav class="navbar navbar-expand-lg sticky-top bg-light shadow-sm">
    <div class="container">

    <router-link
      :to="user.email ? { name: 'Courses' } : { name: 'Home' }"
      class="navbar-brand d-flex align-items-center fw-bold"
    >
      <img src="/images/logo.png" alt="DevAcademy Logo" width="40" height="40" class="me-2" />
      <span class="text-dark">Dev/Academy</span>
    </router-link>

      <button
        class="navbar-toggler"
        type="button"
        data-bs-toggle="collapse"
        data-bs-target="#navbarNavAltMarkup"
        aria-controls="navbarNavAltMarkup"
        aria-expanded="false"
        aria-label="Toggle navigation"
      >
        <span class="navbar-toggler-icon"></span>
      </button>

      <div class="collapse navbar-collapse justify-content-lg-end" id="navbarNavAltMarkup">
        <div class="navbar-nav text-center text-lg-start d-flex flex-column flex-lg-row align-items-lg-center gap-2 gap-lg-3">

          <span class="fw-semibold text-danger px-2 py-1">
            👤 
            <span v-if="user.email">
              {{ user.isAdmin 
                  ? `Welcome Admin (${user.email})` 
                  : `Welcome User (${user.email})` 
              }}
            </span>
            <span v-else>
              Guest
            </span>
          </span>

          <router-link :to="{ name: 'Courses' }" class="nav-link" v-if="user.isAdmin">
            <i class="bi bi-speedometer2 me-1"></i> Dashboard
          </router-link>

          <router-link :to="{ name: 'Courses' }" class="nav-link" v-else>
            <i class="bi bi-journal-bookmark me-1"></i> Courses
          </router-link>

          <router-link :to="{ name: 'Register' }" class="nav-link" v-if="!user.email">
            <i class="bi bi-person-plus me-1"></i> Register
          </router-link>

          <router-link :to="{ name: 'Add Course' }" class="nav-link" v-if="user.isAdmin">
            <i class="bi bi-plus-circle me-1"></i> Add Course
          </router-link>

          <router-link :to="{ name: 'Profile' }" class="nav-link" v-if="user.email">
            <i class="bi bi-person-circle me-1"></i> Profile
          </router-link>

          <router-link :to="{ name: 'Login' }" class="nav-link" v-if="!user.email">
            <i class="bi bi-box-arrow-in-right me-1"></i> Login
          </router-link>

          <router-link :to="{ name: 'Logout' }" class="nav-link" v-else>
            <i class="bi bi-box-arrow-right me-1"></i> Logout
          </router-link>

        </div>
      </div>

    </div>
  </nav>
</template>

<style scoped>
.navbar .nav-link:hover {
  color: #0d6efd;
  transform: translateY(-1px);
  font-weight: 500;
}

.router-link-exact-active {
  color: #0d6efd !important;
  font-weight: 500;
}

.navbar-brand[style*="pointer-events:none"] {
  cursor: default;
}

.navbar-nav {
  gap: 6px;
}

@media (max-width: 991px) {
  .navbar-nav {
    padding-top: 10px;
  }

  .navbar .nav-link {
    padding: 10px 0;
    border-bottom: 1px solid #eee;
  }

  .navbar .nav-link:last-child {
    border-bottom: none;
  }
}
</style>
