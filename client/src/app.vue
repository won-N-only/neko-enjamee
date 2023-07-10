<template>
  <div id="neko" :class="[chatOnly ? 'chat-only' : (!videoOnly && side_to_bottom ? 'side-to-bottom' : '')]">
    <template v-if="!$client.supported">
      <neko-unsupported />
    </template>
    <template v-else>
      <main v-if="!chatOnly" class="neko-main">
        <div v-if="!videoOnly" class="header-container">
          <neko-header />
        </div>
        <div class="video-container">
          <neko-video
            ref="video"
            :hideControls="hideControls"
            :extraControls="isEmbedMode"
            @control-attempt="controlAttempt"
          />
        </div>
        <div v-if="!videoOnly && !hideControls && bottom" class="room-container">
          <neko-members />
          <div class="room-menu">
            <div class="settings">
              <neko-menu />
            </div>
            <div v-if="!chatOnly" class="controls">
              <neko-controls :shakeKbd="shakeKbd" />
            </div>
            <div class="emotes">
              <neko-emotes />
            </div>
          </div>
        </div>
      </main>
      <neko-side v-if="chatOnly || (!videoOnly && side)" />
      <neko-connect v-if="!connected" />
      <neko-about v-if="about" />
      <notifications
        v-if="!videoOnly || !chatOnly"
        group="neko"
        position="top left"
        style="top: 50px; pointer-events: none"
        :ignoreDuplicates="true"
      />
    </template>
  </div>
</template>

<style lang="scss">
  #neko {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    max-width: 100dvw;
    max-height: 100dvh;
    flex-direction: row;
    display: flex;

    .neko-main {
      min-width: 360px;
      max-width: 100%;
      flex-grow: 1;
      flex-direction: column;
      display: flex;
      overflow: auto;

      .header-container {
        background: $background-tertiary;
        height: $menu-height;
        flex-shrink: 0;
        display: flex;
      }

      .video-container {
        background: rgba($color: #000, $alpha: 0.4);
        max-width: 100%;
        flex-grow: 1;
        display: flex;
      }

      .room-container {
        background: $background-tertiary;
        height: $controls-height;
        max-width: 100%;
        flex-shrink: 0;
        flex-direction: column;
        display: flex;
        overflow-x: auto;

        .room-menu {
          max-width: 100%;
          flex: 1;
          display: flex;

          .settings {
            margin-left: 10px;
            flex: 1;
            justify-content: flex-start;
            align-items: center;
            display: flex;
          }

          .controls {
            flex: 1;
            justify-content: center;
            align-items: center;
            display: flex;
          }

          .emotes {
            margin-right: 10px;
            flex: 1;
            justify-content: flex-end;
            align-items: center;
            display: flex;
          }

          ul {
            white-space: nowrap;
          }
        }
      }
    }
  }

  #neko.chat-only {
    flex-direction: column;

    .neko-menu {
      order: 1;
      flex: 1;
      height: 100%;
    }
  }

  #neko.side-to-bottom {
      flex-direction: column;

      .neko-main {
        order: 1;
        height: 50%;
      }

      .neko-menu {
        order: 2;
        flex: 1;
        width: 100%;
      }
  }

  @media only screen and (max-width: 800px) {
    #neko {
      flex-direction: column;

      .neko-main {
        order: 1;
        height: 50%;
      }

      .neko-menu {
        order: 2;
        flex: 1;
        width: 100%;
      }

      .room-menu {
        flex-wrap: wrap;
        overflow-y: auto;
        justify-content: center;

        .settings {
          order: 1;
        }

        .emotes {
          order: 2;
        }

        .controls {
          order: 3;
        }
      }

    }
  }

</style>

<script lang="ts">
  import { Vue, Component, Ref, Watch } from 'vue-property-decorator'

  import Connect from '~/components/connect.vue'
  import Video from '~/components/video.vue'
  import Menu from '~/components/menu.vue'
  import Side from '~/components/side.vue'
  import Controls from '~/components/controls.vue'
  import Members from '~/components/members.vue'
  import Emotes from '~/components/emotes.vue'
  import About from '~/components/about.vue'
  import Header from '~/components/header.vue'
  import Unsupported from '~/components/unsupported.vue'

  @Component({
    name: 'neko',
    components: {
      'neko-connect': Connect,
      'neko-video': Video,
      'neko-menu': Menu,
      'neko-side': Side,
      'neko-controls': Controls,
      'neko-members': Members,
      'neko-emotes': Emotes,
      'neko-about': About,
      'neko-header': Header,
      'neko-unsupported': Unsupported,
    },
  })
  export default class extends Vue {
    @Ref('video') video!: Video

    shakeKbd = false

    get volume() {
      const numberParam = parseFloat(new URL(location.href).searchParams.get('volume') || '1.0')
      return Math.max(0.0, Math.min(!isNaN(numberParam) ? numberParam * 100 : 100, 100))
    }

    get isCastMode() {
      return !!new URL(location.href).searchParams.get('cast')
    }

    get isChatMode() {
      return !!new URL(location.href).searchParams.get('chat')
    }

    mounted() {
      // https://stackoverflow.com/a/53883824
      // https://stackoverflow.com/a/74962180
      let vh = window.innerHeight * 0.01;

      if (window.visualViewport) {
        vh = window.visualViewport.height * 0.01
      }
      
      document.documentElement.style.setProperty('--vh', `${vh}px`)

      window.addEventListener('resize', () => {
        let vh = window.innerHeight * 0.01;

        if (window.visualViewport) {
          vh = window.visualViewport.height * 0.01
        }
        
        document.documentElement.style.setProperty('--vh', `${vh}px`)
      })
    }
    
    get isEmbedMode() {
      return !!new URL(location.href).searchParams.get('embed')
    }

    get hideControls() {
      return this.isCastMode
    }

    get chatOnly() {
      return this.isChatMode
    }

    get videoOnly() {
      return this.isCastMode || this.isEmbedMode
    }

    @Watch('volume', { immediate: true })
    onVolume(volume: number) {
      this.$accessor.video.setVolume(volume)
    }

    @Watch('hideControls', { immediate: true })
    onHideControls(enabled: boolean) {
      if (enabled) {
        this.$accessor.video.setMuted(false)
        this.$accessor.settings.setSound(false)
      }
    }

    controlAttempt() {
      if (this.shakeKbd || this.$accessor.remote.hosted) return

      this.shakeKbd = true
      window.setTimeout(() => (this.shakeKbd = false), 5000)
    }

    get about() {
      return this.$accessor.client.about
    }

    get side() {
      return this.$accessor.client.side
    }

    get side_to_bottom() {
      return this.$accessor.client.side_to_bottom
    }

    get bottom() {
      return this.$accessor.client.bottom
    }

    get connected() {
      return this.$accessor.connected
    }
  }
</script>
