<script setup>
import { onBeforeMount, reactive, ref, watch } from "vue";
import { useRouter } from "vue-router";
import api from "../api";

import { useGlobalStore } from "../stores/global";
import { Notyf } from "notyf";

const notyf = new Notyf();
const { user } = useGlobalStore();
const router = useRouter();
const course = reactive({ data: null });

const existingCourses = ref([]);

function normalizeCourseName(name) {
    return name
        .toLowerCase()
        .replace(/[^a-z0-9]/g, "")
        .trim();
}

onBeforeMount(async () => {
    const token = localStorage.getItem("token");

    if (!token) {
        router.push("/courses");
        return;
    }

    // load existing courses for validation
    try {
        const response = await api.get("/courses");
        existingCourses.value = response.data;
    } catch (err) {
        console.error("Failed to load courses", err);
    }
});

const name = ref("");
const description = ref("");
const price = ref("");
const formattedPrice = ref("");
const isEnabled = ref(false);

watch([name, description, price], (currentValue) => {
    if (currentValue.every(input => input)) {
        isEnabled.value = true;
    } else {
        isEnabled.value = false;
    }
});

watch(price, (newVal) => {
    if (newVal === "" || newVal === null) {
        formattedPrice.value = "";
        return;
    }

    let clean = newVal.toString().replace(/,/g, "");

    formattedPrice.value = Number(clean).toLocaleString("en-US", {
        minimumFractionDigits: 0,
        maximumFractionDigits: 2
    });
});


async function addCourse(e) {
    e.preventDefault();

    const token = localStorage.getItem("token");
    if (!token) return notyf.error("You must be logged in as admin");

    // check duplicate before sending request
    const normalizedInput = normalizeCourseName(name.value);

    const duplicate = existingCourses.value.find(course =>
        normalizeCourseName(course.name) === normalizedInput
    );

    if (duplicate) {
        return notyf.error("Course already exists");
    }

    try {
        const response = await fetch("https://coursebookingapi.onrender.com/courses", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
                "Authorization": `Bearer ${token}`
            },
            body: JSON.stringify({
                name: name.value,
                description: description.value,
                price: Number(price.value.replace(/,/g, ""))
            })
        });

        let data;
        const contentType = response.headers.get("content-type");

        if (contentType && contentType.includes("application/json")) {
            data = await response.json();
        } else {
            data = await response.text();
        }

        if (response.status === 409 || data.message === "Course already exists") {
            notyf.error("Course already exists");
        } 
        else if (response.status === 201 || data.course === "Course added successfully") {
            notyf.success("Course successfully added");
            router.push("/courses");
        } 
        else {
            notyf.error("Unsuccessful course creation");
        }

    } catch (error) {
        console.error("Fetch error:", error);
        notyf.error("Server error: Could not add course");
    }
}

function handlePriceInput(e) {
  let value = e.target.value;

  value = value.replace(/[^0-9.]/g, "");

  const parts = value.split(".");
  if (parts.length > 2) {
    value = parts[0] + "." + parts[1];
  }

  price.value = value;

  if (value) {
    formattedPrice.value = Number(value).toLocaleString("en-US", {
      minimumFractionDigits: 0,
      maximumFractionDigits: 2
    });
  } else {
    formattedPrice.value = "";
  }
}
</script>

<template>
  <div class="container-fluid px-3 px-md-0">
    <div class="row d-flex justify-content-center">

      <h1 class="my-5 pt-3 text-dark text-center">
        <i class="bi bi-plus-circle"></i> Add Course
      </h1> 

      <div class="col-md-6 rounded-3 mx-auto p-3 p-md-2">

        <form @submit="addCourse">

          <!-- Course Name -->
          <div class="form-group position-relative">
            <i class="bi bi-journal-text position-absolute top-50 start-0 translate-middle-y ms-3 text-muted"></i>
            <input
              type="text"
              class="form-control rounded-0 ps-5"
              placeholder="Name"
              v-model="name"
            >
          </div>

          <!-- Description -->
          <div class="form-group mt-3 position-relative">
            <i class="bi bi-card-text position-absolute top-0 start-0 ms-3 mt-2 text-muted"></i>
            <textarea
              class="form-control rounded-0 ps-5"
              rows="5"
              placeholder="Description"
              v-model="description"
            ></textarea>
          </div>

          <!-- Price -->
          <div class="form-group mt-3">
            <div class="input-group">
              <span class="input-group-text rounded-0"><i class="bi bi-currency-dollar"></i></span>
              <input
                type="text"
                class="form-control rounded-0"
                placeholder="Price"
                :value="formattedPrice"
                @input="handlePriceInput"
                @keypress="(e) => {
                  if (!/[0-9.]/.test(e.key)) e.preventDefault();
                }"
              />
            </div>
          </div>

          <!-- Buttons -->
        <div class="d-flex justify-content-end gap-2 mt-3">
          <button
            type="button"
            class="btn btn-outline-secondary rounded-0 d-flex align-items-center gap-1"
            @click="$router.push('/courses')"
          >
            <i class="bi bi-x-circle"></i> Cancel
          </button>

          <button
            type="submit"
            class="btn btn-primary rounded-0 d-flex align-items-center gap-1"
            :disabled="!isEnabled"
          >
            <i class="bi bi-check-circle"></i> Submit
          </button>
        </div>

        </form>

      </div>

    </div>
  </div>
</template>