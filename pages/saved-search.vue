<template>
  <div class="page-saved-search" :class="{ 'can-edit' : canEdit }">
    <top-nav class="m-4 sm:m-8">
      <div v-if="editing" class="m-1 inline-block">
        <a class="paragraph-link mr-3" @click="editing=false">
          Cancel
        </a>
        <a class="btn px-5 py-2 mr-3" @click="save()">Save</a>
      </div>
      <div v-if="canEdit && !editing">
        <div class="mr-6" @click="editing=1">
          <i class="material-icons font-120 cursor-pointer animated">edit</i>
        </div>
      </div>
    </top-nav>
    <section class="header text-center max-w-lg mx-auto mb-4">
      <h1 class="mx-4 text-2xl sm:text-4xl editable" @click="editIfOwner()">
        <template v-if="editing">
          <input
            ref="name"
            v-model="savedSearch.name"
            class="text-3xl text-center w-full text bg-transparent-input mb-2"
          >
        </template>
        <template v-else>
          <i v-if="savedSearch.icon" class="text-gray" :class="savedSearch.icon" />
          {{ savedSearch.description ? savedSearch.description : savedSearch.name }}
          <i v-if="canEdit" class="edit-icon fas fa-pencil-alt" />
        </template>
      </h1>
      <template v-if="editing">
        <input
          ref="description"
          v-model="savedSearch.description"
          class="text-lg text-center no-border block w-full mx-auto mb-2 text bg-transparent-input"
          placeholder="Longer description (also serves as meta description tag)"
        >
        <input
          ref="query"
          v-model="savedSearch.query"
          class="text-lg text-center no-border block w-64 text mx-auto mb-2 bg-transparent-input"
        >
        <input
          ref="slug"
          v-model="savedSearch.slug"
          class="text-lg text-center no-border w-64 block mx-auto mb-2 text bg-transparent-input"
          placeholder="slug"
        >
        <input
          ref="featured_order"
          v-model="savedSearch.featured_order"
          class="text-lg text-center no-border w-24 text bg-transparent-input block mx-auto mb-2"
          placeholder="e.g. 10"
        >
        <input
          ref="icon"
          v-model="savedSearch.icon"
          class="text-lg text-center no-border text bg-transparent-input block mx-auto mb-2 w-64"
          placeholder="e.g. fas fa-location-arrow"
        >
      </template>
    </section>
    <section v-if="savedSearch.users && savedSearch.users.length > 0" class="max-w-2xl mb-8 mx-auto">
      <div class="user-cards m-2 mb-4 sm:mb-8 flex flex-wrap justify-center">
        <user-card
          v-for="user in savedSearch.users.slice(0, 6)"
          :key="user.id"
          class="hoverable w-full sm:max-w-xs m-2"
          :user="user"
        />
      </div>
      <div v-if="savedSearch.users.length > 6" class="centered">
        <nuxt-link class="btn px-5 py-2 bold" :to="{ path: '/search?q=' + savedSearch.query }">
          See more
          <i class="material-icons align-middle" style="margin-right: -7px;">keyboard_arrow_right</i>
        </nuxt-link>
      </div>
    </section>

    <hr v-if="relatedSavedSearches.length" class="mt-16 mb-16">
    <section v-if="relatedSavedSearches.length" class="max-w-3xl mb-8 mx-auto">
      <div class="saved-searches m-2 mb-4 sm:mb-8 flex flex-wrap justify-center">
        <saved-search-card
          v-for="_savedSearch in relatedSavedSearches"
          :key="_savedSearch.id"
          class="mb-12 m-4"
          :saved-search="_savedSearch"
        />
      </div>
    </section>

    <div v-if="editing">
      <input
        id="relatedSavedSearchSlug"
        ref="relatedSavedSearchSlug"
        placeholder="Add related saved search (by slug)"
        class="w-128 mx-auto my-2 p-2 block text bg-transparent-input"
      >
      <input
        id="relatedToRemove"
        ref="relatedToRemove"
        placeholder="Remove related saved search (by slug)"
        class="w-128 mx-auto my-2 p-2 block text bg-transparent-input"
      >
    </div>

    <hr class="mt-16 mb-16">
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
          <nuxt-link :to="{ path: '/' + loggedInUser.username }">
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
import SavedSearchCard from '~/components/SavedSearchCard.vue'
import FooterComponent from '~/components/FooterComponent.vue'
import KeyboardShortcuts from '~/components/KeyboardShortcuts'

export default {
  components: {
    TopNav, UserCard, SavedSearchCard, FooterComponent, KeyboardShortcuts
  },
  data() {
    return {
      savedSearch: { users: [] },
      relatedSavedSearches: [],
      editing: false
    }
  },
  computed: {
    cardImage() {
      return 'https://image.thum.io/get/viewportWidth/900/viewportHeight/450/width/900/noanimate/' +
                    '?url=' + encodeURIComponent(process.env.CARD_BASE_URL + 's/' + this.savedSearch.slug + '/twitter-card?v4')
    },
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

      if (this.loggedInUser.is_admin) {
        return true
      }

      return (this.loggedInUser.id === this.savedSearch.user_id)
    }
  },
  async asyncData({ $axios, params }) {
    if (process.server) {
      const { data } = await $axios.get('saved-searches/' + params.slug)
      return { savedSearch: data }
    }
  },
  head() {
    return {
      title: this.savedSearch.name + ' | pros.global',
      meta: this.$metaTags(this.savedSearch.name, this.savedSearch.description, this.cardImage)
    }
  },
  mounted() {
    window.addEventListener('keyup', this.hotkeys)

    this.$axios.get(this.$api('saved-searches/' + this.$route.params.slug)).then((response) => {
      this.savedSearch = response.data
    })
    this.$axios.get(this.$api('saved-searches/' + this.$route.params.slug + '/related?with_users=1'))
      .then((response) => {
        this.relatedSavedSearches = response.data
      })
  },
  methods: {
    hotkeys(e) {
      if (e.key === 'Escape') {
        this.editing = false
      }

      if (document.activeElement.tagName === 'BODY') {
        if (e.key === 'e') {
          this.editIfOwner()
        }
      }

      if (e.key === 'Enter') {
        // eslint-disable-next-line no-console
        console.log(document.activeElement.id)
        if (document.activeElement.id === 'relatedSavedSearchSlug') {
          this.addRelatedSavedSearch()
        }

        if (document.activeElement.id === 'relatedToRemove') {
          this.removeRelatedSavedSearch()
        }
      }
    },
    editIfOwner() {
      if (this.canEdit) {
        this.editing = true
        this.$nextTick(() => {
          if (this.$refs.name) {
            this.$refs.name.focus()
          }
        })
      }
    },
    save() {
      this.editing = false

      this.$axios.post(this.$api('saved-searches/' + this.savedSearch.id), {
        name: this.savedSearch.name,
        description: this.savedSearch.description,
        query: this.savedSearch.query,
        featured_order: this.savedSearch.featured_order,
        icon: this.savedSearch.icon,
        slug: this.savedSearch.slug
      }).then((response) => {
        this.savedSearch = response.data
        this.$toasted.show('Saved!')

        if (this.savedSearch.slug) {
          this.$router.push({
            path: '/s/' + this.savedSearch.slug
          })
        }
      })
    },
    addRelatedSavedSearch() {
      this.$axios.post(this.$api('saved-searches/' + this.savedSearch.id + '/related'), {
        slug: this.$refs.relatedSavedSearchSlug.value
      }).then((response) => {
        // this.relatedSavedSearches = response.data;
        this.$toast.show('Saved new related saved search!')
      })
    },
    removeRelatedSavedSearch() {
      this.$axios.post(this.$api('saved-searches/' + this.savedSearch.id + '/related/remove'), {
        slug: this.$refs.relatedToRemove.value
      }).then((response) => {
        // this.relatedSavedSearches = response.data;
        this.$toast.show('Removed related saved search!')
      })
    }
  }
}
</script>
