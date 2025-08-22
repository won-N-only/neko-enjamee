<template>
  <div class="settings">
    <ul>
      <li>
        <span>{{ $t('setting.scroll') }}</span>
        <label class="slider">
          <input type="range" min="1" max="100" v-model="scroll" />
        </label>
      </li>
      <li>
        <span>{{ $t('setting.scroll_invert') }}</span>
        <label class="switch">
          <input type="checkbox" v-model="scroll_invert" />
          <span />
        </label>
      </li>
      <li>
        <span>{{ $t('setting.autoplay') }}</span>
        <label class="switch">
          <input type="checkbox" v-model="autoplay" />
          <span />
        </label>
      </li>
      <li>
        <span>{{ $t('setting.ignore_emotes') }}</span>
        <label class="switch">
          <input type="checkbox" v-model="ignore_emotes" />
          <span />
        </label>
      </li>
      <li>
        <span>{{ $t('setting.chat_sound') }}</span>
        <label class="switch">
          <input type="checkbox" v-model="chat_sound" />
          <span />
        </label>
      </li>
      <li>
        <span>{{ $t('setting.keyboard_layout') }}</span>
        <label class="select">
          <select v-model="keyboard_layout">
            <option v-for="(name, code) in keyboard_layouts_list" :key="code" :value="code">{{ name }}</option>
          </select>
          <span />
        </label>
      </li>
      <li class="broadcast" v-if="admin">
        <div>
          <span>{{ $t('setting.broadcast_title') }}</span>
          <button v-if="!broadcast_is_active" @click.stop.prevent="$accessor.settings.broadcastCreate(broadcast_url)">
            <i class="fas fa-play"></i>
          </button>
          <button v-else @click.stop.prevent="$accessor.settings.broadcastDestroy()" class="btn-red">
            <i class="fas fa-stop"></i>
          </button>
        </div>
        <input
          v-model="broadcast_url"
          :disabled="broadcast_is_active"
          class="input"
          placeholder="rtmp://a.rtmp.youtube.com/live2/<stream-key>"
        />
      </li>
      <li v-if="connected">
        <button @click.stop.prevent="logout">{{ $t('logout') }}</button>
      </li>
    </ul>
  </div>
</template>

<style lang="scss" scoped>
  .settings {
    display: flex;
    flex: 1;
    padding: 1rem;

    ul {
      display: flex;
      flex-direction: column;
      width: 100%;
      gap: 0.5rem;

      li {
        display: flex;
        align-items: center;
        padding: 0.5rem 0;
        border-bottom: 0.0625rem solid $background-secondary;

        &:last-child {
          border-bottom: none;
        }

        &.broadcast {
          flex-direction: column;
          gap: 0.5rem;

          div {
            display: flex;
            align-items: center;
            width: 100%;
          }
        }

        span {
          flex: 1;
          font-size: 0.875rem;
          white-space: nowrap;
        }

        button {
          display: flex;
          align-items: center;
          justify-content: center;
          min-width: 2rem;
          height: 2rem;
          padding: 0 1rem;
          border: none;
          border-radius: 0.25rem;
          background: $style-primary;
          color: $text-normal;
          font-weight: 600;
          text-transform: uppercase;
          cursor: pointer;
          transition: background-color 0.2s ease;

          &:hover {
            background-color: lighten($style-primary, 5%);
          }

          &.btn-red {
            background-color: $style-error;

            &:hover {
              background-color: lighten($style-error, 5%);
            }
          }

          i {
            font-size: 0.875rem;
          }
        }

        .switch {
          position: relative;
          width: 2.625rem;
          height: 1.5rem;
          flex-shrink: 0;

          input {
            width: 0;
            height: 0;
            opacity: 0;
          }

          span {
            position: absolute;
            inset: 0;
            background-color: $background-tertiary;
            border-radius: 1.5rem;
            cursor: pointer;
            transition: background-color 0.2s ease;

            &:before {
              content: '';
              position: absolute;
              left: 0.1875rem;
              bottom: 0.1875rem;
              width: 1.125rem;
              height: 1.125rem;
              background-color: white;
              border-radius: 50%;
              transition: transform 0.2s ease;
              box-shadow: 0 0.125rem 0.25rem rgba(0, 0, 0, 0.3);
            }
          }
        }

        input[type='checkbox'] {
          &:checked + span {
            background-color: $style-primary;
          }

          &:checked + span:before {
            transform: translateX(1.125rem);
          }
        }

        .slider {
          display: flex;
          align-items: center;
          width: 7.5rem;
          flex-shrink: 0;

          input[type='range'] {
            width: 100%;
            height: 1.5rem;
            background: transparent;
            appearance: none;

            &::-webkit-slider-thumb {
              appearance: none;
              width: 0.75rem;
              height: 0.75rem;
              border-radius: 50%;
              background: white;
              cursor: pointer;
              margin-top: -0.25rem;
              box-shadow: 0 0.125rem 0.25rem rgba(0, 0, 0, 0.3);
            }

            &::-webkit-slider-runnable-track {
              width: 100%;
              height: 0.25rem;
              background: $style-primary;
              border-radius: 0.125rem;
              cursor: pointer;
            }

            &::-moz-range-thumb {
              width: 0.75rem;
              height: 0.75rem;
              border: none;
              border-radius: 50%;
              background: white;
              cursor: pointer;
              box-shadow: 0 0.125rem 0.25rem rgba(0, 0, 0, 0.3);
            }

            &::-moz-range-track {
              width: 100%;
              height: 0.25rem;
              background: $style-primary;
              border-radius: 0.125rem;
              cursor: pointer;
            }
          }
        }

        .select {
          position: relative;
          width: 7.5rem;
          flex-shrink: 0;

          select {
            width: 100%;
            height: 1.875rem;
            padding: 0 0.625rem;
            border: 0.0625rem solid transparent;
            border-radius: 0.25rem;
            background-color: $background-tertiary;
            color: white;
            font-size: 0.75rem;
            font-weight: 500;
            cursor: pointer;
            appearance: none;
            transition: border-color 0.2s ease;

            &:hover {
              border-color: $background-secondary;
            }

            option {
              color: $text-normal;
              background-color: $background-tertiary;
              font-weight: normal;
            }
          }

          &::after {
            content: '\f078';
            font-family: 'Font Awesome 6 Free';
            font-weight: 900;
            position: absolute;
            right: 0.625rem;
            top: 50%;
            transform: translateY(-50%);
            pointer-events: none;
            font-size: 0.75rem;
            opacity: 0.6;
          }
        }

        .input {
          width: 100%;
          height: 2rem;
          padding: 0 0.625rem;
          border: 0.0625rem solid transparent;
          border-radius: 0.25rem;
          background-color: $background-tertiary;
          color: $text-normal;
          font-size: 0.875rem;
          transition: border-color 0.2s ease;

          &:hover,
          &:focus {
            border-color: $background-secondary;
          }

          &:disabled {
            opacity: 0.5;
            cursor: not-allowed;
          }

          &::placeholder {
            color: rgba($text-normal, 0.5);
          }
        }
      }
    }
  }
</style>

<script lang="ts">
  import { Component, Vue, Watch } from 'vue-property-decorator'

  @Component({ name: 'neko-settings' })
  export default class extends Vue {
    private broadcast_url: string = ''

    get admin() {
      return this.$accessor.user.admin
    }

    get connected() {
      return this.$accessor.connected
    }

    get scroll() {
      return this.$accessor.settings.scroll.toString()
    }

    set scroll(value: string) {
      this.$accessor.settings.setScroll(parseInt(value))
    }

    get scroll_invert() {
      return this.$accessor.settings.scroll_invert
    }

    set scroll_invert(value: boolean) {
      this.$accessor.settings.setInvert(value)
    }

    get autoplay() {
      return this.$accessor.settings.autoplay
    }

    set autoplay(value: boolean) {
      this.$accessor.settings.setAutoplay(value)
    }

    get ignore_emotes() {
      return this.$accessor.settings.ignore_emotes
    }

    set ignore_emotes(value: boolean) {
      this.$accessor.settings.setIgnore(value)
    }

    get chat_sound() {
      return this.$accessor.settings.chat_sound
    }

    set chat_sound(value: boolean) {
      this.$accessor.settings.setSound(value)
    }

    get keyboard_layouts_list() {
      return this.$accessor.settings.keyboard_layouts_list
    }

    get keyboard_layout() {
      return this.$accessor.settings.keyboard_layout
    }

    get broadcast_is_active() {
      return this.$accessor.settings.broadcast_is_active
    }

    get broadcast_url_remote() {
      return this.$accessor.settings.broadcast_url
    }

    @Watch('broadcast_url_remote', { immediate: true })
    onBroadcastUrlChange() {
      this.broadcast_url = this.broadcast_url_remote
    }

    set keyboard_layout(value: string) {
      this.$accessor.settings.setKeyboardLayout(value)
      this.$accessor.remote.changeKeyboard()
    }

    logout() {
      this.$accessor.logout()
    }
  }
</script>
