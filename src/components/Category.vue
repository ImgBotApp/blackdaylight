<template>
  <div class="container">
    <div class="content">
      <h2>{{ category.name }}</h2>
      <ul>
         <li v-for="post in posts" v-if="category.id === post.categories[0]" v-html="post.content.rendered"></li>
      </ul>
    </div>
    <div class="image" v-if="category.slug !== 'daylight-approved'"></div>
  </div>
</template>

<script>

export default {
  name: 'category',
  props: ['category', 'posts'],
  methods: {}
}
</script>

<!-- "scoped" attribute limits CSS to this component only -->
<style scoped lang="sass">
div.container
  display: flex
  height: 100%

div.content
  border-right: 1px solid #be0000
  border-right: 1px solid rgba(254, 0, 0, .75)
  text-align: center
  padding-bottom: 10px
  width: 50%
  #daylight-approved &
    border-right: 0
    width: 100%

div.image
  background-size: 100% 100%
  border-left: 1px solid #be0000
  border-left: 1px solid rgba(254, 0, 0, .75)
  width: 50%

h2
  font-size: 7vw
  line-height: 1.25

ul
  display: flex
  flex-wrap: wrap
  justify-content: center

li
  line-height: 2
  width: 50%
</style>
