<!-- a. Update the script block and add an onBeforeMount() method to retrieve the course details with axios. -->
<script setup>
import { onBeforeMount, reactive, ref } from "vue";
import { useRoute, useRouter } from "vue-router";
import api from "../api";
import { useGlobalStore } from "../stores/global";
import { Notyf } from "notyf";

const notyf = new Notyf();
const { user } = useGlobalStore();
const router = useRouter();
const course = reactive({ data: null });

// ✅ Loading state for Enroll button
const isEnrolling = ref(false);

async function handleEnroll() {

    if (!user.email) {
        notyf.error("Please login first");
        router.push({ name: "Login" });
        return;
    }

    if (isEnrolling.value) return; // prevent double clicks
    isEnrolling.value = true;

    try {
        const { data } = await api.post(
            `https://coursebookingapi.onrender.com/enrollments/enroll`,
            {
                enrolledCourses: [{ courseId: course.data._id }],
                totalPrice: course.data.price
            }
        );

        if (data.success === true) {
            notyf.success(data.message);
            router.push({ path: '/courses' });
        } else {
            notyf.error("Enrollment Failed");
        }

    } catch (err) {
        notyf.error("Enrollment Failed. Please try again.");
    } finally {
        isEnrolling.value = false;
    }
}

onBeforeMount(async () => {
    const route = useRoute();
    const { data } = await api.get(`https://coursebookingapi.onrender.com/courses/specific/${route.params.id}`);
    course.data = data;
});
</script>
<template>
    <div class="container">
        <nav class="my-3" aria-label="breadcrumb">
          <ol class="breadcrumb bg-white rounded-3 p-2">

            <li 
              class="breadcrumb-item"
              v-if="!user.email"
            >
              <router-link to="/" class="text-decoration-none text-primary">
                <i class="bi bi-house"></i> Home
              </router-link>
            </li>

            <!-- Courses -->
            <li class="breadcrumb-item">
              <router-link to="/courses" class="text-decoration-none text-primary">
                <i class="bi bi-book"></i> Courses
              </router-link>
            </li>

            <!-- Current Course -->
            <li class="breadcrumb-item active text-primary fw-semibold" aria-current="page">
              <i class="bi bi-file-earmark-text"></i> 
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
                    class="img-fluid rounded-0"
                    :src="`https://placehold.co/600x400/377399/ffffff?font=lora&text=${encodeURIComponent(
                        course.data.name
                    )}`"
                />
            </div>
            <div class="col-12 col-md-6">
                <div class="d-flex gap-2 text-dark">
                    <h1 class="bi bi-mortarboard text-primary"></h1>
                    <!-- Display the course name -->
                    <h1 class="mb-3">{{ course.data.name }}</h1>
                </div>
                <h6 class="mb-1">Description:</h6>
                <p class="text-muted mb-0">
                    <!-- Display the course description -->
                    {{ course.data.description }}
                </p>
                <!-- Display the course price -->
                <p>Price: &#8369;{{ course.data.price.toLocaleString() }}</p>
                
                <!-- Add a click event to the Enroll button to trigger the handleEnroll function -->
                <!-- use v-if to check if the user email exists and if the user is not an admin -->
                <!-- Enroll button with loading state -->
                <button 
                    class="btn btn-primary" 
                    type="button" 
                    v-if="user.email && !user.isAdmin" 
                    @click="handleEnroll"
                    :disabled="isEnrolling"
                >
                    <span v-if="isEnrolling">Enrolling...</span>
                    <span v-else>Enroll</span>
                </button>
                <!-- add another v-if to check if the user is an admin and disable the button -->
                <button class="btn btn-outline-primary rounded-0" type="button" v-if="user.email && user.isAdmin" disabled>
                    Admin is not allowed to enroll
                </button>
                <!-- add another v-if to check if the user is not logged in and redirect them to the login page  -->
                <router-link to="/login" class="btn btn-outline-primary rounded-0" type="button" v-if="!user.email">
                    Login to enroll
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
