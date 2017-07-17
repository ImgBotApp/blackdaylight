<template>
  <div>
    <h2 v-on:click="toggle(clicked)">
      {{ category.name }}
    </h2>
    <ul v-show="clicked">
       <li v-for="post in posts" v-if="category.id === post.categories[0].id" v-html="post.content.rendered"></li>
    </ul>
  </div>
</template>

<script>

export default {
  name: 'category',
  data () {
    return {
      clicked: false
    }
  },
  props: ['category', 'posts'],
  methods: {
    toggle: function (clicked) {
      this.clicked = !clicked
    }
  }
}
</script>

<!-- "scoped" attribute limits CSS to this component only -->
<style scoped lang="sass">
h2
  font-size: 1.5em
  font-weight: bold
  margin: 1%
</style>
