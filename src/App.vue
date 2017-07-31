
<template>
  <div id="app" v-if="categories.length">
    <section v-if="category.id !== 1 && category.name !== 'home'" :id="category.slug" v-for="category in categories">
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
        linkPosts.sort(sortBySlug)

        this.posts.images = imagePosts
        this.posts.links = linkPosts
        resolve()

        function sortBySlug (a, b) {
          return (a.slug < b.slug) ? -1 : (a.slug > b.slug) ? 1 : 0
        }
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
      const app = document.getElementById('app')
      const preloader = document.getElementById('sk-fading-circle')

      this.filterPosts()
        .then(this.formatCategories())
        .then(this.setSectionsHeight())
        .then(this.setBackgroundImages())

      setTimeout(function () {
        preloader.style.display = 'none'
        app.style.visibility = 'visible'
        app.style.opacity = 1
        document.body.style.overflow = 'auto'
      }, 5000)
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
@import url('https://fonts.googleapis.com/css?family=Rock+Salt');

@font-face
  font-family: 'Amsterdam'
  src: url('assets/fonts/amsterdam.ttf')

*
  box-sizing: border-box

body
  background: #111 url('assets/zwartevilt.png')
  color: #fff
  font-family: 'Rock Salt', 'Amsterdam', cursive
  font-size: 1.75vw
  -webkit-font-smoothing: antialiased
  -moz-osx-font-smoothing: grayscale
  letter-spacing: 1px
  overflow: hidden
  text-transform: lowercase
  @media screen and (min-width: 1440px)
    font-size: 25.2px

header
  border-bottom: 2px solid #000

h1
  color: #fe0000
  font-size: 10vw
  line-height: 2
  margin: 0
  text-align: center
  @media screen and (min-width: 1440px)
    font-size: 144px

a
  color: rgba(254, 0, 0, .85)
  cursor: pointer
  text-decoration: none
  &:hover
    color: #fe0000
    text-decoration: underline

footer
  display: flex
  font-size: .75em
  justify-content: space-between
  margin: 2%

#app-container
  margin: 0 auto
  max-width: 1440px

#app
  opacity: 0
  transition: visibility 0s, opacity 1s ease-in
  visibility: hidden

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

/* ref: http://tobiasahlin.com/spinkit/ */
#sk-fading-circle
  margin: 15vh auto
  width: 75px
  height: 75px
  position: relative
  .sk-circle
    width: 100%
    height: 100%
    position: absolute
    left: 0
    top: 0
  .sk-circle:before
    content: ''
    display: block
    margin: 0 auto
    width: 15%
    height: 15%
    background-color: #fe0000
    border-radius: 100%
    -webkit-animation: sk-circleFadeDelay 1.2s infinite ease-in-out both
            animation: sk-circleFadeDelay 1.2s infinite ease-in-out both
  .sk-circle2
    -webkit-transform: rotate(30deg)
        -ms-transform: rotate(30deg)
            transform: rotate(30deg)
  .sk-circle3
    -webkit-transform: rotate(60deg)
        -ms-transform: rotate(60deg)
            transform: rotate(60deg)
  .sk-circle4
    -webkit-transform: rotate(90deg)
        -ms-transform: rotate(90deg)
            transform: rotate(90deg)
  .sk-circle5
    -webkit-transform: rotate(120deg)
        -ms-transform: rotate(120deg)
            transform: rotate(120deg)
  .sk-circle6
    -webkit-transform: rotate(150deg)
        -ms-transform: rotate(150deg)
            transform: rotate(150deg)
  .sk-circle7
    -webkit-transform: rotate(180deg)
        -ms-transform: rotate(180deg)
            transform: rotate(180deg)
  .sk-circle8
    -webkit-transform: rotate(210deg)
        -ms-transform: rotate(210deg)
            transform: rotate(210deg)
  .sk-circle9
    -webkit-transform: rotate(240deg)
        -ms-transform: rotate(240deg)
            transform: rotate(240deg)
  .sk-circle10
    -webkit-transform: rotate(270deg)
        -ms-transform: rotate(270deg)
            transform: rotate(270deg)
  .sk-circle11
    -webkit-transform: rotate(300deg)
        -ms-transform: rotate(300deg)
            transform: rotate(300deg)
  .sk-circle12
    -webkit-transform: rotate(330deg)
        -ms-transform: rotate(330deg)
            transform: rotate(330deg)
  .sk-circle2:before
    -webkit-animation-delay: -1.1s
            animation-delay: -1.1s
  .sk-circle3:before
    -webkit-animation-delay: -1s
            animation-delay: -1s
  .sk-circle4:before
    -webkit-animation-delay: -0.9s
            animation-delay: -0.9s
  .sk-circle5:before
    -webkit-animation-delay: -0.8s
            animation-delay: -0.8s
  .sk-circle6:before
    -webkit-animation-delay: -0.7s
            animation-delay: -0.7s
  .sk-circle7:before
    -webkit-animation-delay: -0.6s
            animation-delay: -0.6s
  .sk-circle8:before
    -webkit-animation-delay: -0.5s
            animation-delay: -0.5s
  .sk-circle9:before
    -webkit-animation-delay: -0.4s
            animation-delay: -0.4s
  .sk-circle10:before
    -webkit-animation-delay: -0.3s
            animation-delay: -0.3s
  .sk-circle11:before
    -webkit-animation-delay: -0.2s
            animation-delay: -0.2s
  .sk-circle12:before
    -webkit-animation-delay: -0.1s
            animation-delay: -0.1s

@-webkit-keyframes sk-circleFadeDelay
  0%, 39%, 100%
    opacity: 0
  40%
    opacity: 1

@keyframes sk-circleFadeDelay
  0%, 39%, 100%
    opacity: 0
  40%
    opacity: 1
</style>
