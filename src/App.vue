
<template>
  <div id="app" v-if="categories.length">
    <h1>blackdaylight</h1>
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
    formatData: function () {
      this.setPostCategoryNames()
        .then(this.filterPosts())
        .then(this.setBackgroundImages())
        // TODO: fix
        // .then(this.sortCategories())
    },
    setBackgroundImages: function () {
      return new Promise(function (resolve, reject) {
        this.posts.images.forEach(post => {
          // find data-medium-file w/in post.content.rendered
          const imgSrc = post.content.rendered.split('"')[17]

          // if name contains a space, replace it with a hyphen
          var id = post.categories[0].name.replace(/\s+/g, '-')

          document.getElementById(id).style.backgroundImage = 'url(' + imgSrc + ')'
        })
        resolve()
      }.bind(this))
    },
    setPostCategoryNames: function () {
      const categories = this.categories

      return new Promise(function (resolve, reject) {
        this.posts.unfiltered.forEach(post => {
          // add category name to post data, except Uncategorized
          post.categories.forEach((category, i) => {
            // TODO: if category name has already been found, don't call getCategoryById
            if (category !== 1) {
              post.categories[i] = {
                id: category,
                name: getCategoryById(category)
              }
            }
          })
        })
        resolve()
      }.bind(this))

      function getCategoryById (id) {
        const numCategories = categories.length

        // return category name, if id exists
        for (let i = 0; i < numCategories; i++) {
          if (categories[i].id === id) {
            return categories[i].name
          }
        }

        console.error('Category ID (' + id + ') is not available.')
        return
      }
    },
    sortCategories: function () {
      const prioritizedCategories = []

      return new Promise(function (resolve, reject) {
        // set all category priorities, except Uncategorized
        this.categories.forEach(category => {
          if (category.id !== 1) {
            prioritizedCategories.push({
              id: category.id,
              name: category.name,
              priority: category.description
            })
          }
        })
        this.categories = prioritizedCategories
        this.categories.sort(compareCategoryPriorities)

        resolve()
      }.bind(this))

      function compareCategoryPriorities (a, b) {
        return a.priority - b.priority
      }
    }
  },
  mounted: function () {
    this.fetchAll()
  }
}
</script>

<style lang="sass">
#app
  font-family: 'Avenir', Helvetica, Arial, sans-serif
  -webkit-font-smoothing: antialiased
  -moz-osx-font-smoothing: grayscale
  text-align: center
  color: #fff
  display: flex
  flex-flow: row wrap
  justify-content: space-around
#app > section
  border: 1px solid #eee
  box-sizing: border-box
  height: 150px
  margin: 1%
  padding: 1%
  width: 48%
h1
  color: #2c3e50
  flex: 0 1 100%
  font-size: 2em
  margin: 2%
a
  color: #fff
</style>
