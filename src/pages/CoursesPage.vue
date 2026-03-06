<script>
import api from '../api.js';
import { ref, onMounted } from 'vue';
import CourseComponent from '../components/CourseComponent.vue';

export default {
  components: { CourseComponent },

  setup() {
    const courses = ref([]);
    const loading = ref(true);
    const error = ref(null);

    onMounted(async () => {
      try {
        const { data } = await api.get('https://coursebookingapi.onrender.com/courses');

        courses.value = data.reverse();

      } catch (err) {
        console.error(err);
        error.value = "Failed to load courses.";
      } finally {
        loading.value = false;
      }
    });

    return { courses, loading, error };
  }
}
</script>

<template>
  <div class="container-fluid">
    <div class="row mt-3">
      <div class="col my-3">
        <h1 class="text-center text-dark py-1">Courses</h1>
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
        v-for="course in courses"
        :key="course._id"
        :courseData="course"
      />
    </div>
  </div>
</template>

