<template>
  <div @click="toggle(clicked); animate()">
    <h2><span>{{ category.name }}</span></h2>
    <ul v-show="clicked">
       <li v-for="post in posts" v-if="category.id === post.categories[0]" v-html="post.content.rendered"></li>
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
    },
    animate: function () {
      const categoryParent = this.$options._parentElm
      const allSections = document.getElementsByTagName('section')

      for (let i = 0; i < allSections.length; i++) {
        const sectionDiv = allSections[i].getElementsByTagName('ul')[0]
        sectionDiv.style.display = 'none'

        allSections[i].style.width = null
        allSections[i].style.order = null

        if (this.clicked) {
          categoryParent.style.width = '100%'
          categoryParent.style.order = '-1'
        }
      }
    }
  }
}
</script>

<!-- "scoped" attribute limits CSS to this component only -->
<style scoped lang="sass">
div
  cursor: pointer
  display: flex
  flex-flow: column nowrap
  height: 100%;
  justify-content: center;
h2
  font-size: 7vw
  line-height: 1.25

  span
    color: #fff
    padding: 0 5%
    text-shadow: 0 0 10px rgba(0, 0, 0, 1)
</style>
