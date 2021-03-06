<template>
  <div class="page page-profile" :class="{ 'can-edit' : canEdit }">
    <top-nav class="m-4 sm:m-8" :user="user">
      <div v-if="editing" class="edit-profile-wrapper m-1 inline-block">
        <div class="inline-block mr-3">
          <a class="paragraph-link mr-3" @click="cancelEditing()">
            Cancel
          </a>
          <a class="btn px-5 py-2" @click="save">Save</a>
        </div>
      </div>
      <div v-if="canEdit && !editing">
        <div class="mr-6" @click="editing=1">
          <i class="material-icons font-120 cursor-pointer animated">edit</i>
        </div>
      </div>
    </top-nav>
    <section class="header max-w-lg mx-auto text-center">
      <div class="m-4">
        <div class="avatar inline-block mb-1 relative">
          <img :src="user.avatar_path" class="w-16 sm:w-32 h-16 sm:h-32 rounded-full">
          <!--<i v-if="this.isPresent(user)" class="absolute is-present fas fa-circle"></i>-->
        </div>
        <input
          v-if="editing"
          ref="avatar_path"
          v-model="user.avatar_path"
          class="p-2 mb-2 block mx-auto w-128 text bg-transparent-input"
          placeholder="e.g. path to avatar"
        >
        <input
          v-if="editing"
          ref="name"
          v-model="user.name"
          class="p-2 mb-2  block mx-auto w-128 bg-transparent-input text"
          placeholder="e.g. Jane Smith"
        >
        <input
          v-if="editing"
          ref="headline"
          v-model="user.headline"
          class="p-2 mb-2  block mx-auto w-128 bg-transparent-input text"
          placeholder="e.g. I am a person that does certain things!"
        >
        <input
          v-if="editing"
          ref="twitter_username"
          v-model="user.twitter_username"
          class="p-2 mb-2  block mx-auto w-full bg-transparent-input text"
          placeholder="e.g. username"
        >
        <h1 v-if="!editing" class="text-xl sm:text-4xl animated">
          {{ user.headline }}
        </h1>
      </div>
    </section>
    <section class="mx-auto max-w-md text-center">
      <div class="m-4">
        <profile-tags :user="user" :editing="editing" />
      </div>
    </section>
    <div v-if="user.about || editing" class="section mx-auto max-w-md text-md">
      <div class="card m-4">
        <div class="card--inner text-left p-4">
          <div v-if="editing" class="editable-about">
            <textarea ref="about" v-model="user.about" class="font-90 width-100" />
          </div>
          <div v-else @click="editIfOwner()" v-html="markdown(user.about)" />
        </div>
      </div>
    </div>
    <section v-if="hasUpvotes" class="endorsements mx-auto p-4 max-w-sm text-sm leading-tight">
      <div v-for="upvote in user.upvotes" :key="upvote.id" class="card hoverable endorsement-card mb-4">
        <div class="card--inner p-4 flex">
          <div class="avatar centered text-center -ml-3">
            <nuxt-link :to="{ path: '/' + upvote.author_username }">
              <img :src="upvote.author_avatar" class="w-8 h-8 rounded-full">
            </nuxt-link>
          </div>
          <div class="endorsement-message flex-4 sm:flex-6">
            <div>
              <div v-if="upvote.message" class="mb-2" v-html="markdown(upvote.message)" />
              <div v-else class="mb-2">
                {{ upvote.author_firstname }} upvoted
              </div>
              <div class="inline-tag">
                {{ upvote.tag_name }}
              </div>
              <nuxt-link
                v-if="loggedInUser.id === upvote.author_id"
                :to="{ path: '/upvotes/' + upvote.id + '?editing=1' }"
              >
                <i class="edit-upvote material-icons align-middle">edit</i>
              </nuxt-link>
              <div class="inline text-gray-light">
                <nuxt-link
                  class="naked-link text-xs ml-1"
                  :to="{ path: 'upvotes/' + upvote.id }"
                >
                  {{ upvote.created_at | moment("subtract", "6 hours") | moment('from') }}
                </nuxt-link>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>
    <section class="mt-16 text-center text-4xl text-gray-light">
      <a class="naked-link mr-3" target="_blank" :href="linkedInUrl">
        <i class="fab fa-linkedin" />
      </a>
      <a
        v-if="user.twitter_username"
        class="naked-link"
        target="_blank"
        :href="'https://twitter.com/' + user.twitter_username"
      >
        <i class="fab fa-twitter" />
      </a>
    </section>

    <!--<chat-wrapper :user="user"></chat-wrapper>-->
    <hr class="mt-16 mb-16">
    <footer-component :user="user" />
    <keyboard-shortcuts />
  </div>
</template>

<script>
import TopNav from '~/components/TopNav.vue'
import FooterComponent from '~/components/FooterComponent.vue'
import KeyboardShortcuts from '~/components/KeyboardShortcuts'
import ProfileTags from '~/components/ProfileTags'

import showdown from 'showdown'

export default {
  components: {
    TopNav, FooterComponent, KeyboardShortcuts, ProfileTags
  },
  data() {
    return {
      user: {},
      editing: false,
      messages: []
    }
  },
  computed: {
    cardImage() {
      return 'https://image.thum.io/get/viewportWidth/900/viewportHeight/450/width/900/noanimate/' +
                    '?url=' + encodeURIComponent(process.env.CARD_BASE_URL + '/' + this.user.username + '/twitter-card?v5')
    },

    hasUpvotes: function () {
      return this.user.upvotes && this.user.upvotes.length
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

      return (this.loggedInUser.id === this.user.id)
    },
    unreadNotificationCount() {
      return this.$store.state.unreadNotificationCount
    },
    linkedInUrl() {
      return 'https://www.linkedin.com/search/results/all/?keywords=' + this.user.name
    }
  },
  async asyncData({ $axios, params }) {
    if (process.server) {
      const { data } = await $axios.get('users/' + params.username)
      return { user: data }
    }
  },
  head() {
    return {
      title: this.user.name + ' | pros.global',
      meta: this.$metaTags(this.user.name, this.user.headline, this.cardImage)
    }
  },
  mounted() {
    window.addEventListener('keyup', this.hotkeys)

    this.$axios.get(this.$api('users/' + this.$route.params.username)).then((response) => {
      this.user = response.data
    })

    this.$root.$on('upvote-added', (upvote, allUpvotes) => {
      // eslint-disable-next-line no-console
      console.log('added')
      this.user.upvotes = allUpvotes
    })
    this.$root.$on('upvote-removed', (upvote, allUpvotes) => {
      this.user.upvotes = allUpvotes
    })
  },
  methods: {
    editIfOwner() {
      if (this.canEdit) {
        this.editing = true
        this.$nextTick(() => {
          if (this.$refs.headline) {
            this.$refs.headline.focus()
          }
        })
      }
    },
    cancelEditing() {
      this.editing = false
    },
    save() {
      this.editing = false
      this.user.about = this.$refs.about.value
      // Don't need to set avatar_path, headline or name because of v-model

      this.$toast.show('Saved profile!', { duration: 5000, position: 'bottom-right' })

      this.$axios.post(this.$api('users/' + this.user.username), {
        'data': this.user
      }, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } }).then((response) => {
        // eslint-disable-next-line no-console
        console.log(response)
      }
      )
    },
    markdown: function (content) {
      const converter = new showdown.Converter()
      return converter.makeHtml(content)
    },
    hotkeys(e) {
      if (e.key === 'Escape') {
        this.editing = false
      }

      if (document.activeElement.tagName === 'BODY') {
        if (e.key === 'i') {
          window.location = '/admin/impersonate/' + this.user.username
        }
        if (e.key === 'e') {
          this.editIfOwner()
        }
      }
    }

    // isPresent(user) {
    //     let ids = [];
    //     let presentUsers = this.presentUsers;
    //     for (let i in presentUsers) {
    //         ids.push(presentUsers[i].id);
    //     }
    //     return ids.includes(user.id);
    // },
  }
}
</script>
