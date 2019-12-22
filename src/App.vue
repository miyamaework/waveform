<template>
  <div>
    <WaveForm v-if="waveform" :line-width="3" :active="waveformActive" />

    <div>
      waveform active: {{ waveformActive }}
      <button @click="onWaveformPause">Pause</button>
      <button @click="onWaveformStart">Start</button>
    </div>

    <div>
      waveform instance: {{ waveform }}
      <button @click="onWaveformInit">waveform init</button>
      <button @click="onWaveformDestroy">waveform destory</button>
    </div>
  </div>
</template>

<script>
/* eslint-disable no-console */

import WaveForm from './WaveForm.vue'

export default {
  components: {
    WaveForm,
  },

  data() {
    return {
      waveform: false,
      waveformActive: true
    }
  },

  methods: {
    onWaveformPause() {
      this.waveformActive = false
    },

    onWaveformStart() {
      this.waveformActive = true
    },

    onWaveformInit() {
      this.waveform = true
    },

    onWaveformDestroy() {
      this.waveform = false
    },
  }
}
</script>