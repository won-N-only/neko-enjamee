<template>
  <div class="chat">
    <ul class="chat-history" ref="history" @click="onClick">
      <template v-for="(message, index) in history">
        <li
          :key="index"
          class="message"
          v-if="message.type === 'text'"
          :class="{
            bulk: index > 0 && history[index - 1].id == message.id && history[index - 1].type === 'text',
          }"
        >
          <div class="author" @contextmenu.stop.prevent="onContext($event, { member: member(message.id) })">
            <neko-avatar class="avatar" :seed="member(message.id).displayname" :size="40" />
          </div>
          <div class="content">
            <div class="content-head">
              <span>{{ member(message.id).displayname }}</span>
              <span class="timestamp">{{ timestamp(message.created) }}</span>
            </div>
            <neko-markdown class="content-body" :source="message.content" />
          </div>
        </li>
        <li :key="index" class="event" v-if="message.type === 'event'">
          <div
            class="content"
            v-tooltip="{
              content: timestamp(message.created),
              placement: 'left',
              offset: 3,
              boundariesElement: 'body',
            }"
          >
            <strong v-if="message.id === id && $te('you')">{{ $t('you') }}</strong>
            <strong v-else>{{ member(message.id).displayname }}</strong>
            {{ message.content }}
          </div>
        </li>
      </template>
    </ul>
    <neko-context ref="context" />
    <div class="chat-scroll-to-bottom">
      <i
        class="fas fa-angle-double-down"
        @click="
          () => {
            _history.scrollTop = _history.scrollHeight
          }
        "
      />
    </div>
    <div v-if="!muted" class="chat-send">
      <div class="accent" />
      <div class="text-container">
        <textarea ref="input" :placeholder="String($t('send_a_message'))" @keydown="onKeyDown" v-model.lazy="content" />
        <neko-emoji v-if="emoji" @picked="onEmojiPicked" @done="emoji = false" />
        <li>
          <i class="emoji-menu fas fa-laugh" @click.stop.prevent="onEmoji"></i>
        </li>
      </div>
      <input ref="hinput" type="text" />
    </div>
  </div>
</template>

<style lang="scss" scoped>
  .chat {
    display: flex;
    flex-direction: column;
    height: 100%;
    width: 100%;
    overflow: hidden;

    .chat-history {
      flex: 1;
      display: flex;
      flex-direction: column;
      overflow-y: auto;
      scrollbar-width: thin;
      scrollbar-color: $background-tertiary transparent;

      li {
        display: flex;
        gap: 0.75rem;
        padding: 0.75rem;
        border-radius: 0.5rem;
        background: rgba($background-primary, 0.5);

        &.message {
          padding: 0.25rem 0.75rem;

          .author {
            flex-shrink: 0;
            width: 2.5rem;
            height: 2.5rem;
            border-radius: 50%;
            overflow: hidden;

            .avatar {
              width: 100%;
              height: 100%;
              object-fit: cover;
            }
          }

          .content {
            flex: 1;
            display: flex;
            flex-direction: column;
            gap: 0.5rem;
            min-width: 0;

            .content-head {
              display: flex;
              align-items: baseline;
              gap: 0.5rem;
              flex-wrap: wrap;

              span {
                font-size: 1rem;
                line-height: 1.2;
              }

              .timestamp {
                font-size: 0.875rem;
                color: $text-muted;
              }
            }

            .content-body {
              font-size: 1rem;
              line-height: 1.5;
            }
          }

          &.bulk {
            padding: 0.25rem 0.75rem;
            margin: 0rem;

            .author {
              visibility: hidden;
              height: 0;
            }

            .content-head {
              display: none;
            }
          }
        }

        &.event {
          justify-content: center;
          text-align: center;
          font-size: 0.875rem;
          color: $text-muted;
          padding: 0.5rem;
        }
      }
    }

    .chat-scroll-to-bottom {
      position: fixed;
      bottom: 5rem;
      right: 1rem;
      width: 2.5rem;
      height: 2.5rem;
      display: flex;
      align-items: center;
      justify-content: center;
      border-radius: 50%;
      background: $background-primary;
      box-shadow: 0 0.125rem 0.625rem rgba(0, 0, 0, 0.1);
      cursor: pointer;
      opacity: 0.8;
      transition: all 0.2s ease;

      i {
        font-size: 1.25rem;
      }

      &:hover {
        opacity: 1;
        transform: translateY(-0.125rem);
      }
    }

    .chat-send {
      flex-shrink: 0;
      display: flex;
      flex-direction: column;
      gap: 0.5rem;
      padding: 0.75rem;
      background: $background-primary;

      .accent {
        height: 0.0625rem;
        background: rgba(255, 255, 255, 0.05);
      }

      .text-container {
        display: flex;
        gap: 0.5rem;
        padding: 0.5rem;
        border-radius: 0.5rem;
        background: rgba(255, 255, 255, 0.05);

        textarea {
          flex: 1;
          font-size: 1rem;
          line-height: 1.5;
          padding: 0.5rem;
          height: 2.5rem;
          resize: none;
          border: none;
          background: transparent;
          color: inherit;
          overflow-y: hidden;
          transition: height 0.2s ease;

          &:focus {
            height: auto;
            overflow-y: auto;
          }

          &::placeholder {
            color: $text-muted;
          }
        }

        li {
          display: flex;
          align-items: flex-start;
          padding-top: 0.5rem;
        }

        .emoji-menu {
          font-size: 1.25rem;
          padding: 0.5rem;
          cursor: pointer;
          transition: transform 0.2s ease;
          color: $text-muted;

          &:hover {
            transform: scale(1.1);
            color: $text-normal;
          }
        }
      }

      input[type='text'] {
        position: absolute;
        opacity: 0;
        pointer-events: none;
      }
    }
  }
</style>

<script lang="ts">
  import { formatRelative } from 'date-fns'
  import { Component, Ref, Vue, Watch } from 'vue-property-decorator'

  import { Member } from '~/neko/types'

  import Avatar from './avatar.vue'
  import Content from './context.vue'
  import Emoji from './emoji.vue'
  import Markdown from './markdown'

  const length = 512 // max length of message

  @Component({
    name: 'neko-chat',
    components: {
      'neko-markdown': Markdown,
      'neko-context': Content,
      'neko-emoji': Emoji,
      'neko-avatar': Avatar,
    },
  })
  export default class extends Vue {
    @Ref('input') readonly _input!: HTMLTextAreaElement
    @Ref('hinput') readonly _hinput!: HTMLInputElement
    @Ref('history') readonly _history!: HTMLElement
    @Ref('context') readonly _context!: any

    emoji = false
    content = ''

    get id() {
      return this.$accessor.user.id
    }

    get muted() {
      return this.$accessor.user.muted
    }

    get history() {
      return this.$accessor.chat.history
    }

    @Watch('history')
    onHistroyChange() {
      if (this._history && this._history.scrollTop + this._history.clientHeight >= this._history.scrollHeight - 100) {
        this.$nextTick(() => {
          this._history.scrollTop = this._history.scrollHeight
        })
      }

      if (this.history.length > 200) {
        this.$nextTick(() => {
          this.history.splice(0, this.history.length - 200)
        })
      }
    }

    @Watch('muted')
    onMutedChange(muted: boolean) {
      if (muted) {
        this.content = ''
      }
    }

    mounted() {
      this.$nextTick(() => {
        if (this._history) {
          this._history.scrollTop = this._history.scrollHeight
        }
      })
    }

    member(id: string) {
      return this.$accessor.user.members[id] || { id, displayname: this.$t('somebody') }
    }

    timestamp(time: Date) {
      const str = formatRelative(time, new Date())
      return `${str.charAt(0).toUpperCase()}${str.slice(1)}`
    }

    onEmoji() {
      this.emoji = !this.emoji
      this._input.focus()
    }

    onEmojiPicked(emoji: string) {
      const text = `:${emoji}:`
      if (this._input.selectionStart || this._input.selectionStart === 0) {
        var startPos = this._input.selectionStart
        var endPos = this._input.selectionEnd
        this.content = this.content.substring(0, startPos) + text + this.content.substring(endPos, this.content.length)
        this.$nextTick(() => {
          this._input.selectionStart = startPos + text.length
          this._input.selectionEnd = startPos + text.length
        })
      } else {
        this.content += text
      }
      this._input.focus()
      this.emoji = false
    }

    onContext(event: MouseEvent, { member }: { member: Member }) {
      if (member.id === this.id) {
        return
      }
      this._context.open(event, { member })
    }

    onClick(event: { target?: HTMLElement; preventDefault(): void }) {
      const { target } = event
      if (!target) {
        return
      }

      if (target.tagName.toLowerCase() === 'span' && target.classList.contains('spoiler')) {
        target.classList.add('active')
        event.preventDefault()
      }

      if (!target.parentElement) {
        return
      }

      if (target.parentElement.tagName.toLowerCase() === 'span' && target.parentElement.classList.contains('spoiler')) {
        target.parentElement.classList.add('active')
        event.preventDefault()
      }
    }

    onKeyDown(event: KeyboardEvent) {
      if (this.muted) {
        return
      }

      if (event.isComposing || event.key === 'Process') {
        return
      }

      if (event.key === 'Enter' && !event.shiftKey) {
        event.preventDefault()

        this._hinput.focus()
        this._input.focus()

        if (this.content.length === 0) {
          return
        }

        if (this.content.length > length) {
          this.content = this.content.substring(0, length)
          return
        }

        this.$accessor.chat.sendMessage(this.content)
        this.content = ''

        this.$nextTick(() => {
          this._history.scrollTop = this._history.scrollHeight
        })
      }
    }
  }
</script>
