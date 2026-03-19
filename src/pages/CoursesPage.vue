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
    const priceRange = ref("all");

    const loadCourses = async () => {
      try {
        let response;

        if (user.isAdmin) {
          response = await api.get("/courses/all", {
            headers: {
              Authorization: `Bearer ${localStorage.getItem("token")}`
            }
          });
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
      return courses.value.filter(course => {

        const matchesSearch =
          !search.value ||
          course.name.toLowerCase().startsWith(search.value.toLowerCase());

        let matchesPrice = true;

        if (priceRange.value === "0-500") {
          matchesPrice = course.price >= 0 && course.price <= 500;
        } else if (priceRange.value === "500-1000") {
          matchesPrice = course.price > 500 && course.price <= 1000;
        } else if (priceRange.value === "1000-2000") {
          matchesPrice = course.price > 1000 && course.price <= 2000;
        } else if (priceRange.value === "2000+") {
          matchesPrice = course.price > 2000;
        }

        return matchesSearch && matchesPrice;
      });
    });

    const clearFilters = () => {
      search.value = "";
      priceRange.value = "all";
    };

    return { 
      courses,
      loading,
      error,
      user,
      search,
      priceRange,
      filteredCourses,
      clearFilters
    };
  }
}
</script>

<template>
  <div class="container-fluid">

    <!-- TITLE -->
    <div class="row mt-3">
      <div class="col my-3">
        <h1 class="text-center text-dark py-1">
          {{ user.isAdmin ? "Courses Dashboard" : "Courses" }}
        </h1>
      </div>
    </div>

    <!-- ADMIN FILTERS -->
    <div v-if="user.isAdmin" class="d-flex justify-content-center">
      <div style="width:48%;" class="row g-2 align-items-stretch">

        <!-- LEFT: Search + Price -->
        <div class="col-md-8 d-flex flex-column gap-1">
          <input
            type="text"
            class="form-control form-control-sm rounded-0"
            placeholder="Search courses"
            v-model="search"
          >

          <select
            class="form-select form-select-sm rounded-0"
            v-model="priceRange"
          >
            <option value="all">All Prices</option>
            <option value="0-500">₱0 – ₱500</option>
            <option value="500-1000">₱500 – ₱1000</option>
            <option value="1000-2000">₱1000 – ₱2000</option>
            <option value="2000+">₱2000+</option>
          </select>
        </div>

        <!-- RIGHT: Clear -->
        <div class="col-md-4 d-flex">
          <button
            class="btn btn-primary btn-sm rounded-0 w-100 h-100"
            @click="clearFilters"
          >
            Clear
          </button>
        </div>

      </div>
    </div>

    <!-- USER / GUEST FILTERS -->
    <div v-if="!user.isAdmin" class="row mb-3 g-2 px-3 m-3 justify-content-center align-items-stretch">

      <div class="col-md-6 d-flex flex-column gap-1">
        <input
          type="text"
          class="form-control form-control-sm rounded-0"
          placeholder="Search courses"
          v-model="search"
        >

        <select
          class="form-select form-select-sm rounded-0"
          v-model="priceRange"
        >
          <option value="all">All Prices</option>
          <option value="0-500">₱0 – ₱500</option>
          <option value="500-1000">₱500 – ₱1000</option>
          <option value="1000-2000">₱1000 – ₱2000</option>
          <option value="2000+">₱2000+</option>
        </select>
      </div>

      <div class="col-md-2 d-flex">
        <button
          class="btn btn-primary btn-sm rounded-0 w-100 h-100"
          @click="clearFilters"
        >
          Clear
        </button>
      </div>

    </div>

    <!-- STATES -->
    <div v-if="loading" class="text-center py-5">
      <h4>Loading courses...</h4>
    </div>

    <div v-else-if="error" class="text-center text-danger py-5">
      <h4>{{ error }}</h4>
    </div>

    <div v-else-if="filteredCourses.length === 0" class="text-center py-5">
      <h4 class="text-muted">No courses found.</h4>
    </div>

    <!-- ADMIN TABLE -->
    <div v-else-if="user.isAdmin" class="p-3 m-3 d-flex justify-content-center">
      <div style="width: 50%;">
        <table class="table table-hover align-middle">
          <thead class="table-dark">
            <tr>
              <th>Course</th>
              <th>Description</th>
              <th>Price</th>
              <th>Status</th>
              <th>Actions</th>
            </tr>
          </thead>

          <tbody>
            <CourseComponent
              v-for="course in filteredCourses"
              :key="course._id"
              :courseData="course"
            />
          </tbody>
        </table>
      </div>
    </div>

    <!-- USER GRID -->
    <div v-else class="row p-3 m-3">
      <CourseComponent
        v-for="course in filteredCourses"
        :key="course._id"
        :courseData="course"
      />
    </div>

  </div>
</template>