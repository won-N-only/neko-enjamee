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
      <i class="fas fa-angle-double-down" @click="() => { _history.scrollTop = _history.scrollHeight }" />
    </div>
    <div v-if="!muted" class="chat-send">
      <div class="accent" />
      <div class="text-container">
        <textarea ref="input" :placeholder="$t('send_a_message')" @keydown="onKeyDown" v-model="content" />
        <neko-emoji v-if="emoji" @picked="onEmojiPicked" @done="emoji = false" />
        <li>
          <i class="emoji-menu fas fa-laugh" @click.stop.prevent="onEmoji"></i>
        </li>
      </div>
      <input ref="hinput" type="text"/>
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
      position: absolute;
      bottom: 100px;
      right: 20px;
      z-index: 1;
      cursor: pointer;
      color: $text-muted;
      font-size: 20px;
      transition: color 0.2s ease-in-out;

      &:hover {
        color: $text-normal;
      }
    }

    .chat-send {
      flex-shrink: 0;
      height: 80px;
      max-height: 80px;
      padding: 0 10px 10px 10px;
      flex-direction: column;
      display: flex;

      .accent {
        width: 100%;
        height: 1px;
        background: rgba($color: #fff, $alpha: 0.05);
        margin: 5px 0 10px 0;
      }

      input {
          height: 0;
          opacity: 0;
          font-size: 16px;
          pointer-events: none;
        }

      .text-container {
        flex: 1;
        width: 100%;
        height: 100%;
        background-color: rgba($color: #fff, $alpha: 0.05);
        border-radius: 5px;
        position: relative;
        display: flex;
        
        li {
          display: inline-block;
        }

        .emoji-menu {
          width: 20px;
          height: 20px;
          font-size: 20px;
          margin: 8px 5px 0 0;
          cursor: pointer;
        }

        .clear-button {
          width: 20px;
          height: 20px;
          font-size: 20px;
          margin: 8px 5px 0 0;
          cursor: pointer;
        }

        textarea {
          flex: 1;
          font-family: $text-family;
          border: none;
          caret-color: $text-normal;
          color: $text-normal;
          resize: none;
          margin: 5px;
          background-color: transparent;
          scrollbar-width: thin;
          scrollbar-color: $background-tertiary transparent;

          &::placeholder {
            color: $text-muted;
          }

          &::-webkit-scrollbar {
            width: 4px;
          }

          &::-webkit-scrollbar-track {
            background-color: transparent;
          }

          &::-webkit-scrollbar-thumb {
            background-color: $background-tertiary;
            border-radius: 4px;
          }

          &::-webkit-scrollbar-thumb:hover {
            background-color: $background-floating;
          }

          &::selection {
            background: $text-link;
          }
        }

      }
    }
  }
</style>

<script lang="ts">
  import { Component, Ref, Watch, Vue } from 'vue-property-decorator'
  import { formatRelative } from 'date-fns'

  import { Member } from '~/neko/types'

  import Markdown from './markdown'
  import Content from './context.vue'
  import Emoji from './emoji.vue'
  import Avatar from './avatar.vue'

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
      this.$nextTick(() => {
        if (this._history.scrollTop + this._history.clientHeight >= this._history.scrollHeight - 100) {
          this._history.scrollTop = this._history.scrollHeight
        }

        if (this.history.length > 200) {
          this.history.splice(0, this.history.length - 200)
        }
      })
    }

    @Watch('muted')
    onMutedChange(muted: boolean) {
      if (muted) {
        this.content = ''
      }
    }

    mounted() {
      this.$nextTick(() => {
        this._history.scrollTop = this._history.scrollHeight
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
      // Do nothing if user is muted by admin
      if (this.muted) {
        return
      }

      // Workaround: ignore IME composing event
      if (event.isComposing || event.key === 'Process') {
        return
      }

      if (event.key === 'Enter' && !event.shiftKey) {
        // Prevent enter keypress event
        event.preventDefault()

        // Workaround: iOS IME CJK compositing buffer bug
        this._hinput.focus()
        this._input.focus()

        // Check if text is empty
        if (this.content.length === 0) {
          return
        }

        // Cut message if it's over limit and notify to user
        if (this.content.length > length) {
          this.content = this.content.substring(0, length)
          return
        }

        this.$accessor.chat.sendMessage(this.content)
        this.content = ''
        this.$nextTick(() => {
          this._history.scrollTop = this._history.scrollHeight
        })

        return
      }
    }
  }
</script>
