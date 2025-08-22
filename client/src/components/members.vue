<template>
  <div class="members">
    <div class="members-container">
      <ul class="members-list">
        <li v-if="member">
          <div :class="[{ host: member.id === host }, 'self', 'member']">
            <neko-avatar class="avatar" :seed="member.displayname" :size="50" />
          </div>
        </li>
        <template v-for="(member, index) in members">
          <li
            v-if="member.id !== id && member.connected"
            :key="index"
            v-tooltip="{ content: member.displayname, placement: 'bottom', offset: -15, boundariesElement: 'body' }"
          >
            <div
              :class="[{ host: member.id === host, admin: member.admin }, 'member']"
              @contextmenu.stop.prevent="onContext($event, { member })"
            >
              <neko-avatar class="avatar" :seed="member.displayname" :size="50" />
            </div>
          </li>
        </template>
      </ul>
    </div>
    <neko-context ref="context" />
  </div>
</template>

<style lang="scss" scoped>
  .members {
    display: flex;
    flex: 1;
    overflow-x: auto;
    overflow-y: hidden;
    padding: 0 0 0.875rem 0;
    scrollbar-width: thin;
    scrollbar-color: $background-secondary $background-tertiary;
    min-height: 3.75rem;

    &::-webkit-scrollbar {
      height: 0.25rem;
    }

    &::-webkit-scrollbar-track {
      background-color: $background-tertiary;
    }

    &::-webkit-scrollbar-thumb {
      background-color: $background-secondary;
      border-radius: 0.25rem;
    }

    &::-webkit-scrollbar-thumb:hover {
      background-color: $background-primary;
    }

    .members-container {
      display: flex;
      align-items: center;
      padding: 0 1.25rem;
      margin: 0 auto;

      .members-list {
        display: flex;
        align-items: center;
        gap: 0.625rem;

        li {
          display: flex;
          align-items: center;

          .member {
            position: relative;
            display: flex;
            align-items: center;
            justify-content: center;
            width: 3.125rem;
            height: 3.125rem;
            margin: 0.625rem 0.3125rem 0;
            transition: transform 0.2s ease;

            &:hover {
              transform: scale(1.1);
            }

            &.self {
              &::before {
                font-family: 'Font Awesome 6 Free';
                font-weight: 900;
                content: '\f2bd';
                background: $background-floating;
                color: $style-primary;
                position: absolute;
                display: flex;
                align-items: center;
                justify-content: center;
                width: 1rem;
                height: 1rem;
                font-size: 1.25rem;
                top: -0.125rem;
                right: -0.125rem;
                border-radius: 50%;
              }
            }

            &.admin {
              &::before {
                font-family: 'Font Awesome 6 Free';
                font-weight: 900;
                content: '\f3ed';
                color: $style-primary;
                background: transparent;
                position: absolute;
                display: flex;
                align-items: center;
                justify-content: center;
                width: 0.875rem;
                height: 0.875rem;
                font-size: 0.875rem;
                top: -0.125rem;
                right: -0.5rem;
              }
            }

            &.host::after {
              font-family: 'Font Awesome 6 Free';
              font-weight: 900;
              content: '\f521';
              background: $style-primary;
              color: $background-floating;
              position: absolute;
              display: flex;
              align-items: center;
              justify-content: center;
              width: 1.25rem;
              height: 1.25rem;
              font-size: 0.625rem;
              bottom: -0.625rem;
              left: -1.125rem;
              border-radius: 50%;
            }

            .avatar {
              width: 100%;
              height: 100%;
              border-radius: 50%;
              overflow: hidden;
            }
          }

          &:nth-child(2) {
            position: relative;
            margin-left: 1.25rem;

            &::before {
              content: '';
              position: absolute;
              left: -0.75rem;
              height: 2.8125rem;
              width: 0.125rem;
              background: $background-secondary;
            }
          }
        }
      }
    }
  }
</style>

<script lang="ts">
  import { Component, Ref, Vue } from 'vue-property-decorator'

  import Avatar from './avatar.vue'
  import Content from './context.vue'

  @Component({
    name: 'neko-members',
    components: {
      'neko-context': Content,
      'neko-avatar': Avatar,
    },
  })
  export default class extends Vue {
    @Ref('context') readonly _context!: any

    get id() {
      return this.$accessor.user.id
    }

    get host() {
      return this.$accessor.remote.id
    }

    get member() {
      return this.$accessor.user.member
    }

    get members() {
      return this.$accessor.user.members
    }

    onContext(event: MouseEvent, data: any) {
      this._context.open(event, data)
    }
  }
</script>
