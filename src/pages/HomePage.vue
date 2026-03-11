<script>
  import BannerComponent from '../components/BannerComponent.vue';
  import HighlightsComponent from '../components/HighlightsComponent.vue';

  export default {
    components: {
      BannerComponent,
      HighlightsComponent
    },
    data() {
      return {
          bannerProps: {
            title: "Dev/Academy",
            tagline: "Opportunities for everyone, everywhere.",
            destination: "Courses",
            buttonLabel: "View our courses"
          }
      };
    }


  }
</script>

<template>
  <div class="container-fluid">
    <BannerComponent :bannerProps=bannerProps />
    <HighlightsComponent />
  </div>
</template>