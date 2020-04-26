<template>
  <div class="letter shadow mr-3">
    <span v-if="opened">{{value}}</span>
    <span v-else></span>
  </div>
</template>
<script>
export default {
  props: {
    value: {
      type: String,
      default: ""
    },
    opened: {
      type: Boolean,
      default: false
    }
  }
};
</script>
<style scoped>
.letter {
  width: 100px;
  height: 100px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 25pt;
}
</style>