<template>
  <ul>
    <li><i @click.stop.prevent="about" class="fas fa-question-circle" /></li>
    <li>
      <i
        class="fas fa-shield-alt"
        v-tooltip="{
          content: $t('admin_loggedin'),
          placement: 'right',
          offset: 5,
          boundariesElement: 'body',
        }"
        v-if="admin"
      />
    </li>
    <li>
      <select v-model="$i18n.locale">
        <option v-for="(lang, i) in langs" :key="`Lang${i}`" :value="lang">
          {{ lang }}
        </option>
      </select>
    </li>
  </ul>
</template>

<style lang="scss" scoped>
  ul {
    display: flex;
    align-items: center;
    gap: 1rem;

    li {
      display: flex;
      align-items: center;

      i {
        font-size: 1.5rem;
        cursor: pointer;
        transition: color 0.2s ease;

        &:hover {
          color: $text-normal;
        }
      }
    }
  }

  select {
    appearance: none;
    background-color: $background-tertiary;
    border: 1px solid $background-primary;
    color: white;
    cursor: pointer;
    border-radius: 0.25rem;
    height: 2rem;
    padding: 0 0.5rem;
    display: flex;
    align-items: center;
    transition: all 0.2s ease;

    option {
      font-weight: normal;
      color: $text-normal;
      background-color: $background-tertiary;
    }

    &:hover {
      border-color: lighten($background-primary, 10%);
      background-color: lighten($background-tertiary, 5%);
    }

    &:focus {
      outline: none;
      border-color: $text-link;
    }
  }
</style>

<script lang="ts">
  import { Component, Vue } from 'vue-property-decorator'
  import { messages } from '~/locale'

  @Component({ name: 'neko-menu' })
  export default class extends Vue {
    get admin() {
      return this.$accessor.user.admin
    }

    get langs() {
      return Object.keys(messages)
    }

    about() {
      this.$accessor.client.toggleAbout()
    }

    mounted() {
      const default_lang = new URL(location.href).searchParams.get('lang')
      if (default_lang && this.langs.includes(default_lang)) {
        this.$i18n.locale = default_lang
      }
    }
  }
</script>
