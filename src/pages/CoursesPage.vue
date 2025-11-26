<script>
import api from '../api.js';
import { ref, onBeforeMount } from 'vue';
import CourseComponent from '../components/CourseComponent.vue';

export default {
  components: { CourseComponent },
  setup() {
    const courses = ref([]);

    onBeforeMount(async () => {
      const { data } = await api.get('https://coursebookingapi.onrender.com/courses');
      console.log(data);
      courses.value = data;
    });


    return { courses };
  }
}
</script>

<template>
  <div class="container-fluid">
    <div class="col my-5">
      <h1 class="text-center text-primary">Courses</h1>
    </div>
    <div class="row p-5 m-2">
      <CourseComponent
        v-for="course in courses"
        :key="course._id"
        :courseData="course"
      />
    </div>
  </div>
</template>
