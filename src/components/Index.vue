<template>
<div class="container-fluid mt-5">
  <div class="row">
    <div class="col-12 text-white">
      <h1>Radio Anime</h1>
      <p class="lead">More info: <a href="https://radioani.me">https://radioani.me</a></p>
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
