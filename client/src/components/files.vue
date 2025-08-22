<template>
  <div class="files">
    <div class="files-cwd">
      <p>{{ cwd }}</p>
      <i class="fas fa-rotate-right refresh" @click="refresh" />
    </div>
    <div class="files-list">
      <div v-for="item in files" :key="item.name" class="files-list-item">
        <i :class="fileIcon(item)" />
        <p class="file-name" :title="item.name">{{ item.name }}</p>
        <p class="file-size">{{ fileSize(item.size) }}</p>
        <i v-if="item.type !== 'dir'" class="fas fa-download download" @click="download(item)" />
      </div>
    </div>
    <div class="transfer-area">
      <div class="transfers" v-if="transfers.length > 0">
        <p v-if="downloads.length > 0" class="transfers-list-header">
          <span>{{ $t('files.downloads') }}</span>
          <i class="fas fa-xmark remove-transfer" @click="downloads.forEach((t) => removeTransfer(t))"></i>
        </p>
        <div v-for="download in downloads" :key="download.id" class="transfers-list-item">
          <div class="transfer-info">
            <i
              class="fas transfer-status"
              :class="{
                'fa-clock': download.status === 'pending',
                'fa-arrows-rotate': download.status === 'inprogress',
                'fa-check': download.status === 'completed',
                'fa-warning': download.status === 'failed',
              }"
            ></i>
            <p class="file-name" :title="download.name">{{ download.name }}</p>
            <p class="file-size">{{ Math.min(100, Math.round((download.progress / download.size) * 100)) }}%</p>
            <i class="fas fa-xmark remove-transfer" @click="removeTransfer(download)"></i>
          </div>
          <div v-if="download.status === 'failed'" class="transfer-error">{{ download.error }}</div>
          <progress
            v-else
            class="transfer-progress"
            :aria-label="download.name + ' progress'"
            :value="download.progress"
            :max="download.size"
          ></progress>
        </div>
        <p v-if="uploads.length > 0" class="transfers-list-header">
          <span>{{ $t('files.uploads') }}</span>
          <i class="fas fa-xmark remove-transfer" @click="uploads.forEach((t) => removeTransfer(t))"></i>
        </p>
        <div v-for="upload in uploads" :key="upload.id" class="transfers-list-item">
          <div class="transfer-info">
            <i
              class="fas transfer-status"
              :title="upload.status"
              :class="{
                'fa-clock': upload.status === 'pending',
                'fa-arrows-rotate': upload.status === 'inprogress',
                'fa-check': upload.status === 'completed',
                'fa-warning': upload.status === 'failed',
              }"
            ></i>
            <p class="file-name" :title="upload.name">{{ upload.name }}</p>
            <p class="file-size">{{ Math.min(100, Math.round((upload.progress / upload.size) * 100)) }}%</p>
            <i class="fas fa-xmark remove-transfer" @click="removeTransfer(upload)"></i>
          </div>
          <div v-if="upload.status === 'failed'" class="transfer-error">{{ upload.error }}</div>
          <progress
            v-else
            class="transfer-progress"
            :aria-label="upload.name + ' progress'"
            :value="upload.progress"
            :max="upload.size"
          ></progress>
        </div>
      </div>
      <div
        class="upload-area"
        :class="{ 'upload-area-drag': uploadAreaDrag }"
        @dragover.prevent="uploadAreaDrag = true"
        @dragleave.prevent="uploadAreaDrag = false"
        @drop.prevent="(e) => upload(e.dataTransfer)"
        @click="openFileBrowser"
      >
        <i class="fas fa-file-arrow-up" />
        <p>{{ $t('files.upload_here') }}</p>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
  .files {
    display: flex;
    flex-direction: column;
    width: 100%;
    height: 100%;
    gap: 0.625rem;
    padding: 0.625rem;

    .files-cwd {
      display: flex;
      align-items: center;
      gap: 0.625rem;
      padding: 0.5rem;
      font-weight: 600;
      background-color: rgba(255, 255, 255, 0.05);
      border-radius: 0.3125rem;

      p {
        flex: 1;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
      }

      .refresh {
        cursor: pointer;
        transition: transform 0.2s ease;

        &:hover {
          transform: rotate(45deg);
        }
      }
    }

    .files-list {
      flex: 1;
      display: flex;
      flex-direction: column;
      background-color: rgba(255, 255, 255, 0.05);
      border-radius: 0.3125rem;
      overflow-y: auto;
      scrollbar-width: thin;
      scrollbar-color: $background-tertiary transparent;

      &::-webkit-scrollbar {
        width: 0.5rem;
      }

      &::-webkit-scrollbar-track {
        background-color: transparent;
      }

      &::-webkit-scrollbar-thumb {
        background-color: $background-tertiary;
        border: 0.125rem solid $background-primary;
        border-radius: 0.25rem;
      }

      &::-webkit-scrollbar-thumb:hover {
        background-color: $background-floating;
      }

      .files-list-item {
        display: flex;
        align-items: center;
        gap: 0.5rem;
        padding: 0.5rem;
        border-bottom: 0.125rem solid rgba(255, 255, 255, 0.1);

        &:last-child {
          border-bottom: none;
        }

        i {
          flex-shrink: 0;
          width: 1rem;
          font-size: 0.875rem;

          &.download {
            cursor: pointer;
            opacity: 0.6;
            transition: opacity 0.2s ease;

            &:hover {
              opacity: 1;
            }
          }
        }

        .file-name {
          flex: 1;
          min-width: 0;
          text-overflow: ellipsis;
          overflow: hidden;
          white-space: nowrap;
        }

        .file-size {
          flex-shrink: 0;
          color: rgba(255, 255, 255, 0.4);
          margin-left: 0.5rem;
        }
      }
    }

    .transfer-area {
      display: flex;
      flex-direction: column;
      gap: 0.625rem;
      margin-top: auto;

      .transfers {
        display: flex;
        flex-direction: column;
        background-color: rgba(255, 255, 255, 0.05);
        border-radius: 0.3125rem;
        max-height: 50vh;
        overflow-y: auto;
        scrollbar-width: thin;
        scrollbar-color: $background-tertiary transparent;

        &::-webkit-scrollbar {
          width: 0.5rem;
        }

        &::-webkit-scrollbar-track {
          background-color: transparent;
        }

        &::-webkit-scrollbar-thumb {
          background-color: $background-tertiary;
          border: 0.125rem solid $background-primary;
          border-radius: 0.25rem;
        }

        &::-webkit-scrollbar-thumb:hover {
          background-color: $background-floating;
        }

        .transfers-list-header {
          display: flex;
          align-items: center;
          justify-content: space-between;
          padding: 0.625rem;
          font-weight: 600;
          border-bottom: 0.125rem solid rgba(255, 255, 255, 0.1);

          .remove-transfer {
            cursor: pointer;
            opacity: 0.6;
            transition: opacity 0.2s ease;

            &:hover {
              opacity: 1;
            }
          }
        }

        .transfers-list-item {
          display: flex;
          flex-direction: column;
          gap: 0.5rem;
          padding: 0.625rem;

          .transfer-info {
            display: flex;
            align-items: center;
            gap: 0.5rem;

            .transfer-status {
              flex-shrink: 0;
              width: 1rem;
              font-size: 0.875rem;
            }

            .file-name {
              flex: 1;
              min-width: 0;
              text-overflow: ellipsis;
              overflow: hidden;
              white-space: nowrap;
            }

            .file-size {
              flex-shrink: 0;
            }

            .remove-transfer {
              flex-shrink: 0;
              cursor: pointer;
              opacity: 0.6;
              transition: opacity 0.2s ease;

              &:hover {
                opacity: 1;
              }
            }
          }

          .transfer-error {
            padding: 0.625rem;
            border: 0.0625rem solid $style-error;
            border-radius: 0.3125rem;
            color: $style-error;
          }

          .transfer-progress {
            width: 100%;
            height: 0.5rem;
            border-radius: 0.25rem;
            overflow: hidden;

            &::-webkit-progress-bar {
              background-color: rgba(255, 255, 255, 0.1);
            }

            &::-webkit-progress-value {
              background-color: $style-primary;
            }

            &::-moz-progress-bar {
              background-color: $style-primary;
            }
          }
        }
      }

      .upload-area {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        gap: 0.625rem;
        padding: 1.5rem;
        background-color: rgba(255, 255, 255, 0.05);
        border-radius: 0.3125rem;
        cursor: pointer;
        transition: background-color 0.2s ease;

        &:hover,
        &.upload-area-drag {
          background-color: rgba(255, 255, 255, 0.1);
        }

        i {
          font-size: 2rem;
          opacity: 0.6;
        }

        p {
          font-size: 1rem;
          opacity: 0.8;
        }
      }
    }
  }
</style>

<script lang="ts">
  import { Component, Vue } from 'vue-property-decorator'

  import { FileListItem, FileTransfer } from '~/neko/types'
  import Content from './context.vue'
  import Markdown from './markdown'

  @Component({
    name: 'neko-files',
    components: {
      'neko-markdown': Markdown,
      'neko-context': Content,
    },
  })
  export default class extends Vue {
    public uploadAreaDrag: boolean = false

    get cwd() {
      return this.$accessor.files.cwd
    }

    get files() {
      return this.$accessor.files.files
    }

    get transfers() {
      return this.$accessor.files.transfers
    }

    get downloads() {
      return this.$accessor.files.transfers.filter((t) => t.direction === 'download')
    }

    get uploads() {
      return this.$accessor.files.transfers.filter((t) => t.direction === 'upload')
    }

    refresh() {
      this.$accessor.files.refresh()
    }

    download(item: FileListItem) {
      if (this.downloads.map((t) => t.name).includes(item.name)) {
        return
      }

      const url =
        '/file?pwd=' + encodeURIComponent(this.$accessor.password) + '&filename=' + encodeURIComponent(item.name)
      const abortController = new AbortController()

      let transfer: FileTransfer = {
        id: Math.round(Math.random() * 10000),
        name: item.name,
        direction: 'download',
        // this may be smaller than the actual transfer amount, but for large files the
        // content length is not sent (chunked transfer)
        size: item.size,
        progress: 0,
        status: 'pending',
        abortController: abortController,
      }

      this.$http
        .get(url, {
          responseType: 'blob',
          signal: abortController.signal,
          withCredentials: false,
          onDownloadProgress: (x) => {
            transfer.progress = x.loaded

            if (x.total && transfer.size !== x.total) {
              transfer.size = x.total
            }
            if (transfer.progress === transfer.size) {
              transfer.status = 'completed'
            } else if (transfer.status !== 'inprogress') {
              transfer.status = 'inprogress'
            }
          },
        })
        .then((res) => {
          const url = window.URL.createObjectURL(new Blob([res.data]))
          const link = document.createElement('a')
          link.href = url
          link.setAttribute('download', item.name)
          document.body.appendChild(link)
          link.click()
          document.body.removeChild(link)

          transfer.progress = transfer.size
          transfer.status = 'completed'
        })
        .catch((error) => {
          this.$log.error(error)

          transfer.status = 'failed'
          transfer.error = error.message
        })

      this.$accessor.files.addTransfer(transfer)
    }

    upload(dt: DataTransfer) {
      const url = '/file?pwd=' + encodeURIComponent(this.$accessor.password)
      this.uploadAreaDrag = false

      for (const file of dt.files) {
        const abortController = new AbortController()

        const formdata = new FormData()
        formdata.append('files', file, file.name)

        let transfer: FileTransfer = {
          id: Math.round(Math.random() * 10000),
          name: file.name,
          direction: 'upload',
          size: file.size,
          progress: 0,
          status: 'pending',
          abortController: abortController,
        }

        this.$http
          .post(url, formdata, {
            signal: abortController.signal,
            withCredentials: false,
            onUploadProgress: (x: any) => {
              transfer.progress = x.loaded

              if (transfer.size !== x.total) {
                transfer.size = x.total
              }
              if (transfer.progress === transfer.size) {
                transfer.status = 'completed'
              } else if (transfer.status !== 'inprogress') {
                transfer.status = 'inprogress'
              }
            },
          })
          .catch((error) => {
            this.$log.error(error)

            transfer.status = 'failed'
            transfer.error = error.message
          })

        this.$accessor.files.addTransfer(transfer)
      }
    }

    openFileBrowser() {
      const input = document.createElement('input')
      input.type = 'file'
      input.setAttribute('multiple', 'true')
      input.onchange = (e: Event) => {
        if (e === null) return

        const dt = new DataTransfer()
        const target = e.target as HTMLInputElement
        if (target.files === null) return

        for (const f of target.files) {
          dt.items.add(f)
        }

        this.upload(dt)
      }
      input.click()
    }

    removeTransfer(transfer: FileTransfer) {
      if (transfer.status !== 'completed') {
        transfer.abortController?.abort()
      }
      this.$accessor.files.removeTransfer(transfer)
    }

    fileIcon(file: FileListItem) {
      let className = 'file-icon fas '
      // if is directory
      if (file.type === 'dir') {
        className += 'fa-folder'
        return className
      }
      // try to get file extension
      const ext = file.name.split('.').pop()
      if (ext === undefined) {
        className += 'fa-file'
        return className
      }
      // try to find icon
      switch (ext.toLowerCase()) {
        case 'txt':
        case 'md':
          className += 'fa-file-text'
          break
        case 'pdf':
          className += 'fa-file-pdf'
          break
        case 'zip':
        case 'rar':
        case '7z':
        case 'gz':
          className += 'fa-archive'
          break
        case 'aac':
        case 'flac':
        case 'midi':
        case 'mp3':
        case 'ogg':
        case 'wav':
          className += 'fa-music'
          break
        case 'avi':
        case 'mkv':
        case 'mov':
        case 'mpeg':
        case 'mp4':
        case 'webm':
          className += 'fa-film'
          break
        case 'bmp':
        case 'gif':
        case 'jpeg':
        case 'jpg':
        case 'png':
        case 'svg':
        case 'tiff':
        case 'webp':
          className += 'fa-image'
          break
        default:
          className += 'fa-file'
      }
      return className
    }

    fileSize(size: number) {
      if (size < 1024) {
        return size + ' B'
      }
      if (size < 1024 * 1024) {
        return Math.round(size / 1024) + ' KB'
      }
      if (size < 1024 * 1024 * 1024) {
        return Math.round(size / (1024 * 1024)) + ' MB'
      }
      if (size < 1024 * 1024 * 1024 * 1024) {
        return Math.round(size / (1024 * 1024 * 1024)) + ' GB'
      }
      return Math.round(size / (1024 * 1024 * 1024 * 1024)) + ' TB'
    }
  }
</script>
