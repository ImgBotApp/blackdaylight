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
  height: 100%
  @media screen and (min-width: 768px)
    display: flex

div.content
  border-right: 1px solid #000
  height: auto
  min-height: 32vh
  text-align: center
  padding-bottom: 10px
  width: 100%
  @media screen and (min-width: 768px)
    min-height: auto
    width: 50%
  #daylight-approved &
    border-right: 0
    width: 100%

div.image
  background-size: 100% 100%
  border-left: 1px solid #000
  height: 50vh
  width: 100%
  @media screen and (min-width: 768px)
    height: auto
    width: 50%

h2
  font-size: 8vw
  line-height: 2
  @media screen and (min-width: 768px)
    font-size: 7vw
    line-height: 1.5
  @media screen and (min-width: 1440px)
    font-size: 100.8px
ul
  align-items: center
  display: flex
  flex-direction: column
  @media screen and (min-width: 768px)
    flex-direction: row
    flex-wrap: wrap
    justify-content: center
li
  line-height: 2
  width: 100%
  @media screen and (min-width: 768px)
    width: 50%
</style>
