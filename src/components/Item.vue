<template>
  <li class="item">
    <div class="user-box">
      <div class="avatar" :style="'background-image: url('+user.photoURL+')'">
      </div>
      <p class="user-name">{{user.name}}</p>
    </div>
    <div v-if="editing" class="editor">
      <textarea v-model="post.content" placeholder="edit post" @keypress.enter="updatePost">
      </textarea>
      <p class="message">Press Enter to Post</p>
    </div>
    <div v-else class="content" v-html="post.content"></div>
    <button v-if="currentUser && currentUser.uid == user.id" @click="showBtns = !showBtns">
      <fa icon="ellipsis-v" />
    </button>
    <div v-if="showBtns" class="controls">
      <li @click="editing = !editing">edit</li>
      <li @click="deletePost" style="color: red">
        delete
      </li>
    </div>
  </li>
</template>

<script>
import { db } from '../main'
import { auth } from '../main'

export default {
  props: ['id', 'uid'],
  data () {
    return {
      post: {},
      user: {},
      currentUser: {},
      showBtns: false,
      editing: false
    }
  },
  firestore () {
    return {
      post: db.collection('posts').doc(this.$props.id),
      user: db.collection('users').doc(this.$props.uid)
    }
  },
  created () {
    auth.onAuthStateChanged(user => {
      this.currentUser = user
    })
  },
  methods: {
    deletePost () {
      if (window.confirm('Are You Sure to Delete This Post?')) {
        db.collection('posts').doc(this.$props.id).delete()
      }
    },
    updatePost () {
      const date = new Date()
      db.collection('posts').doc(this.post.id).set({
        'content': this.post.content,
        'date': date
      }, { merge: true })
      .then(
        this.editing = false
      )
    }
  },
}
</script>

<style lang="stylus">
.item
  list-style none
  border-top 1px solid #eee
  padding 5px 15px
  display flex
  flex-wrap no-wrap
  justify-content flex-start
  position relative
  .editor
    position relative
    width 100%
    textarea
      background transparent
      resize none
      height 80px
      width 96%
      border none
      padding 10px 2%
      font-size .9rem
      font-weight lighter
      &:focus
        outline none
    .message
      opacity 0  
      position absolute
      bottom 5px
      right 10px
      transition .2s
      font-size .8rem
    &:hover
      .message
        opacity 1
  &:first-child
    border none
  &:hover
    background rgba(0,0,0,.02)
  .user-box
    margin 10px 10px 10px 0
    .avatar
      height 50px
      width 50px
      border-radius 50%
      border 1px solid #eee
      background-size cover
    .user-name
      margin 5px 0 0 0
      text-align center
      font-size .7rem
      line-height .7rem
      width 50px
  .content
    padding 10px
  button
    position absolute
    top 5px
    right 0
    background transparent
    color #555
    font-size .9rem
    opacity 0
    transition .2s
  .controls
    background white
    position absolute
    top 5px
    right 35px
    box-shadow 0 0 5px rgba(0,0,0,.05)
    border-radius 3px
    opacity 0
    li
      padding 5px 20px
      border-top 1px solid #eee
      cursor pointer
      &:first-child
        border none
  &:first-child
    border none
  &:hover
    background rgba(0,0,0,.02)
    .content
    button
      opacity 1
    .controls
      opacity 1
</style>