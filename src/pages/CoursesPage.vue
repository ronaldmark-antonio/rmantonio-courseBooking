<script>
import api from '../api.js';
import { ref, watch, computed } from 'vue';
import CourseComponent from '../components/CourseComponent.vue';
import { useGlobalStore } from "../stores/global";

export default {
  components: { CourseComponent },

  setup() {

    const { user } = useGlobalStore();

    const courses = ref([]);
    const loading = ref(true);
    const error = ref(null);
    const search = ref("");

    const loadCourses = async () => {

      try {

        let response;

        if (user.isAdmin) {

          response = await api.get(
            "/courses/all",
            {
              headers: {
                Authorization: `Bearer ${localStorage.getItem("token")}`
              }
            }
          );

        } else {

          response = await api.get("/courses");

        }

        courses.value = response.data.reverse();

      } catch (err) {

        console.error(err);
        error.value = "Failed to load courses.";

      } finally {

        loading.value = false;

      }

    };

    watch(
      () => user.isAdmin,
      () => {
        loadCourses();
      },
      { immediate: true }
    );

    const filteredCourses = computed(() => {

      if (!search.value) return courses.value;

      return courses.value.filter(course =>
        course.name.toLowerCase().includes(search.value.toLowerCase())
      );

    });

    return { courses, loading, error, user, search, filteredCourses };

  }
}
</script>

<template>
  <div class="container-fluid">

    <div class="row mt-3">
      <div class="col my-3">
        <h1 class="text-center text-dark py-1">
          {{ user.isAdmin ? "Courses Dashboard" : "Courses" }}
        </h1>
      </div>
    </div>

    <!-- Search for everyone -->
    <div class="row justify-content-center mb-3">
      <div class="col-md-4">
        <input
          type="text"
          class="form-control rounded-0"
          placeholder="Search courses..."
          v-model="search"
        >
      </div>
    </div>

    <div v-if="loading" class="text-center py-5">
      <h4>Loading courses...</h4>
    </div>

    <div v-else-if="error" class="text-center text-danger py-5">
      <h4>{{ error }}</h4>
    </div>

    <div v-else class="row p-3 m-3">
      <CourseComponent
        v-for="course in filteredCourses"
        :key="course._id"
        :courseData="course"
      />
    </div>

  </div>
</template>

