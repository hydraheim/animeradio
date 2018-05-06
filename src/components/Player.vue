<template>
<div class="row no-gutters">
  <div class="row header p-2 align-items-center fixed-top bg-dark text-white">
    <div class="col-12 col-sm-8">
      <h1>Anime Radio</h1>
    </div>
    <div class="col-12 col-sm-4">
      <b-form-select v-if="radioStations"
        v-model="selectedRadioStation"
        :options="radioStations"
        size="sm"
        stacked
        value-field="shortName"
      >
      <template slot="first">
        <!-- this slot appears above the options from 'options' prop -->
        <option :value="null" disabled>Please select a station</option>
      </template>
      </b-form-select>
      <div v-else>
        <font-awesome-icon icon="spinner" spin /> Loading station data...
      </div>
    </div>
  </div>

  <div class="player station-info text-dark fixed-bottom w-100">
    <div class="font-weight-bold mb-1" v-if="currentlyPlaying">
      <div class="p-2 bg-dark text-white">
        <h4 class="mb-0">{{ radioStationName }}</h4>
        <p class="font-weight-bold mb-0"><em>{{ radioStationMeta }}</em></p>
        <em>{{ radioStationWebPage }}</em>
      </div>
      <div class="row align-items-center p-2">
        <div class="col-12 col-md-10">
          <h4>
            <Soundbars />
            Playing now: {{ currentSongName }}
          </h4>
        </div>
        <div class="col-12 col-md-2 text-right">
          <img
            :src="currentSongArt"
            v-if="currentSongArt"
            class="img-fluid"
            width="100px"
            alt="Song art"
          />
        </div>
      </div>
    </div>
    <div class="p-2" v-else>
      <font-awesome-icon icon="music" />
      Waiting for selection...
    </div>
    <div>
      <audio
        id="audioPlayer"
        ref="audio"
        class="w-100"
        controls="controls"
        autoplay="true"
        @change="playing"
        preload="auto"
        @playing="playing"
        volume="0.05"
        :src="audioSource"
      ></audio>
    </div>
  </div>
</div>
</template>

<script>
import Soundbars from '@/components/Soundbars'
import axios from 'axios'
export default {
  data () {
    return {
      currentTime: '00:00',
      audioSource: null,
      currentSongName: '',
      radioStationName: '',
      radioStationMeta: '',
      radioStationWebPage: '',
      currentSongArt: '',
      bitRate: 0,
      res: false,
      refreshID: '',
      selectedRadioStation: null,
      currentlyPlaying: false
    }
  },
  components: {
    Soundbars
  },
  mounted () {
    this.$refs.audio.volume = 0.07
    window.clearInterval(this.refreshID)
  },
  // Options / Data
  watch: {
    selectedRadioStation () {
      console.log('Changed Radio Station')
      this.resetEverything()
      this.audioSource = this.radioStations[this.selectedRadioStation].streamUrl
      this.radioStationName = this.radioStations[this.selectedRadioStation].text
      this.updateSongMeta(this.selectedRadioStation)
      // Register a regular interval that we ask for updates on what's playing
      window.clearInterval(this.refreshID)
      if (this.currentlyPlaying === true) {
        this.refreshID = window.setInterval(() => {
          this.updateSongMeta(this.selectedRadioStation)
        }, 25000)
      }
    }
  },
  asyncComputed: {
    radioStations () {
      return axios.get(`https://intense-dusk-45481.herokuapp.com/stations`)
        .then(res => {
          return res.data
        })
    }
  },
  methods: {
    resetEverything () {
      window.clearInterval(this.refreshID)
      this.refreshID = null
      this.currentlyPlaying = false
      this.currentSongArt = ''
      this.audioSource = null
      this.radioStationName = ''
    },
    timeListener () {
      this.currentTime = this.$refs.audio.currentTime
    },
    playing () {
      this.currentlyPlaying = true
    },
    retrieveSongImage (term) {
      let url = `http://ws.audioscrobbler.com/2.0/?method=track.search&track=${term}&api_key=444f161c60e2f8caf55a2403c20720e4&format=json`
      axios.get(url)
        .then(res => {
          if (res.data.results.trackmatches.track.length) {
            this.currentSongArt = res.data.results.trackmatches.track[0].image[2]['#text']
          }
        })
    },
    updateSongMeta (url) {
      axios.get(`https://intense-dusk-45481.herokuapp.com/?station=${this.selectedRadioStation}`)
        .then(res => {
          this.currentSongName = res.data.currentlyPlaying
          this.bitRate = res.data.bitrate
          this.radioStationMeta = res.data.stationDescription
          this.retrieveSongImage(this.currentSongName)
        })
    }
  }
}
</script>

<style scoped>
audio::-webkit-media-controls-panel {
  background: #f2f2f2;
  color: #fff;
}
audio::-webkit-media-controls-play-button {
  color: #fff;
}
audio::-webkit-media-controls-volume-slider-container,
audio::-webkit-media-controls-volume-slider,
audio::-webkit-media-controls-timeline-container,
audio::-webkit-media-controls-current-time-display,
audio::-webkit-media-controls-timeline {
  color: #222;
  font-size: 120%;
  font-weight: 700;
}
  h1,h2,h3 {
    font-size: 110%;
  }
</style>
