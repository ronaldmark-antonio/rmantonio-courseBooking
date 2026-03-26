<script setup>
    import { watch, ref, onBeforeMount } from 'vue';
    import { Notyf } from 'notyf';
    import { useRouter } from 'vue-router';
    import { useGlobalStore } from '../stores/global.js';
    import axios from 'axios';

    const firstName = ref("");
    const lastName = ref("");
    const mobileNum = ref("");
    const email = ref("");
    const password = ref("");
    const confirmPass = ref("");
    const isEnabled = ref(false)
    const showPassword = ref(false);
    const showConfirmPassword = ref(false);

    const notyf = new Notyf();
    const router = useRouter()
    const {user} = useGlobalStore();

    const isSubmitting = ref(false);

    onBeforeMount(() => {
        if(user.email){
            router.push({path: '/courses'})
        }
    })

    watch([ email, password,confirmPass], (currentValue, oldValue) => {

        if(currentValue.every(input => input) && currentValue[1] === currentValue[2]){
            isEnabled.value = true
        } else {
            isEnabled.value = false
        }
    })

    async function handleSubmit(e){
        e.preventDefault();

        if(isSubmitting.value) return;

        isSubmitting.value = true;

        try{

            let response = await axios.post(
                'https://coursebookingapi.onrender.com/users/register',
                {
                    firstName: firstName.value,
                    lastName: lastName.value,
                    email: email.value,
                    mobileNo: mobileNum.value,
                    password: password.value
                }
                );

            if(response.status === 201){

                notyf.success(response.data.message);

                firstName.value = "";
                lastName.value = "";
                mobileNum.value = "";
                email.value = "";
                password.value = "";
                confirmPass.value = "";

                router.push({path: '/login'});
            }

        }
        catch(err){

            if(err.response && err.response.status === 409){
                notyf.error("Email Address already exists");
            }
            else if(err.response && err.response.data.message){
                notyf.error(err.response.data.message);
            }
            else {
                notyf.error("Register Failed. Please contact administrator");
            }

        }
        finally{
            isSubmitting.value = false;
        }
    }

    function handleMobileInput() {
        mobileNum.value = mobileNum.value.replace(/[^0-9]/g, '');
    }

</script>

<template>
    <div class="container-fluid">
        <div class="row d-flex justify-content-center p-5">
            <div class="col-md-5 border border rounded-0 mx-auto p-5">
                <div class="d-flex justify-content-center align-items-center gap-2 mb-5">
                  <i class="bi bi-person-plus text-dark" style="font-size: 2rem;"></i>
                  <h1 class="text-dark m-0">Register</h1>
                </div>
                <form v-on:submit="handleSubmit">
                    <div class="mb-3 position-relative">
                      <i class="bi bi-person position-absolute top-50 start-0 translate-middle-y ms-3 text-muted"></i>
                      <input 
                      type="text" 
                      class="form-control rounded-0 ps-5" 
                      placeholder="Firstname" 
                      v-model="firstName" 
                      />
                  </div>
                  <div class="mb-3 position-relative">
                      <i class="bi bi-person-badge position-absolute top-50 start-0 translate-middle-y ms-3 text-muted"></i>
                      <input 
                      type="text" 
                      class="form-control rounded-0 ps-5" 
                      placeholder="Lastname" 
                      v-model="lastName" 
                      />
                  </div>
                  <div class="mb-3 position-relative">
                      <i class="bi bi-phone position-absolute top-50 start-0 translate-middle-y ms-3 text-muted"></i>
                      <input
                      type="text"
                      class="form-control rounded-0 ps-5"
                      placeholder="Mobile Number"
                      v-model="mobileNum"
                      maxlength="11"
                      @input="handleMobileInput"
                      />
                  </div>
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
                      <i class="bi bi-lock position-absolute top-50 start-0 translate-middle-y ms-3 text-muted"></i>

                      <input
                      :type="showPassword ? 'text' : 'password'"
                      class="form-control rounded-0 ps-5 pe-5"
                      placeholder="Password"
                      v-model="password"
                      />

                      <i
                      :class="showPassword ? 'bi bi-eye-slash' : 'bi bi-eye'"
                      class="position-absolute top-50 end-0 translate-middle-y me-3"
                      style="cursor: pointer;"
                      @click="showPassword = !showPassword"
                      ></i>
                  </div>

                  <div class="mb-3 position-relative">
                      <i class="bi bi-shield-lock position-absolute top-50 start-0 translate-middle-y ms-3 text-muted"></i>

                      <input
                      :type="showConfirmPassword ? 'text' : 'password'"
                      class="form-control rounded-0 ps-5 pe-5"
                      placeholder="Confirm Password"
                      v-model="confirmPass"
                      />

                      <i
                      :class="showConfirmPassword ? 'bi bi-eye-slash' : 'bi bi-eye'"
                      class="position-absolute top-50 end-0 translate-middle-y me-3"
                      style="cursor: pointer;"
                      @click="showConfirmPassword = !showConfirmPassword"
                      ></i>
                  </div>

                  <div class="d-grid mt-3">
                      <button
                      type="submit"
                      class="btn btn-primary rounded-0 d-flex align-items-center justify-content-center gap-2"
                      :disabled="!isEnabled || isSubmitting"
                      >
                      <span v-if="isSubmitting">
                          <i class="bi bi-arrow-repeat spin"></i> Registerring...
                      </span>
                      <span v-else>
                          <i class="bi bi-person-check"></i> Register
                      </span>
                  </button>
              </div>
          </form>
          
          <p class="text-center p-3 mb-1">
              Already have an account? 
              <RouterLink to="/login" class="text-primary">Click here</RouterLink> to log in.
          </p>
      </div>
  </div>
</div>
</template>