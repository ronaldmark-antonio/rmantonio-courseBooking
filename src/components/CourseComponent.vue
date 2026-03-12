<script>
import { useRouter } from "vue-router";
import { useGlobalStore } from "../stores/global";
import { Notyf } from "notyf";
import { ref } from "vue";
import axios from "axios";

const notyf = new Notyf();

export default {
  props: {
    courseData: Object
  },
  setup(props) {
    const { user } = useGlobalStore();
    const router = useRouter();

    const isEnrolling = ref(false);
    const isActivating = ref(false);
    const isDeactivating = ref(false);

    const handleEnroll = async () => {

      if (!user.email) {
        notyf.error("Please login first");
        router.push({ name: "Login" });
        return;
      }

      if (isEnrolling.value) return;

      isEnrolling.value = true;

      try {
        await new Promise(resolve => setTimeout(resolve, 1000));
        notyf.success("Course enrolled successfully");
      } catch {
        notyf.error("Course enrollment failed");
      } finally {
        isEnrolling.value = false;
      }
    };

    const handleDeactivate = async () => {

      if (isDeactivating.value) return;

      isDeactivating.value = true;

      try {

        const res = await axios.patch(
          `https://coursebookingapi.onrender.com/courses/${props.courseData._id}/archive`,
          {},
          {
            headers: {
              Authorization: `Bearer ${localStorage.getItem("token")}`
            }
          }
        );

        if (res.status === 200) {
          props.courseData.isActive = false;
          notyf.success("Course archived successfully");
        }

      } catch {
        notyf.error("Failed to archive course");
      } finally {
        isDeactivating.value = false;
      }
    };

    const handleActivate = async () => {

      if (isActivating.value) return;

      isActivating.value = true;

      try {

        const res = await axios.patch(
          `https://coursebookingapi.onrender.com/courses/${props.courseData._id}/activate`,
          {},
          {
            headers: {
              Authorization: `Bearer ${localStorage.getItem("token")}`
            }
          }
        );

        if (res.status === 200) {
          props.courseData.isActive = true;
          notyf.success("Course activated successfully");
        }

      } catch {
        notyf.error("Failed to Could not activate course");
      } finally {
        isActivating.value = false;
      }
    };

    return {
      user,
      handleEnroll,
      handleActivate,
      handleDeactivate,
      isEnrolling,
      isActivating,
      isDeactivating
    };
  }
};
</script>

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

        <div class="d-grid gap-2 mt-md-auto">
          <template v-if="user.isAdmin">

            <router-link
              class="btn btn-primary rounded-0"
              :to="{ path: `/admin/edit-course/${courseData._id}` }"
            >
              Edit
            </router-link>

            <button
              v-if="!courseData.isActive"
              class="btn btn-outline-success rounded-0"
              :disabled="isActivating"
              @click="handleActivate"
            >
              <span v-if="isActivating">Activating...</span>
              <span v-else>Activate</span>
            </button>

            <button
              v-else
              class="btn btn-outline-danger rounded-0"
              :disabled="isDeactivating"
              @click="handleDeactivate"
            >
              <span v-if="isDeactivating">Deactivating...</span>
              <span v-else>Deactivate</span>
            </button>

          </template>

          <template v-else>

            <button 
              class="btn btn-primary rounded-0"
              :disabled="isEnrolling"
              @click="handleEnroll"
            >
              <span v-if="isEnrolling">Enrolling...</span>
              <span v-else>Enroll</span>
            </button>

            <router-link 
              class="btn btn-outline-primary rounded-0" 
              :to="{ path: `/courses/${courseData._id}`}"
            >
              View
            </router-link>

          </template>

        </div>

      </div>
    </div>
  </div>
</template>