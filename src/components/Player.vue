<template>
<div class="row no-gutters">
  <div class="col-12">
    <b-form-select
    v-model="audioSource"
    :options="radioStations"
    stacked
    value-field="streamUrl"
    ></b-form-select>
  </div>
  <h4 v-if="audioSource === null">Select a radio station from the list to get started...</h4>
  <div v-else-if="!currentSongName || !radioStationName">
    <span class="fa fa-spin fa-2x fa-spinner"></span> Loading station data...
  </div>
  <div class="player station-info text-dark fixed-bottom w-100">
    <div class="p-3 bg-dark text-white text-center">
      <h4 class="mb-0">{{ radioStationName }}</h4>
      <p class="font-weight-bold mb-0">{{ radioStationMeta }}</p>
      <em>{{ radioStationWebPage }}</em>
    </div>
    <hr />
    <p class="p-2 lead font-weight-bold mb-1">Playing now: {{ currentSongName }}</p>
    <b-badge variant="info">Bitrate: {{ bitRate }} kbps</b-badge>
    <div v-show="!currentlyPlaying">
      <span class="fa fa-spin fa-2x fa-spinner"></span>
      Waiting for stream to start playing...
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
import icy from 'icy'
// import IcyParser from 'icecast-parser'
import axios from 'axios'
import RadioStations from '@/assets/stations.json'
export default {
  data () {
    return {
      currentTime: '00:00',
      audioSource: null,
      currentSongName: '',
      radioStationName: '',
      radioStationMeta: '',
      radioStationWebPage: '',
      radioStations: RadioStations,
      bitRate: 0,
      res: false,
      refreshID: '',
      selectedRadioStation: '',
      currentlyPlaying: false
    }
  },
  // Options / Data
  watch: {
    audioSource () {
      console.log('Audio source changed')
      console.dir(this.audioSource)
      this.currentlyPlaying = false
      this.updateSongMeta(this.audioSource)
    }
  },
  methods: {
    timeListener () {
      this.currentTime = this.$refs.audio.currentTime
    },
    playing () {
      this.$refs.audio.volume = 0.10
      this.currentlyPlaying = true
      window.clearInterval(this.refreshID)
      if (this.currentlyPlaying === true) {
        this.refreshID = window.setInterval(() => {
          this.updateSongMeta(this.audioSource)
        }, 45000)
      }
      console.log('now we are playing')
    },
    updateSongMeta (url) {
      axios.get(`https://intense-dusk-45481.herokuapp.com/?station=${url}`)
        .then(res => {
          this.currentSongName = res.data.currentlyPlaying
          this.bitRate = res.data.bitrate
          this.radioStationMeta = res.data.stationDescription
          console.log(res)
        })
    },
    retrieveStation (radioUrl) {
      /* let newURL = 'http://174.37.159.206:8052/stats?sid=1'
      var request = require('request')
      request(newURL, (error, response, body) => {
        console.log('REQUESTING OSV')
        console.log('error:', error) // Print the error if one occurred
        console.log('statusCode:', response && response.statusCode) // Print the response status code if a response was received
        console.log('body:', body) // Print the HTML for the Google homepage.
      })

      const Parser = require('icecast-parser')

      const radioStation = new Parser({
        url: newURL, // URL to radio station
        keepListen: true, // don't listen radio station after metadata was received
        autoUpdate: true, // update metadata after interval
        errorInterval: 10 * 60, // retry connection after 10 minutes
        emptyInterval: 5 * 60, // retry get metadata after 5 minutes
        metadataInterval: 5 // update metadata after 5 seconds
      })

      radioStation.on('metadata', function (metadata) {
        console.log('Here you will receive metadata each 10 seconds')
        console.log(metadata.StreamTitle)
      })

      radioStation.on('stream', function (stream) {
        stream.pipe(process.stdout) // Stream it's readable stream and you can pipe it to any writable stream
      })

      radioStation.on('error', function (error) {
        console.log('ERROR OSV')
        console.log(error)
        console.log(['Connection to', this.getConfig('url'), 'was rejected'].join(' '))
      })

      radioStation.on('empty', function () {
        console.log(['Radio station', this.getConfig('url'), 'doesn\'t have metadata'].join(' '))
      })
      // let url = 'http://finalfantasystation.com:8000/stream.mp3'

      fetch(newURL, {
        headers: {
          'Access-Control-Allow-Origin': '*',
          'User-Agent': 'Mozilla/5.0',
          'Icy-MetaData': '1'
        }
      })
        .then(res => {
          console.log(res)
        })
      /* var r = new XMLHttpRequest();
      r.open("GET", "http://finalfantasystation.com:8000/stream.mp3", true);
      r.onreadystatechange = function () {
        console.log(r.response)
      };
      r.send("banana=yellow");
      var radioStation = new IcyParser(radioUrl)

      radioStation.on('error', function (error) {
        console.log(error)
        console.log(['Connection to', this.getConfig('url'), 'was rejected'].join(' '))
      })

      radioStation.on('empty', function () {
        console.log(['Radio station', this.getConfig('url'), 'doesn\'t have metadata'].join(' '))
      })
      radioStation.on('metadata', metadata => {
        console.log(metadata)
      })
      /* var instance = axios.create({
        baseURL: '',
        timeout: 10000,
        headers: {'Icy-MetaData': 1, 'Content-Type': 'audio/mpeg'}
      })
      instance.get(radioUrl)
        .then(res => {
          console.log('AXIOS RES')
          console.log(res)
        })
        .catch(e => {
          console.log(e)
        }) */
      /* icy.get(radioUrl, res => {
        this.res = res
        // log the HTTP response headers
        console.log(res.headers)
        let header = res.headers
        this.bitRate = header['icy-br']
        this.radioStationWebPage = header['icy-url']
        this.radioStationName = header['icy-name']
        this.radioStationMeta = header['icy-description']

        // log any "metadata" events that happen
        res.on('metadata', metadata => {
          console.log('Retrieving meta data')
          var parsed = icy.parse(metadata)
          this.currentSongName = parsed.StreamTitle

          console.log('audio ref')
          console.log(parsed)
        })
      }) */
    },
    retrieveSongData () {
      icy.get(this.audioSource, res => {
        res.on('metadata', metadata => {
          console.log('Retrieving meta data')
          var parsed = icy.parse(metadata)
          this.currentSongName = parsed.StreamTitle

          console.log('audio ref update')
          console.log(parsed)
        })
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
</style>
