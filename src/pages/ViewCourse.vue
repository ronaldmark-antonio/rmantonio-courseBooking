<!-- a. Update the script block and add an onBeforeMount() method to retrieve the course details with axios. -->
<script setup>
    import { onBeforeMount, reactive } from "vue";
    //import useRoute to access details from the route.
    // import useRouter to access methods for page navigation.
    import { useRoute, useRouter } from "vue-router";
    import api from "../api";

    // import global stores, which contains our global user states.
    import { useGlobalStore } from "../stores/global";

    // Import and instanciate Notyf for notifications toast
    import { Notyf } from "notyf";

    const notyf = new Notyf();

    //Get the global user state from the global store.
    const { user } = useGlobalStore();

    //useRouter() returns our configurated router as an object with different methods for page navigation.
    const router = useRouter();

    //Create a course reactive state to store the course details retreived from the Database.
    const course = reactive({ data: null });

    //Create the handleEnroll() function as async to allow the use of await.
    async function handleEnroll() {
        //api.post() is an alias to the axios() method.
        //It is a shortcut to creating an axios post request. This will no longer need us to add the method of the request.
        //api.post(url,data,options)
        // this request required authrozation header (token) but since we already setup the interceptor, we don't need to add it.
        //destructure the response object returned by api.post() and save the data property.
        let { data } = await api.post(
            `/enrollments/enroll`,
            {
                //Pass the courseId, we can access the course's id from the retrieve course details.
                enrolledCourses: [
                    {
                        courseId: course.data._id
                    },
                ],
                totalPrice: course.data.price
            }
        );

        //data.success returned is a boolean. Add an if statement which will check if successfully enrolled a student into the course.
        if(data.success === true){

            notyf.success(data.message)
            router.push({path: '/courses'});

        } else {

            notyf.error("Enrollment Failed")

        }
    }

    onBeforeMount(async () => {
        //Create an instance of the route by using the useRoute() method.
        //This will return an object that contains details about the current route.
        //Including its path and params.
        //params contain the data passed in the url.
        //the property name of params is based on the name given in the route setup in main.js
        const route = useRoute();
        //console.log(route);

        //axios.get() is an alias of the axios() method.
        //It is a shortcut to creating an axios get request. This will no longer need us to add the method of the request.
        //Instead it takes 3 arguments. The url, the data (body),options(headers)
        //The url is a string and is the route to the resource.
        //data is an object which acts as the request body.
        //options is an object that can contain the headers.
        //axios.get(url,data,options)
        let { data } = await api.get(`/courses/specific/${route.params.id}`);

        //console.log(data);

        //Update the reactive course state with the data retrieved from the API.
        course.data = data;
    });
</script>

<template>
    <div class="container">
        <nav class="my-3" aria-label="breadcrumb">
            <ol class="breadcrumb">
                <li class="breadcrumb-item"><router-link to="/courses">Courses</router-link></li>
                <!-- Display the course name in the breakcrumb -->
                <li class="breadcrumb-item active" aria-current="page">
                    {{ course.data ? course.data.name : "..." }}
                </li>
            </ol>
        </nav>
        <!-- Add conditional rendering with v-if to display the course data when it is available -->
        <div class="row mx-auto my-3 gap-4 gap-md-0" v-if="course.data">
            <div class="col-12 col-md-6">
                <!-- placehold.co will generate a placeholder image based on the text provided -->
                 <!-- Display the course name in the image -->
                <img
                    class="img-fluid rounded"
                    :src="`https://placehold.co/600x400/377399/ffffff?font=lora&text=${encodeURIComponent(
                        course.data.name
                    )}`"
                />
            </div>
            <div class="col-12 col-md-6">
                <div class="d-flex gap-2 text-primary">
                    <h1 class="bi bi-mortarboard"></h1>
                    <!-- Display the course name -->
                    <h1 class="mb-3">{{ course.data.name }}</h1>
                </div>
                <h6>Course Description:</h6>
                <p class="text-muted">
                    <!-- Display the course description -->
                    {{ course.data.description }}
                </p>
                <!-- Display the course price -->
                <p>Price: PHP {{ course.data.price }}</p>
                
                <!-- Add a click event to the Enroll button to trigger the handleEnroll function -->
                <!-- use v-if to check if the user email exists and if the user is not an admin -->
                <button class="btn btn-primary" type="button" v-if="user.email && !user.isAdmin" @click="handleEnroll">
                    Enroll
                </button>
                <!-- add another v-if to check if the user is an admin and disable the button -->
                <button class="btn btn-danger" type="button" v-if="user.email && user.isAdmin" disabled>
                    Admin are not allowed to enroll
                </button>
                <!-- add another v-if to check if the user is not logged in and redirect them to the login page  -->
                <router-link to="/login" class="btn btn-outline-danger" type="button" v-if="!user.email">
                    Login to Enroll
                </router-link>
                
            </div>
        </div>

        <!-- Show a spinner if the course data is null or undefined -->
        <!-- This is why the initial value of course.data state is null. It will show the loading state when the data is not yet retreived -->
        <div class="text-center my-5" v-if="!course.data">
            <div class="spinner-grow"></div>
        </div>
    </div>
</template>
