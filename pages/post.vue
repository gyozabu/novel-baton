<template>
  <v-card class="mx-auto py-6">
    <div v-if="isAuthenticated">
      <v-form ref="form" v-model="valid">
        <v-card-title class="headline">新規投稿</v-card-title>
        <v-card-text>
          <p>作成日：2019/11/02</p>
          <p class="title text--primary">タイトル</p>
          <div class="text--primary">
            <v-text-field
              v-model="title"
              :rules="titleRules"
              label="20文字以内で入力してね"
              required
              filled
            />
          </div>
          <p class="title text--primary mt-6">文章</p>
          <div class="text--primary">
            <v-textarea
              v-model="content"
              :rules="postRules"
              label="140文字以内で入力してね"
              rows="10"
              filled
            />
          </div>
        </v-card-text>
        <v-card-actions class="justify-center">
          <v-btn raised large color="grey lighten-1 accent-4" @click="goback"
            >戻る</v-btn
          >
          <v-btn
            :disabled="!valid"
            raised
            large
            color="primary accent-4"
            @click="post"
            >作成する</v-btn
          >
        </v-card-actions>
      </v-form>
    </div>
    <div v-else>投稿するにはログインが必要です。</div>
  </v-card>
</template>
<script>
import { mapState, mapGetters, mapActions } from 'vuex'
import firebase from '~/plugins/firebase'
export default {
  data: () => ({
    valid: true,
    name: '',
    titleRules: [
      (v) => !!v || 'タイトルは必須です',
      (v) => (v && v.length <= 20) || 'タイトルは20文字以内で入力してください'
    ],
    postRules: [
      (v) => !!v || '文章は必須です',
      (v) => (v && v.length <= 140) || '文章は140文字以内で入力してください'
    ],
    title: '',
    content: ''
  }),
  computed: {
    ...mapState(['user']),
    ...mapGetters(['isAuthenticated'])
  },
  methods: {
    ...mapActions(['setPost']),
    goback() {
      this.$router.go(-1)
    },
    post() {
      const db = firebase.firestore()
      const title = this.title
      const content = this.content
      const user = this.$store.state.user
      db.collection('novel')
        .add({
          title,
          headline: content,
          createdAt: firebase.firestore.FieldValue.serverTimestamp(),
          updatedAt: firebase.firestore.FieldValue.serverTimestamp(),
          userUid: user.uid,
          postUsers: firebase.firestore.FieldValue.arrayUnion(user.uid),
          favoriteCount: 0
        })
        .then(function(docRef) {
          docRef.collection('content').add({
            body: content,
            order: 1,
            createdAt: firebase.firestore.FieldValue.serverTimestamp(),
            updatedAt: firebase.firestore.FieldValue.serverTimestamp(),
            userUid: user.uid
          })
          db.collection('user')
            .doc(user.uid)
            .update({
              posts: firebase.firestore.FieldValue.arrayUnion(docRef.id)
            })
        })
      this.setPost(true)
      this.$router.go(-1)
    }
  }
}
</script>
