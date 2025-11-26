<script setup>
     /* ACTIVITY SOLUTION START */
    //import both watch and ref hooks     
    import { watch, ref, onBeforeMount } from 'vue';
    //import Notyf to allow the use of an alternative notification window.
    import { Notyf } from 'notyf';
    import { useRouter } from 'vue-router';
    import { useGlobalStore } from '../stores/global.js';
    import axios from 'axios';
    /* ACTIVITY SOLUTION END */

    //Create a reactive state for each user input
    //We will bind the input to our reactive state

    const firstName = ref("");
    const lastName = ref("");
    const mobileNum = ref("");
    const email = ref("");
    const password = ref("");
    const confirmPass = ref("");
    const isEnabled = ref(false)

    const notyf = new Notyf();

    const router = useRouter()

    const {user} = useGlobalStore();

    
     /* ACTIVITY SOLUTION START */
    onBeforeMount(() => {
        if(user.email){
            router.push({path: '/courses'})
        }
    })

    watch([ email, password,confirmPass], (currentValue, oldValue) => {

        //console.log(currentValue);

        if(currentValue.every(input => input) && currentValue[1] === currentValue[2]){
            isEnabled.value = true
        } else {
            isEnabled.value = false
        }
    })

    async function handleSubmit(e){

        console.log("submitted");

        e.preventDefault();

        try{

            // await axios.post("http://localhost:4000/users/checkEmail", {
            //     email: email.value
            // })]
            console.log("trying...")

            let response = await axios.post('https://coursebookingapi.onrender.com/users/register', {
                            firstName: firstName.value,
                            lastName: lastName.value,
                            email: email.value,
                            mobileNo: mobileNum.value,
                            password: password.value
                        })

            console.log(response);

            if(response.status === 201){ // 201 - CREATED

                notyf.success(response.data.message);

                firstName.value = "";
                lastName.value = "";
                mobileNum.value = ""
                email.value = "";
                password.value = "";
                confirmPass.value = "";

                router.push({path: '/login'})
            }
            else {
                notyf.error("Registration failed. Please contact administrator.")
            }
        }
        catch(err){
                
            // Add a check to see if the error is a 404, 401, or 400 status code.
            // If it is, use the notyf.error() method and pass the error message to notify the user of the failed login.
            // These status codes are defined in the our backend API. Any other error code will be considered an unexpected error. 
            // if(err.response.status === 404 || err.response.status === 401 || err.response.status === 400){
            //     notyf.error(err.response.data.message);
            // } else {
                notyf.error("Register Failed. Please contact administrator.");
            // }


        }      
    }
</script>


<template>
    <div class="container-fluid">
        <div class="row d-flex justify-content-center p-5">
            <div class="col-md-5 border border rounded-0 mx-auto p-5">
                <h1 class="text-dark mb-5">Register</h1> 
                <form v-on:submit="handleSubmit">
                    <div class="mb-3">
                        <input type="text" class="form-control rounded-0" id="fName" placeholder="Firstname" v-model="firstName" />
                    </div>
                    <div class="mb-3">
                        <input type="text" class="form-control rounded-0" id="lName" placeholder="Lastname" v-model="lastName" />
                    </div>
                    <div class="mb-3">
                        <input type="text" class="form-control rounded-0" id="mobile" placeholder="Mobile Number" v-model="mobileNum" />
                    </div>

                    <div class="mb-3">
                        <input type="email" class="form-control rounded-0" id="emailInput" placeholder="Email Address" v-model="email"
                        />
                    </div>
                    <div class="mb-3">
                        <input type="password" class="form-control rounded-0" id="passwordInput" placeholder="Password" v-model="password"
                        />
                    </div>
                    <div class="mb-3">
                        <input type="password" class="form-control rounded-0" id="cpasswordInput" placeholder="Confirm Password" v-model="confirmPass"
                        />
                    </div>
                    <div class="d-grid mt-3">
                        <button type="submit" class="btn btn-primary btn-block rounded-0" v-if=isEnabled>Submit</button>
                        <button type="submit" class="btn btn-primary btn-block rounded-0" disabled v-else>Submit</button>
                    </div>
                </form>
            </div>
        </div>
    </div>
</template>