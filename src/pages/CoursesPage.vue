<script>
import api from '../api.js';
import { ref, onBeforeMount } from 'vue';
import CourseComponent from '../components/CourseComponent.vue';

export default {
  components: { CourseComponent },
  setup() {
    const courses = ref([]);

    onBeforeMount(async () => {
      let { data } = await api.get('https://coursebookingapi.onrender.com/courses/getAllCourses');
      console.log(data);
      courses.value = data;
    });

    return { courses };
  }
}
</script>

<template>
  <div class="container">
    <CourseComponent
      v-for="course in courses"
      :key="course._id"
      :courseData="course"
    />
  </div>
</template>
