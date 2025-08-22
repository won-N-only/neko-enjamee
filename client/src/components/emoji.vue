<template>
  <div class="neko-emoji" v-on-clickaway="onClickAway">
    <div class="search">
      <div class="search-contianer">
        <input type="text" ref="search" v-model="search" />
      </div>
    </div>
    <div class="list" ref="scroll" @scroll="onScroll">
      <ul :class="['group-list']" :style="{ display: search === '' ? 'flex' : 'none' }">
        <li v-for="(group, index) in groups" :key="index" class="group" ref="groups">
          <span class="label">{{ group.name }}</span>
          <ul class="emoji-list">
            <li
              v-for="emoji in index === 0 ? recent : group.list"
              :key="`${group.id}-${emoji}`"
              :class="['emoji-container', hovered === emoji ? 'active' : '']"
            >
              <span
                :class="['emoji']"
                @mouseenter.stop.prevent="onMouseEnter($event, emoji)"
                @click.stop.prevent="onClick($event, emoji)"
                :data-emoji="emoji"
              ></span>
            </li>
          </ul>
        </li>
      </ul>
      <ul :class="['emoji-container']" :style="{ display: search === '' ? 'none' : 'flex' }">
        <li v-for="emoji in filtered" :key="emoji" :class="['emoji-item', hovered === emoji ? 'active' : '']">
          <span
            :class="['emoji']"
            @mouseenter.stop.prevent="onMouseEnter($event, emoji)"
            @click.stop.prevent="onClick($event, emoji)"
            :data-emoji="emoji"
          ></span>
        </li>
      </ul>
    </div>
    <div class="details">
      <div class="details-container" v-if="hovered !== ''">
        <span :class="['emoji']" :data-emoji="hovered" /><span class="emoji-id">:{{ hovered }}:</span>
      </div>
    </div>
    <div class="groups">
      <ul>
        <li
          v-for="(group, index) in groups"
          :key="index"
          :class="[group.id, active.id === group.id && search === '' ? 'active' : '']"
          @click.stop.prevent="scrollTo($event, index)"
        >
          <span :class="[`group-${group.id} fas`]" />
        </li>
      </ul>
    </div>
  </div>
</template>

<style lang="scss" scoped>
  $emoji-width: 18.75rem;

  .neko-emoji {
    position: absolute;
    z-index: 10000;
    width: $emoji-width;
    height: 21.875rem;
    background: $background-secondary;
    bottom: 4.6875rem;
    right: 0.3125rem;
    display: flex;
    flex-direction: column;
    border-radius: 0.3125rem;
    overflow: hidden;
    box-shadow: $elevation-high;

    .search {
      display: flex;
      padding: 0.625rem;
      border-bottom: 0.0625rem solid $background-tertiary;

      .search-contianer {
        display: flex;
        flex: 1;
        position: relative;
        border-radius: 0.3125rem;
        overflow: hidden;

        &::before {
          content: '\f002';
          font-weight: 900;
          font-family: 'Font Awesome 6 Free';
          position: absolute;
          display: flex;
          align-items: center;
          justify-content: center;
          width: 1rem;
          height: 1rem;
          top: 0.375rem;
          right: 0.375rem;
          opacity: 0.5;
        }

        input {
          flex: 1;
          border: none;
          background-color: $background-floating;
          color: $interactive-normal;
          padding: 0.5rem;
          font-weight: 500;

          &::placeholder {
            color: $text-muted;
            font-weight: 500;
          }
        }
      }
    }

    .list {
      flex: 1;
      display: flex;
      flex-direction: column;
      overflow-y: auto;
      overflow-x: hidden;
      scrollbar-width: thin;
      scrollbar-color: $background-tertiary transparent;
      scroll-behavior: smooth;
      padding: 0.3125rem;

      &::-webkit-scrollbar {
        width: 0.25rem;
      }

      &::-webkit-scrollbar-track {
        background-color: transparent;
      }

      &::-webkit-scrollbar-thumb {
        background-color: $background-tertiary;
        border-radius: 0.25rem;
      }

      &::-webkit-scrollbar-thumb:hover {
        background-color: $background-floating;
      }

      .group-list {
        display: flex;
        flex-direction: column;
        width: 100%;
        gap: 0.5rem;

        li.group {
          display: flex;
          flex-direction: column;
          gap: 0.3125rem;

          .label {
            position: sticky;
            top: -0.3125rem;
            z-index: 2;
            padding: 0.5rem 0;
            background-color: rgba($background-secondary, 0.9);
            font-size: 0.75rem;
            font-weight: 500;
            text-transform: uppercase;
          }

          .emoji-list {
            display: flex;
            flex-wrap: wrap;
            gap: 0.125rem;

            li.emoji-container {
              display: flex;
              align-items: center;
              justify-content: center;
              padding: 0.125rem;
              border-radius: 0.1875rem;
              cursor: pointer;
              transition: background-color 0.2s ease;

              &:hover {
                background-color: $background-floating;
              }

              &.active {
                background-color: lighten($background-floating, 5%);
              }

              .emoji {
                display: flex;
                align-items: center;
                justify-content: center;
                width: 1.5rem;
                height: 1.5rem;
              }
            }
          }
        }
      }
    }

    .details {
      display: flex;
      align-items: center;
      height: 2.25rem;
      background: $background-tertiary;
      padding: 0 0.625rem;

      .details-container {
        display: flex;
        align-items: center;
        gap: 0.3125rem;

        span {
          cursor: default;

          &.emoji {
            display: flex;
            align-items: center;
            justify-content: center;
            width: 1.25rem;
            height: 1.25rem;
          }

          &.emoji-id {
            font-size: 1rem;
            font-weight: 500;
          }
        }
      }
    }

    .groups {
      display: flex;
      height: 1.875rem;
      background: $background-floating;
      padding: 0 0.3125rem;

      ul {
        display: flex;
        flex: 1;
        gap: 0.125rem;

        li {
          display: flex;
          flex: 1;
          align-items: center;
          justify-content: center;
          height: 1.6875rem;
          cursor: pointer;
          transition: border-color 0.2s ease;

          &.active {
            border-bottom: 0.1875rem solid $style-primary;
          }

          span {
            display: flex;
            align-items: center;
            justify-content: center;
            width: 1.25rem;
            height: 1.25rem;
            font-size: 1rem;
          }
        }
      }
    }
  }
</style>

<script lang="ts">
  import { directive as onClickaway } from 'vue-clickaway'
  import { Component, Ref, Vue } from 'vue-property-decorator'
  import { get } from '../utils/localstorage'

  @Component({
    name: 'neko-emoji',
    directives: {
      onClickaway,
    },
  })
  export default class extends Vue {
    @Ref('scroll') readonly _scroll!: HTMLElement
    @Ref('search') readonly _search!: HTMLInputElement
    @Ref('groups') readonly _groups!: HTMLElement[]

    waitingForPaint = false
    search = ''
    index = 0
    hovered = ''
    recent: string[] = JSON.parse(get('emoji_recent', '[]'))

    get active() {
      return this.$accessor.emoji.groups[this.index]
    }

    get keywords() {
      return this.$accessor.emoji.keywords
    }

    get groups() {
      return this.$accessor.emoji.groups
    }

    get list() {
      return this.$accessor.emoji.list
    }

    get filtered() {
      const filtered = []
      for (const emoji of this.list) {
        if (
          emoji.includes(this.search) || typeof this.keywords[emoji] !== 'undefined'
            ? this.keywords[emoji].some((keyword) => keyword.includes(this.search))
            : false
        ) {
          filtered.push(emoji)
        }
      }
      return filtered
    }

    scrollTo(event: MouseEvent, index: number) {
      if (!this._groups[index]) {
        return
      }
      this._scroll.scrollTop = index == 0 ? 0 : this._groups[index].offsetTop
    }

    onScroll() {
      if (!this.waitingForPaint) {
        this.waitingForPaint = true
        window.requestAnimationFrame(this.onScrollPaint.bind(this))
      }
    }

    onScrollPaint() {
      this.waitingForPaint = false
      let scrollTop = this._scroll.scrollTop
      let active = 0
      for (const [i] of this.groups.entries()) {
        let component = this._groups[i]
        if (component && component.offsetTop > scrollTop) {
          break
        }
        active = i
      }
      if (this.index !== active) {
        this.index = active
      }
    }

    onMouseExit() {
      this.hovered = ''
    }

    onMouseEnter(event: MouseEvent, emoji: string) {
      this.hovered = emoji
      this._search.placeholder = `:${emoji}:`
    }

    onClick(event: MouseEvent, emoji: string) {
      this.$accessor.emoji.setRecent(emoji)
      this.$emit('picked', emoji)
    }

    onClickAway() {
      this.$emit('done')
    }
  }
</script>
