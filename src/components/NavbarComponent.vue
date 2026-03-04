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

      <router-link :to="{ name: 'Home' }" class="navbar-brand d-flex align-items-center fw-bold">
        <img src="/images/logo.png" alt="DevAcademy Logo" width="40" height="40" class="me-2" />
        <span class="text-dark">DevAcademy</span>
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

      <div class="collapse navbar-collapse" id="navbarNavAltMarkup">
        <div class="navbar-nav ms-auto text-center text-lg-start">
          <router-link :to="{ name: 'Courses' }" class="nav-link">Courses</router-link>
          <router-link :to="{ name: 'Register' }" class="nav-link" v-if="!user.email">Register</router-link>
          <router-link :to="{ name: 'Add Course' }" class="nav-link" v-if="user.isAdmin">Add Course</router-link>
          <router-link :to="{ name: 'Profile' }" class="nav-link" v-if="user.email">Profile</router-link>
          <router-link :to="{ name: 'Login' }" class="nav-link" v-if="!user.email">Login</router-link>
          <router-link :to="{ name: 'Logout' }" class="nav-link" v-else>Logout</router-link>
        </div>
      </div>

    </div>
  </nav>
</template>
