<template>
  <div class="col-12 col-md-6 col-lg-3 p-2 d-flex flex-row">
    <div id="CourseCard" class="card cardHighlights shadow-sm rounded-0" style="min-height: 100%">
      
      <img 
        class="card-img-top rounded-0"
        :src="`https://placehold.co/600x400/377399/ffffff?font=lora&text=${encodeURIComponent(courseData.name)}`"
        :alt="courseData.name"
      >

      <div class="card-body d-flex flex-column">
        <h4 class="card-title fw-bold mb-0">
          {{ courseData.name }}
        </h4>

        <p class="card-text text-muted mb-0">
          {{ courseData.description.slice(0, 100) + (courseData.description.length > 100 ? '...' : '') }}
        </p>

        <p class="mb-3">
          <span>Price:</span> &#8369;{{ courseData.price.toLocaleString() }}
        </p>

        <!-- Buttons -->
        <div class="d-grid gap-2 mt-md-auto">

          <!-- Enroll Button -->
          <button 
            v-if="!user.isAdmin"
            class="btn btn-primary rounded-0"
            @click="handleEnroll"
          >
            Enroll
          </button>

          <!-- View Course -->
          <router-link 
            class="btn btn-outline-primary rounded-0" 
            :to="{ path: `/courses/${courseData._id}`}"
          >
            View Course
          </router-link>

        </div>

      </div>
    </div>
  </div>
</template>

<script>
import { useRouter } from "vue-router";
import { useGlobalStore } from "../stores/global";
import { Notyf } from "notyf";

const notyf = new Notyf();

export default {
  props: {
    courseData: Object
  },
  setup() {
    const { user } = useGlobalStore();
    const router = useRouter();

    const handleEnroll = () => {

      if (!user.email) {
        notyf.error("Please login first.");
        router.push({ name: "Login" });
        return;
      }

      notyf.success("Successfully enrolled!");
    };

    return {
      user,
      handleEnroll
    };
  }
};
</script>