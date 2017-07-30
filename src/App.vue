
<template>
  <div id="app" v-if="categories.length">
    <section class="js-equal-height" v-if="category.id !== 1 && category.name !== 'home'" :id="category.slug" v-for="category in categories">
      <category :category="category" :posts="posts.links"></category>
    </section>
  </div>
</template>

<script>
import axios from 'axios'
import Category from './components/Category'

export default {
  name: 'app',
  components: {
    Category
  },
  data () {
    return {
      categories: [],
      posts: {
        images: [],
        links: [],
        unfiltered: []
      }
    }
  },
  methods: {
    fetchAll: function () {
      axios.all([this.fetchCategories(), this.fetchPosts()])
        .then(() => {
          this.formatData()
        }).catch(error => {
          console.error('fetchAll ' + error)
        })
    },
    fetchCategories: function () {
      return axios.get('http://50.87.249.59/~blackday/wp-json/wp/v2/categories?per_page=100')
        .then(response => {
          this.categories = response.data
        }).catch(error => {
          console.error('fetchCategories ' + error)
        })
    },
    fetchPosts: function () {
      return axios.get('http://50.87.249.59/~blackday/wp-json/wp/v2/posts?per_page=100')
        .then(response => {
          this.posts.unfiltered = response.data
        }).catch(error => {
          console.error('fetchPosts ' + error)
        })
    },
    filterPosts: function () {
      const imagePosts = []
      const linkPosts = []

      return new Promise(function (resolve, reject) {
        this.posts.unfiltered.forEach(post => {
          if (post.format === 'image') {
            imagePosts.push(post)
          } else if (post.format === 'link') {
            linkPosts.push(post)
          }
        })
        this.posts.images = imagePosts
        this.posts.links = linkPosts
        resolve()
      }.bind(this))
    },
    formatCategories: function () {
      const prioritizedCategories = []
      const imagePosts = this.posts.images

      return new Promise(function (resolve, reject) {
        // set all category priorities, except Uncategorized
        this.categories.forEach(category => {
          if (category.id !== 1) {
            prioritizedCategories.push({
              id: category.id,
              image: getImage(category.id),
              name: category.name,
              priority: parseInt(category.description),
              slug: category.slug
            })
          }
        })
        this.categories = prioritizedCategories
        this.categories.sort(sortByPriority)
        resolve()
      }.bind(this))

      function getImage (id) {
        const numOfPosts = imagePosts.length

        for (let i = 0; i < numOfPosts; i++) {
          if (id === imagePosts[i].categories[0]) {
            // return 'large-medium-file'
            return imagePosts[i].content.rendered.split('"')[19]
          }
        }
        return
      }

      function sortByPriority (a, b) {
        return a.priority - b.priority
      }
    },
    formatData: function () {
      this.filterPosts()
        .then(this.formatCategories())
        .then(this.setSectionsHeight())
        .then(this.setBackgroundImages())
    },
    setBackgroundImages: function () {
      return new Promise(function (resolve, reject) {
        this.categories.forEach(category => {
          if (document.getElementById(category.slug) && category.slug !== 'daylight-approved') {
            setTimeout(function () {
              document.getElementById(category.slug).getElementsByClassName('image')[0].style.backgroundImage = 'url(' + category.image + ')'
            }, 0)
          }
        })
        resolve()
      }.bind(this))
    },
    setSectionsHeight: function () {
      if (document.addEventListener) {
        window.addEventListener('resize', function () {
          setHeight('section')
        })
      }

      setTimeout(function () {
        setHeight('section')
      }, 1000)

      function setHeight (selector) {
        const elements = document.querySelectorAll(selector)
        const elementsLength = elements.length
        let maxHeight = 0

        if (elements && elementsLength) {
          for (let i = 0; i < elementsLength; i++) { // get max height
            elements[i].style.height = '' // reset height attr
            if (elements[i].clientHeight > maxHeight) {
              maxHeight = elements[i].clientHeight
            }
          }

          for (let i = 0; i < elementsLength; i++) { // set max height to all elements
            elements[i].style.height = (maxHeight + 10) + 'px'
          }
        }
      }
    }
  },
  mounted: function () {
    this.fetchAll()
  }
}
</script>

<!--
  red: #fe0000 rgba(254, 0, 0, 1)
-->
<style lang="sass">
@import url('https://fonts.googleapis.com/css?family=Rock+Salt"');

@font-face
  font-family: 'Moonhouse'
  src: url('assets/fonts/nimavisual_moonhouse/moonhouse.ttf')

*
  box-sizing: border-box

body
  background: #111 url('assets/zwartevilt.png')
  color: #fff
  font-family: 'Moonhouse', 'Rock Salt', cursive
  font-size: 1.75vw
  -webkit-font-smoothing: antialiased
  -moz-osx-font-smoothing: grayscale
  letter-spacing: 1px
  text-transform: lowercase

header
  border-bottom: 2px solid #000

h1
  color: #fe0000
  font-size: 10vw
  line-height: 2
  margin: 0
  text-align: center

a
  color: rgba(254, 0, 0, .85)
  cursor: pointer
  text-decoration: none
  &:hover
    color: #fe0000
    text-decoration: underline

footer
  display: flex
  font-size: .65em
  justify-content: space-between
  margin: 2%

#app > section
  border-bottom: 2px solid #000
  height: auto
  &:nth-child(even)
    div.content
      border-left: 1px solid #000
      border-right: 0
      order: 1
    div.image
      border-left: 0
      border-right: 1px solid #000
  a
    display: block
</style>
