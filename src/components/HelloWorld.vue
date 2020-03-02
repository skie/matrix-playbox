<template>
  <div class="hello">
    <Matrix />
  </div>
</template>

<script>
import Matrix from './Matrix.vue'

export default {
  name: 'HelloWorld',
  components: {
      'Matrix': Matrix,
  },
  props: {
    msg: String
  }
}
</script>

<style scoped>
</style>
