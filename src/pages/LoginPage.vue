<!-- ACTIVITY SOLUTION START -->
<script setup>

    import { watch, ref, onBeforeMount } from 'vue';
    import { Notyf } from 'notyf'; 
    import { useGlobalStore } from '../stores/global'; // <<
    import { useRouter } from 'vue-router'; // <<
    import axios from 'axios';

    const router = useRouter() // <<
    const {getUserDetails, user} = useGlobalStore(); // <<
   

    const email = ref("");
    const password = ref("");
    const isEnabled = ref(false);

    const notyf = new Notyf();

    watch([email,password], (currentValue, oldValue) => {

        if(currentValue.every(input => input !== "")){
            isEnabled.value = true
        } else {
            isEnabled.value = false
        }
    });

        async function handleSubmit(e){
            e.preventDefault();

            try {
                    let res = await axios({
                        method: 'post',
                        url: 'https://coursebookingapi.onrender.com/users/login',
                        data: {
                            email: email.value,
                            password: password.value
                        }
                    });

                    console.log(res);


                    if(res.data){

                        notyf.success("Login Successful");
                        localStorage.setItem("token", res.data.access);   

                        // localStorage.setItem("email", email.value);


                        
                        // update global store with token
                        getUserDetails(res.data.access);

                        email.value = "";
                        password.value = "";

                        router.push({ path: '/courses' });
                    }
                }
            catch(err){
                    
                // Add a check to see if the error is a 404, 401, or 400 status code.
                // If it is, use the notyf.error() method and pass the error message to notify the user of the failed login.
                // These status codes are defined in the our backend API. Any other error code will be considered an unexpected error. 
                if(err.response.status === 404 || err.response.status === 401 || err.response.status === 400){
                    notyf.error(err.response.data.message);
                } else {
                    notyf.error("Login Failed. Please contact administrator.");
                }


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
                <h1 class="text-dark mb-5">Login Page</h1> 
	            <form v-on:submit="handleSubmit">
	                <div class="mb-3">
	                    <input type="email" class="form-control rounded-0" id="emailInput" placeholder="Email Address" v-model="email" />
	                </div>
	                <div class="mb-3">
	                    <input type="password" class="form-control rounded-0" id="passwordInput" placeholder="Password" v-model="password" />
	                </div>
	                <div class="d-grid mt-3">
	                	<button type="submit" class="btn btn-primary btn-block rounded-0"  v-if="isEnabled">Login</button>
                		<button type="submit" class="btn btn-primary btn-block rounded-0" disabled v-else>Login</button>
	                </div>
	            </form>
	        </div>
	    </div>
    </div>
</template>
<!-- ACTIVITY SOLUTION END -->