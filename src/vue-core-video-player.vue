<template>
  <div class="vcp-container" ref="vcp-el">
    <video
      ref="vcp-video"
      :title="title"
      :cover="cover"
      :loop="loop"
      :src="source"></video>
    <Layers />
    <Dashboard :controls="controls" />
  </div>
</template>

<script>
import './style/common.less'

import './directives'
import { EVENTS, DEFAULT_CONFIG } from './constants'
import { parseMediaList } from './helper/media'
import { initVideoCore } from './core'
import coreMixins from './mixins'
import Dashboard from './dashboard/dashboard.vue'
import Layers from './layers/layers.vue'

export default {
  name: 'VueCoreVideoPlayer',
  mixins: [coreMixins],
  components: {
    Dashboard,
    Layers
  },
  props: {
    src: [String, Array],
    autoplay: {
      type: Boolean,
      default: true
    },
    loop: {
      type: Boolean,
      default: true
    },
    title: String,
    cover: String,
    logo: String,
    controls: {
      type: [String, Boolean],
      default: true
    },
    lang: {
      type: [String, Object],
      default: 'en'
    }
  },
  computed: {
    source: function () {
      const { src } = this
      if (!src) {
        return ''
      }
      let resolution = this.resolution || DEFAULT_CONFIG.resolution
      const medias = parseMediaList(src)
      let url
      medias.forEach((media) => {
        if (media.resolution === resolution) {
          url = media.src
        }
      })
      if (!url) {
        if (typeof medias[0] === 'object' && medias[0].src) {
          url = medias[0].src
        } else if (typeof medias[0] === 'string') {
          url = medias[0]
        }
      }
      if (this.$player) {
        this.$player.setSource(src)
      }
      return url
    }
  },
  mounted () {
    // const self = this
    this.$player = this.videoCore = initVideoCore({
      ...this.$props,
      videoEl: this.$refs['vcp-video'],
      el: this.$refs['vcp-el'],
      eventEmitter: {
        on: this.on,
        emit: this.emit
      }
    })
    this._coreID = this.videoCore.id
    this.emit(EVENTS.LIFECYCLE_INITING, this.$player)
  },
  beforeDestroy () {
    this.$player.destroy()
  }

}
</script>

<style land="less">

.vcp-container {
  position: relative;
  width: 100%;
  height: 100%;
  margin: 0 auto;
  background-color: #000;
}
.vcp-container  video{
  background-color: #000;
  width: 100%;
  height: 100%;
}
</style>
