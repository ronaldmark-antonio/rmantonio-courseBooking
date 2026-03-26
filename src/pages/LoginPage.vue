<script setup>
import { watch, ref, onBeforeMount } from 'vue';
import { Notyf } from 'notyf'; 
import { useGlobalStore } from '../stores/global';
import { useRouter } from 'vue-router';
import axios from 'axios';

const router = useRouter()
const { getUserDetails, user } = useGlobalStore();

const email = ref("");
const password = ref("");
const isEnabled = ref(false);
const showPassword = ref(false);

const notyf = new Notyf();
const isLoggingIn = ref(false);

watch([email,password], (currentValue, oldValue) => {

    if(currentValue.every(input => input !== "")){
        isEnabled.value = true
    } else {
        isEnabled.value = false
    }
});

async function handleSubmit(e){
    e.preventDefault();

    if(isLoggingIn.value) return;

    isLoggingIn.value = true;

    try {
        let res = await axios({
            method: 'post',
            url: 'https://coursebookingapi.onrender.com/users/login',
            data: {
                email: email.value,
                password: password.value
            }
        });

        if(res.data){
            notyf.success("Login successfully");
            localStorage.setItem("token", res.data.access);   
            getUserDetails(res.data.access);

            email.value = "";
            password.value = "";

            router.push({ path: '/courses' });
        }

    } catch(err){

        if(err.response && 
         (err.response.status === 404 || 
            err.response.status === 401 || 
            err.response.status === 400)) {

            notyf.error(err.response.data.message);

    } else {
        notyf.error("Login Failed. Please contact administrator");
    }

} finally {
    isLoggingIn.value = false;
}
}

onBeforeMount(()=> {
    if(user.email){
        router.push({path: "/courses"})
    }
})
</script>

<template>
    <div class="container-fluid">
        <div class="row d-flex justify-content-center p-5">
            <div class="col-md-5 border border rounded-0 mx-auto p-5">
                <div class="d-flex justify-content-center align-items-center gap-2 mb-5">
                  <i class="bi bi-box-arrow-in-right text-dark" style="font-size: 2rem;"></i>
                  <h1 class="text-dark m-0">Login</h1>
                </div>
                <form v-on:submit="handleSubmit">
                    <div class="mb-3 position-relative">
                      <i class="bi bi-envelope position-absolute top-50 start-0 translate-middle-y ms-3 text-muted"></i>
                      <input 
                      type="email" 
                      class="form-control rounded-0 ps-5" 
                      placeholder="Email Address" 
                      v-model="email" 
                      />
                  </div>
                  <div class="mb-3 position-relative">
                      <!-- Left icon -->
                      <i class="bi bi-lock position-absolute top-50 start-0 translate-middle-y ms-3 text-muted"></i>

                      <!-- Input -->
                      <input
                      :type="showPassword ? 'text' : 'password'"
                      class="form-control rounded-0 ps-5 pe-5"
                      placeholder="Password"
                      v-model="password"
                      />

                      <!-- Toggle icon -->
                      <i
                      :class="showPassword ? 'bi bi-eye-slash' : 'bi bi-eye'"
                      class="position-absolute top-50 end-0 translate-middle-y me-3"
                      style="cursor: pointer;"
                      @click="showPassword = !showPassword"
                      ></i>
                  </div>
                  <div class="d-grid mt-3">
                      <button
                      type="submit"
                      class="btn btn-primary rounded-0 d-flex align-items-center justify-content-center gap-2"
                      :disabled="!isEnabled || isLoggingIn"
                      >
                      <span v-if="isLoggingIn">
                          <i class="bi bi-arrow-repeat spin"></i> Logging in...
                      </span>
                      <span v-else>
                          <i class="bi bi-box-arrow-in-right"></i> Login
                      </span>
                  </button>
              </div>
          </form>
          <p class="text-center mt-3">
              Don't have an account yet? 
              <RouterLink to="/register" class="text-primary">Click here</RouterLink> to register.
          </p>
      </div>
  </div>
</div>
</template>