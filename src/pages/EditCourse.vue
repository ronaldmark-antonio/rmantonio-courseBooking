<script setup>
import { ref, onMounted } from "vue";
import { useRoute, useRouter } from "vue-router";
import axios from "axios";
import { Notyf } from "notyf";

const route = useRoute();
const router = useRouter();
const notyf = new Notyf();

const name = ref("");
const description = ref("");
const price = ref("");

const isSubmitting = ref(false);

onMounted(async () => {
  try {
    const res = await axios.get(
      `https://coursebookingapi.onrender.com/courses/${route.params.id}`
    );

    name.value = res.data.name;
    description.value = res.data.description;
    price.value = res.data.price;

  } catch (err) {
    notyf.error("Failed to load course");
  }
});

const handleUpdate = async (e) => {
  e.preventDefault();

  if (isSubmitting.value) return;
  isSubmitting.value = true;

  try {
    const res = await axios.patch(
      `https://coursebookingapi.onrender.com/courses/${route.params.id}`,
      {
        name: name.value,
        description: description.value,
        price: price.value
      },
      {
        headers: {
          Authorization: `Bearer ${localStorage.getItem("token")}`
        }
      }
    );

    if (res.status === 200) {
      notyf.success("Course updated successfully");
      router.push("/courses");
    }

  } catch (err) {
    notyf.error("Failed to update course");
  } finally {
    isSubmitting.value = false;
  }
};
</script>

<template>
  <div class="container mt-5" style="max-width: 500px;">
    <h2 class="mb-4 d-flex align-items-center gap-2">
      <i class="bi bi-pencil-square"></i> Edit Course
    </h2>

    <form @submit="handleUpdate">

      <!-- Name -->
      <div class="mb-3 position-relative">
        <i class="bi bi-book position-absolute top-50 start-0 translate-middle-y ms-3"></i>
        <input
          type="text"
          class="form-control ps-5"
          v-model="name"
          placeholder="Course Name"
          required
        />
      </div>

      <!-- Description -->
      <div class="mb-3 position-relative">
        <i class="bi bi-card-text position-absolute top-0 start-0 mt-3 ms-3"></i>
        <textarea
          class="form-control ps-5"
          rows="3"
          v-model="description"
          placeholder="Description"
          required
        ></textarea>
      </div>

      <div class="mb-3 position-relative">
        <i class="bi bi-cash position-absolute top-50 start-0 translate-middle-y ms-3"></i>
        <input
          type="number"
          class="form-control ps-5"
          v-model="price"
          placeholder="Price"
          required
        />
      </div>

      <button
        class="btn btn-primary w-100 d-flex align-items-center justify-content-center gap-2"
        :disabled="isSubmitting"
      >
        <span v-if="isSubmitting">
          <i class="bi bi-arrow-repeat spin"></i> Updating...
        </span>
        <span v-else>
          <i class="bi bi-check-circle"></i> Update Course
        </span>
      </button>

    </form>
  </div>
</template>

<style scoped>
.spin {
  animation: spin 1s linear infinite;
}
@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}
</style>