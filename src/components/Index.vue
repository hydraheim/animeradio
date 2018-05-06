<template>
<div class="container-fluid mt-5">
  <div class="row">
    <div class="col-12 text-white">
      <Player></Player>
    </div>
  </div>
</div>
</template>

<script>
import Player from '@/components/Player'
export default {
  name: 'Index',
  data () {
    return {
    }
  },
  components: {
    Player
  }
}
</script>

<style scoped>
  h1,h2,h3 {
    font-size: 110%;
  }
</style>
