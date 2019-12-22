<template>
  <div class="waveform">
    <canvas ref="canvas"></canvas>
  </div>
</template>

<style scoped>
.waveform {
  width: 160px;
  height: 40px;
}

canvas {
  width: 100%;
  height: 100%;
}
</style>

<script>
/* eslint-disable no-console */

const DPR = 1.5

export default {
  props: {
    lineColor: {
      type: String,
      default: '#07A766'
    },
    lineWidth: {
      type: Number,
      default: 3
    },
    active: {
      type: Boolean,
      default: true
    },
  },

  data() {
    return {
      audioContext: undefined,
      audioStream: undefined,
      audioSource: undefined,
      waveform: undefined,
      context: undefined,
      rafId: undefined,
      rafCount: 0,
      isPaused: false,
      drawPaused: false,
    }
  },

  async mounted() {
    await this.getUserMedia()
    this.initVisualizer()
    window.addEventListener('webkitvisibilitychange', this.onVisibilityChange)
  },

  beforeDestroy() {
    this.destroyVisualizer()
    this.destroyAudioCtx()
    window.removeEventListener('webkitvisibilitychange', this.onVisibilityChange)
  },

  methods: {
    async getUserMedia() {
      const stream = await navigator.mediaDevices.getUserMedia({
        audio: true,
        video: false
      })

      this.getUserMediaSuccess(stream)
    },

    getUserMediaSuccess(stream) {
      this.audioContext = new AudioContext()
      this.audioStream = stream
      this.audioSource = this.audioContext.createMediaStreamSource(stream)
      this.waveform = this.audioContext.createAnalyser()
      this.waveform.fftSize = 1024
      this.audioSource.connect(this.waveform)
    },

    destroyAudioCtx() {
      if (this.audioStream) {
        this.audioStream.getTracks().forEach(track => track.stop())
        this.audioStream = undefined
      }
      if (this.audioSource) {
        this.audioSource.disconnect()
        this.audioSource = undefined
      }
      if (this.audioContext) {
        this.audioContext.close()
        this.audioContext = undefined
      }
    },

    onVisibilityChange() {
      if (document.webkitHidden) {
        this.onPause()
      } else {
        this.onStart()
      }
    },

    onPause() {
      this.isPaused = true
    },

    onStart() {
      this.isPaused = false
    },

    initVisualizer() {
      const el = this.$refs.canvas
      const { width, height } = el.getBoundingClientRect()

      this.context = el.getContext('2d')
      this.width = width * DPR
      this.height = height * DPR
      el.setAttribute('width', this.width)
      el.setAttribute('height', this.height)

      this.$nextTick(() => {
        this.tick()
      })
    },

    destroyVisualizer() {
      if (this.context) {
        this.context.clearRect(0, 0, this.width, this.height)
        this.context = undefined
      }

      if (this.rafId) {
        cancelAnimationFrame(this.rafId)
        this.rafId = undefined
        this.rafCount = 0
      }
    },

    resizeVisualizer() {
      const el = this.$refs.canvas
      const { width, height } = el.getBoundingClientRect()

      this.context = el.getContext('2d')
      this.width = width * DPR
      this.height = height * DPR
      el.setAttribute('width', this.width)
      el.setAttribute('height', this.height)
    },

    tick() {
      console.log('tick')

      this.rafId = requestAnimationFrame(this.tick)
      this.rafCount++

      if (!this.waveform || this.rafCount % 2 === 0) {
        this.rafCount = 0
        return
      }

      if (!this.active || this.isPaused) {
        if (!this.drawPaused) {
          this.drawPaused = true
          this.context.clearRect(0, 0, this.width, this.height)
          this.context.lineWidth = this.lineWidth
          this.context.strokeStyle = this.lineColor
          this.context.beginPath()
          this.context.moveTo(0, this.height / 2)
          this.context.lineTo(this.width, this.height / 2)
          this.context.stroke()
        }

        return
      }

      this.drawPaused = false

      const audioData = new Uint8Array(this.waveform.frequencyBinCount)
      const sliceWidth = this.width / audioData.length
      let x = this.width * 0.16666

      this.waveform.getByteTimeDomainData(audioData)
      this.context.clearRect(0, 0, this.width, this.height)
      this.context.lineWidth = this.lineWidth
      this.context.strokeStyle = this.lineColor
      this.context.beginPath()
      this.context.moveTo(0, this.height / 2)
      this.context.lineTo(this.width * 0.15, this.height / 2)

      for (let index = 0; index < audioData.length; index++) {
        if (index % 3 !== 0) {
          const item = audioData[index]
          const y = (item / 255) * this.height
          this.context.lineTo(x, y)
          x = x + sliceWidth
        }
      }

      this.context.lineTo(this.width * (1 - 0.15), this.height / 2)
      this.context.lineTo(this.width, this.height / 2)
      this.context.stroke()
    },
  }
}
</script>