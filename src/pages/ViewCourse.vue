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

const isEnrolling = ref(false);

async function handleEnroll() {

    if (!user.email) {
        notyf.error("Please login first");
        router.push({ name: "Login" });
        return;
    }

    if (isEnrolling.value) return; 
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

            <li class="breadcrumb-item">
              <router-link to="/courses" class="text-decoration-none text-primary">
                <i class="bi bi-book"></i> Courses
              </router-link>
            </li>

            <li class="breadcrumb-item active text-primary fw-semibold" aria-current="page">
              <i class="bi bi-file-earmark-text"></i> 
              {{ course.data ? course.data.name : "..." }}
            </li>

          </ol>
        </nav>

        <div class="row mx-auto my-3 gap-4 gap-md-0" v-if="course.data">
            <div class="col-12 col-md-6">
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
                    <h1 class="mb-3">{{ course.data.name }}</h1>
                </div>
                <h6 class="mb-1">Description:</h6>
                <p class="text-muted mb-0">
                    <!-- Display the course description -->
                    {{ course.data.description }}
                </p>
                <p>Price: &#8369;{{ course.data.price.toLocaleString() }}</p>
                
                <button 
                  class="btn btn-primary d-flex align-items-center gap-2"
                  type="button" 
                  v-if="user.email && !user.isAdmin" 
                  @click="handleEnroll"
                  :disabled="isEnrolling"
                >
                  <span v-if="isEnrolling">
                    <i class="bi bi-arrow-repeat spin"></i> Enrolling...
                  </span>
                  <span v-else>
                    <i class="bi bi-journal-plus"></i> Enroll
                  </span>
                </button>
                <button class="btn btn-outline-primary rounded-0" type="button" v-if="user.email && user.isAdmin" disabled>
                    Admin is not allowed to enroll
                </button>
                <router-link to="/login" class="btn btn-outline-primary rounded-0" type="button" v-if="!user.email">
                    <i class="bi bi-box-arrow-in-right"></i>
                    Login to enroll
                </router-link>
                
            </div>
        </div>

        <div class="text-center my-5" v-if="!course.data">
            <div class="spinner-grow"></div>
        </div>
    </div>
</template>
