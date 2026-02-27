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
            :disabled="isEnrolling"
            @click="handleEnroll"
          >
            <span v-if="isEnrolling">Enrolling...</span>
            <span v-else>Enroll</span>
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
import { ref } from "vue";

const notyf = new Notyf();

export default {
  props: {
    courseData: Object
  },
  setup(props) {
    const { user } = useGlobalStore();
    const router = useRouter();
    const isEnrolling = ref(false);

    const handleEnroll = async () => {

      if (!user.email) {
        notyf.error("Please login first");
        router.push({ name: "Login" });
        return;
      }

      if (isEnrolling.value) return;

      isEnrolling.value = true;

      try {
        // simulate API call (replace with real axios call later)
        await new Promise(resolve => setTimeout(resolve, 1000));

        notyf.success("Successfully enrolled");
      } catch (err) {
        notyf.error("Enrollment failed");
      } finally {
        isEnrolling.value = false;
      }
    };

    return {
      user,
      handleEnroll,
      isEnrolling
    };
  }
};
</script>