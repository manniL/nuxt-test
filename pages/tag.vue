<template>
  <div class="page-saved-search" :class="{ 'can-edit' : canEdit }">
    <top-nav class="m-4 sm:m-8">
      <div v-if="editing" class="m-1 inline-block">
        <a class="paragraph-link mr-3" @click="editing=false">
          Cancel
        </a>
        <a class="btn px-5 py-2 mr-3" @click="save()">Save</a>
      </div>
    </top-nav>
    <section class="header text-center max-w-lg mx-auto mb-4">
      <h1 class="mx-4 text-2xl sm:text-4xl editable" @click="editIfOwner()">
        <template v-if="editing">
          <input
            ref="name"
            v-model="tag.name"
            v-shortkey="['enter']"
            class="text-3xl text-center no-border w-full"
            @shortkey="save()"
          >
        </template>
        <template v-else>
          <i v-if="tag.icon" class="text-gray" :class="tag.icon" />
          Tag: {{ tag.name }}
        </template>
      </h1>
      <template v-if="editing">
        <input
          ref="slug"
          v-model="tag.slug"
          v-shortkey="['enter']"
          class="text-lg text-center no-border w-full"
          placeholder="slug"
          @shortkey="save()"
        >
        <input
          ref="icon"
          v-model="tag.icon"
          v-shortkey="['enter']"
          class="text-lg text-center no-border w-full"
          placeholder="e.g. fas fa-location-arrow"
          @shortkey="save()"
        >
      </template>
    </section>
    <section class="max-w-2xl mb-8 mx-auto">
      <div class="user-cards m-2 mb-4 sm:mb-8 flex flex-wrap justify-center">
        <user-card
          v-for="user in tag.users.slice(0, 24)"
          :key="user.id"
          class="hoverable w-full sm:max-w-xs m-2"
          :user="user"
        />
      </div>
      <div v-if="tag.users.length > 24" class="centered">
        <nuxt-link class="btn px-5 py-2" :to="{name: 'search-query', params: { query: tag.query }}">
          See more
          <i class="fas fa-caret-right ml-2" />
        </nuxt-link>
      </div>
    </section>
    <section class="max-w-lg mb-8 mx-auto p-4 text-center">
      <h2 class="mb-4">
        Want to be added to this list?
      </h2>
      <div v-if="!loggedIn">
        <a class="btn px-5 py-2" href="/auth/linkedin" target="_blank">Sign up for free</a>
      </div>
      <div v-else class="text-xl">
        <p>
          If you want to be added to this list and aren't on it already, just
          <nuxt-link :to="{path: '/' + loggedInUser.username }">
            tag your profile
          </nuxt-link>
          with the tags that this list is associated with.
        </p>
      </div>
    </section>
    <hr class="mt-16 mb-16">
    <keyboard-shortcuts />
    <footer-component />
  </div>
</template>
<script>
import TopNav from '~/components/TopNav.vue'
import UserCard from '~/components/UserCard.vue'
import FooterComponent from '~/components/FooterComponent.vue'
import KeyboardShortcuts from '~/components/KeyboardShortcuts'

export default {
  components: {
    TopNav, UserCard, FooterComponent, KeyboardShortcuts
  },
  data() {
    return {
      tag: { users: [] },
      editing: false
    }
  },
  head() {
    return {
      title: this.tag.name + ' | pros.global'
    }
  },
  computed: {
    loggedIn() {
      return this.$store.state.user && this.$store.state.user.id
    },
    loggedInUser: function () {
      return this.$store.state.user
    },
    canEdit() {
      if (!this.loggedIn) {
        return false
      }

      return !!this.loggedInUser.is_admin
    }
  },
  mounted() {
    this.$axios.get(this.$api('tags/' + this.$route.params.slug)).then((response) => {
      this.tag = response.data
    })
  },
  methods: {
    editIfOwner() {
      if (this.canEdit) {
        this.editing = true
        this.$nextTick(() => {
          this.$refs.name.focus()
        })
      }
    },
    save() {
      this.editing = false

      this.$axios.post(this.$api('tag/' + this.tag.slug), {
        name: this.tag.name,
        slug: this.tag.slug,
        icon: this.tag.icon
      }).then((response) => {
        this.$toasted.show('Saved!')
      })
    }
  }
}
</script>
