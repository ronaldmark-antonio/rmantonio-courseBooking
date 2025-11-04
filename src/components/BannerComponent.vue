<!-- When creating a new VueJS component, it must consist of 3 blocks, template, script and style. -->
<!-- When naming component, it must consist of two words and preferably in PascalCase -->

<template>
   <div class="row mb-5 py-5" id="banner">
      <div class="col-md-10 p-5 ms-5">
        <h1 class="mb-3" id="banner-title">{{bannerProps.title}}</h1>
        <p class="mb-5" id="motto">{{bannerProps.tagline}}</p>
        <!-- ACTIVITY SOLUTION START -->
        <router-link class="btn btn-primary btn-lg rounded-pill px-3" :to="{ name: bannerProps.destination }">{{bannerProps.buttonLabel}} <i class="bi bi-arrow-right"></i></router-link>
        <!-- ACTIVITY SOLUTION END -->
      </div>
    </div>
</template>


<script>

  // Options API was first used in VueJS Development and is mostly used for smaller scaled applications.
  // Defining the props property using the Options API will allow components to be completely resuable.
  export default {
    // props property is from the Options API that allow us to define the props that can be passed to a component.
    props: {
      // We will replace the hardcoded text in our template to details/data passed from the parent.
      // title: String,
      // tagline: String
      bannerProps: Object
    },
    // mounted() method can be added which is a function/method that will run when the component is mounted.
    mounted() {
      console.log(this.bannerProps.title);
      console.log(this.bannerProps.tagline);
    }

  }

</script>




<!-- By adding scoped attribute, the styles defined here will only be applied to this component.  -->
<style scoped>
  /* modify the top div to have a background image */
  #banner {
    background: linear-gradient(to right, rgb(255, 255, 255), 70%, rgba(126, 209, 247, 0.5)), url('/images/banner-background.png');
    background-size: cover;
    background-position: center;
    height: 500px;
  }

  /* Modify the h1 to have a Montserrat font */
  /* Since this is scoped, it will only be applied to the h1 element of this component */
  h1 {
      font-family: serif;
      font-style: normal;
      font-weight: 700;
      font-size: 48px;
  }

</style>