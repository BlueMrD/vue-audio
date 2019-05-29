<template>
  <div class="mt-range">
    <div class="mt-range-content" ref="content">
      <div class="mt-range-runway" style="border-top-width: 1px;"></div>
      <div class="mt-range-progress" style="height: 1px;" ref="progress" :style="{'width': thumbPosition + 'px'}"></div>
      <div class="mt-range-thumb" :style="{'left': thumbPosition + 'px'}" ref="thumb">
      </div>
    </div>
  </div>
</template>



<script>
  export default {
    name: "mt-range",
    props: {
      thumbPosition: {
        type: Number,
        default: 0
      },
    },
    computed: {},
    data() {
      return {
        content: null,
        thumb: null,
        progress: null,

      }
    },
    mounted() {
      this.content = this.$refs.content;
      this.thumb = this.$refs.thumb;
      this.progress = this.$refs.progress;
    },
    methods: {

    },

  }

</script>
