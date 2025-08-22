<template>
  <aside class="neko-menu">
    <div class="tabs-container">
      <ul>
        <li :class="{ active: tab === 'chat' }" @click.stop.prevent="change('chat')">
          <i class="fas fa-comment-alt" />
          <span>{{ $t('side.chat') }}</span>
        </li>
        <li v-if="filetransferAllowed" :class="{ active: tab === 'files' }" @click.stop.prevent="change('files')">
          <i class="fas fa-file" />
          <span>{{ $t('side.files') }}</span>
        </li>
        <li :class="{ active: tab === 'settings' }" @click.stop.prevent="change('settings')">
          <i class="fas fa-sliders-h" />
          <span>{{ $t('side.settings') }}</span>
        </li>
      </ul>
    </div>
    <div class="page-container">
      <neko-chat v-if="tab === 'chat'" />
      <neko-files v-if="tab === 'files'" />
      <neko-settings v-if="tab === 'settings'" />
    </div>
  </aside>
</template>

<style lang="scss">
  .neko-menu {
    width: $side-width;
    background-color: $background-primary;
    flex: 0 0 auto;
    height: 100%;
    display: flex;
    flex-direction: column;

    .tabs-container {
      background: $background-tertiary;
      height: $menu-height;
      flex: 0 0 auto;
      display: flex;
      align-items: flex-end;
      padding: 0 1rem;

      ul {
        display: flex;
        gap: 0.25rem;
        width: 100%;
        padding: 0.5rem 0 0 0;

        li {
          display: flex;
          align-items: center;
          gap: 0.25rem;
          background: $background-secondary;
          border-radius: 0.25rem 0.25rem 0 0;
          padding: 0.5rem 0.75rem;
          font-weight: 600;
          cursor: pointer;
          transition: background-color 0.2s ease;

          i {
            font-size: 0.875rem;
          }

          &.active {
            background: $background-primary;
          }

          &:hover:not(.active) {
            background: lighten($background-secondary, 5%);
          }
        }
      }
    }

    .page-container {
      flex: 1;
      display: flex;
      overflow: hidden;
      padding: 0.5rem;

      > * {
        flex: 1;
        display: flex;
        flex-direction: column;
      }
    }
  }
</style>

<script lang="ts">
  import { Component, Vue, Watch } from 'vue-property-decorator'

  import Chat from '~/components/chat.vue'
  import Files from '~/components/files.vue'
  import Settings from '~/components/settings.vue'

  @Component({
    name: 'neko',
    components: {
      'neko-settings': Settings,
      'neko-chat': Chat,
      'neko-files': Files,
    },
  })
  export default class extends Vue {
    get filetransferAllowed() {
      return (
        this.$accessor.remote.fileTransfer && (this.$accessor.user.admin || !this.$accessor.isLocked('file_transfer'))
      )
    }

    get tab() {
      return this.$accessor.client.tab
    }

    @Watch('tab', { immediate: true })
    @Watch('filetransferAllowed', { immediate: true })
    onTabChange() {
      // do not show the files tab if file transfer is disabled
      if (this.tab === 'files' && !this.filetransferAllowed) {
        this.change('chat')
      }
    }

    @Watch('filetransferAllowed')
    onFileTransferAllowedChange() {
      if (this.filetransferAllowed) {
        this.$accessor.files.refresh()
      }
    }

    change(tab: string) {
      this.$accessor.client.setTab(tab)
    }
  }
</script>
