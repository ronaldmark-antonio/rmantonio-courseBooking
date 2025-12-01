<script setup>
import { onBeforeMount, reactive, ref, watch } from "vue";
import { useRoute, useRouter } from "vue-router";
import api from "../api";

import { useGlobalStore } from "../stores/global";
import { Notyf } from "notyf";

const notyf = new Notyf();
const { user } = useGlobalStore();
const router = useRouter();
const course = reactive({ data: null });

onBeforeMount(() => {
        if(!user.email || !user.isAdmin){
            router.push({path: '/courses'})
        }
    })

const name = ref("");
const description = ref("");
const price = ref("");
const isEnabled = ref(false)

watch([ name, description, price], (currentValue, oldValue) => {
    if(currentValue.every(input => input)){
        isEnabled.value = true
    } else {
        isEnabled.value = false
    }
})

async function addCourse(e) {
e.preventDefault();

const token = localStorage.getItem("token");
if (!token) return notyf.error("You must be logged in as admin");

try {
    const response = await fetch("https://coursebookingapi.onrender.com/courses", {
    method: "POST",
    headers: {
        "Content-Type": "application/json",
        "Authorization": `Bearer ${token}`
    }, body: JSON.stringify({
            name: name.value,
            description: description.value,
            price: Number(price.value)
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
        notyf.error("Course Already Exists");
    } else if (response.status === 201 || data.course === "Course added successfully" ) {
        notyf.success("Course Added");
        router.push("/courses");
    } else {
        notyf.error("Unsuccessful Course Creation");
    }
} catch (error) {
    console.error("Fetch error:", error);
    notyf.error("Server error: Could not add course");
}

}

</script>

<template>
    <div class="container-fluid">
        <div class="row d-flex justify-content-center">
            <h1 class="my-5 pt-3 text-dark text-center">Add Course</h1> 
            <div class="col-md-10 rounded-3 mx-auto p-2">
                <form @submit="addCourse">
                    <div class="form-group mt-3">
                        <input type="text" class="form-control rounded-0" id="nameInput" aria-describedby="nameHelp" placeholder="Name" v-model="name">
                    </div>
                    <div class="form-group mt-3">
                        <textarea class="form-control rounded-0" id="descriptionInput" rows="5" placeholder="Description"v-model="description"> </textarea>
                    </div>
                    <div class="form-group mt-3">
                        <input type="number" class="form-control rounded-0" id="priceInput" aria-describedby="priceHelp" placeholder="Price" min="0" v-model="price">
                    </div>
                    <div class="text-end">
                        <button type="submit" class="btn btn-primary my-3 rounded-0 mx-2" v-if="!isEnabled" disabled>Cancel</button>
                        <button type="submit" class="btn btn-primary my-3 rounded-0" v-if="!isEnabled" disabled>Submit</button>
                        <button type="submit" class="btn btn-primary mt-3 flat" v-else>Submit</button>
                    </div>
                </form>
            </div>
        </div>
    </div>
</template>