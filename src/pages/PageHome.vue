<template>
  <q-page class="relative-position">
    <q-scroll-area class="absolute full-width full-height">
      <div class="q-py-lg q-px-md row items-end q-col-gutter-md">
        <div class="col">
          <q-input
            bottom-slots
            v-model="newQweetContent"
            placeholder="What's happening?"
            counter
            autogrow
            class="new-qweet"
            maxlength="280"
          >
            <template v-slot:before>
              <q-avatar size="xl">
                <img src="https://cdn.quasar.dev/img/avatar5.jpg">
              </q-avatar>
            </template>
          </q-input>
        </div>

        <div class="col col-shrink">
          <q-btn
            unelevated
            rounded
            color="primary"
            label="Qweet"
            no-caps
            :disable="!newQweetContent"
            class="q-mb-lg"
            @click="addNewQweet"
          />
        </div>
      </div>
      <q-separator
        class="divider"
        size="10px"
        color="grey-2"
      />

      <q-list separator>
        <transition-group
          appear
          enter-active-class="animated fadeIn slow"
          leave-active-class="animated fadeOut slow"
        >
          <q-item
            v-for="qweet in qweets"
            :key="qweet.id"
            clsas="qweet q-py-md"
          >
          <q-item-section avatar top>
            <q-avatar size="xl">
              <img src="https://cdn.quasar.dev/img/avatar2.jpg">
            </q-avatar>
          </q-item-section>

          <q-item-section>
            <q-item-label class="text-subtitle1">
              <strong>Danny Connell</strong>
              <span class="text-grey-7">
                @danny_connell
                <br class="lt-md">&bull; {{ qweet.date | relativeDate }}
              </span>
            </q-item-label>
            <q-item-label class="qweet-content text-body1"> {{ qweet.content }}
            </q-item-label>
            <div class="qweet-icons row justify-between q-mt-sm">
              <q-btn
                flat
                round
                size="sm"
                color="grey"
                icon="far fa-comment"
              />
              <q-btn
                flat
                round
                size="sm"
                color="grey"
                icon="fas fa-retweet"
              />
              <q-btn
                @click="toggleLiked(qweet)"
                flat
                round
                size="sm"
                :icon="qweet.liked ? 'fas fa-heart' : 'far fa-heart'"
                :color="qweet.liked ? 'pink': 'grey'"
              />
              <q-btn
                @click="deleteQweet(qweet)"
                flat
                round
                size="sm"
                color="grey"
                icon="fas fa-trash"
              />

            </div>
          </q-item-section>

        </q-item>
        </transition-group>
      </q-list>
    </q-scroll-area>
  </q-page>
</template>

<script>
import db from 'src/boot/firebase'

import { formatDistance } from 'date-fns'
export default {
  name: 'PageHome',
  filters: {
    relativeDate(value) {
      return formatDistance(value, new Date())
    }
  },
  methods: {
    addNewQweet() {
      let newQweet = {
        content: this.newQweetContent,
        date: Date.now(),
        liked: false
      }
      db.collection("qweets").add(newQweet).then(function(docRef) {
        console.log('Document written with ID: ', docRef.id)
      }).catch(function(error) {
        console.log('Error adding document: ', error)
      })
      this.newQweetContent = ''
    },
    deleteQweet(qweet) {
      db.collection('qweets').doc(qweet.id).delete().then(function() {
        console.log('Document successfully deleted!')
      }).catch(function(error) {
        console.log('Error removing document: ', error)
      })
    },
    toggleLiked(qweet) {
      db.collection('qweets').doc(qweet.id).update({
        liked: !qweet.liked
      })
      .then(function () {
        console.log('Document successfully updated!')
      })
      .catch(function(error) {
        console.log('Error updating document: ', error)
      })
    },
  },
  data() {
    return {
      newQweetContent: '',
      qweets: [
        // {
        //   id: 'id1',
        //   content: 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Curabitur sagittis efficitur tortor, in lobortis felis condimentum ac. Morbi rutrum felis quam, in commodo neque faucibus sed. Nam et mollis lectus. Donec fermentum metus vitae neque suscipit, at tempor justo aliquet. Phasellus sed arcu tortor. Maecenas lobortis massa magna. Sed ut semper tortor. Aenean lectus tortor, viverra non dolor quis, ullamcorper imperdiet libero.',
        //   date: 1611653239221,
        //   liked: false
        // },
        // {
        //   id: 'id2',
        //   content: 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Curabitur sagittis efficitur tortor, in lobortis felis condimentum ac. Morbi rutrum felis quam, in commodo neque faucibus sed. Nam et mollis lectus. Donec fermentum metus vitae neque suscipit, at tempor justo aliquet. Phasellus sed arcu tortor. Maecenas lobortis massa magna. Sed ut semper tortor. Aenean lectus tortor, viverra non dolor quis, ullamcorper imperdiet libero.',
        //   date: 1611653238221,
        //   liked: true
        // }
      ]
    }
  },
  mounted() {
    db.collection('qweets').orderBy('date').onSnapshot((snapshot) => {
        snapshot.docChanges().forEach((change) => {
          let qweetChange = change.doc.data()
          qweetChange.id = change.doc.id
          if (change.type === 'added') {
            this.qweets.unshift(qweetChange )
          }
          if (change.type === 'modified') {
            let index = this.qweets.findIndex(qweet => qweet.id === qweetChange.id)
            Object.assign(this.qweets[index], qweetChange)
          }
          if (change.type === 'removed') {
            let index = this.qweets.findIndex(qweet => qweet.id === qweetChange.id)
            this.qweets.splice(index, 1)
          }
        })
      })
  }
}
</script>

<style lang="sass">
.new-qweet textarea
  font-size: 19px
  line-height: 1.4 !important

.divider
  border-top: 1px solid
  border-bottom: 1px solid
  border-color: $grey-4

.qweet:not(:first-child)
  border-top: 1px solid rgb(0, 0, 0, 0.12)

.qweet-content
  white-space: pre-line

.qweet-icons
  margin-left: -5px

</style>
