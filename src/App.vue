<template>
  <div id="app">
    <Audio width="500" bgColor="#345345" :songs="songs" auto-play></Audio>
  </div>
</template>


<script>
  import Audio from '@/components/Audio.vue';
  import songs from '../public/static/audioData.json'
  export default {
    components: {
      Audio
    },
    data() {
      return {
        songs: songs['data']
      }
    },
    created() {
      console.log(this.songs);
      console.log(typeof this.songs);
    },
  };

</script>

<style>

</style>
