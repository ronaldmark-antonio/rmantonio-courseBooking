<script>
  export default {
    props: {
      bannerProps: Object
    },
    mounted() {
      console.log(this.bannerProps.title);
      console.log(this.bannerProps.tagline);
    }

  }
</script>

<template>
   <div class="row mb-5 py-5" id="banner">
      <div class="col-md-10 p-5 ms-5">
        <h1 class="mb-3" id="banner-title">{{bannerProps.title}}</h1>
        <p class="mb-5" id="motto">{{bannerProps.tagline}}</p>
        <router-link class="btn btn-primary btn-lg rounded-0 px-3" :to="{ name: bannerProps.destination }">{{bannerProps.buttonLabel}} <i class="bi bi-arrow-right"></i></router-link>
      </div>
    </div>
</template>

<style scoped>
#banner {
  background: linear-gradient(to right, rgb(255, 255, 255), 70%, rgba(126, 209, 247, 0.5)), 
              url('/images/banner-background.png');
  background-size: cover;
  background-position: center;
  height: 500px;
}

h1 {
  font-family: serif;
  font-style: normal;
  font-weight: 700;
  font-size: 48px;
}

@media (max-width: 768px) {

  #banner {
    text-align: center;
  }

  #banner .col-md-10 {
    margin-left: 0 !important;
    padding: 20px !important;
  }

  #banner-title,
  #motto {
    text-align: center;
  }

}
</style>