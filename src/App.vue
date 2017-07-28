
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
        .then(this.setBackgroundImages())
    },
    setBackgroundImages: function () {
      return new Promise(function (resolve, reject) {
        this.categories.forEach(category => {
          if (document.getElementById(category.slug)) {
            setTimeout(function () {
              document.getElementById(category.slug).style.backgroundImage = 'url(' + category.image + ')'
            }, 0)
          }
        })
        resolve()
      }.bind(this))
    }
  },
  mounted: function () {
    this.fetchAll()
  }
}
</script>

<style lang="sass">
body
  background-color: #fff
  font-family: 'Rock Salt', cursive
  -webkit-font-smoothing: antialiased
  -moz-osx-font-smoothing: grayscale

h1
  color: #2c3e50
  font-size: 10vw
  line-height: 1.5
  margin: 0
  text-align: center

a
  color: #fff

footer
  font-size: .85em
  margin: 2%
  text-align: right

  a
    color: #000;

#app
  align-content: space-around
  color: #fff
  display: flex
  flex-flow: row wrap
  justify-content: space-around
  text-align: center

#app > section
  background-size: 100% 100%
  border: 5px solid #fff
  box-sizing: border-box
  flex-grow: 1
  height: 40vh
  width: 50%
</style>
