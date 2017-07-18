
<template>
  <div id="app" v-if="categories.length">
    <h1>blackdaylight</h1>
    <section v-if="category.id !== 1 && category.name !== 'home'" :id="category.slug" v-for="category in categories">
      <category :category="category" :posts="linkPosts"></category>
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
      linkPosts: []
    }
  },
  methods: {
    fetchAll: function () {
      axios.all([this.fetchCategories(), this.fetchPosts()])
        .then(() => {
          this.setPostCategoryNames()
        }).catch(function (error) {
          console.error('fetchAll ' + error)
        })
    },
    fetchCategories: function () {
      return axios.get('http://50.87.249.59/~blackday/wp-json/wp/v2/categories?per_page=25')
        .then(response => {
          this.categories = response.data
        }).catch(function (error) {
          console.error('fetchCategories ' + error)
        })
    },
    fetchPosts: function () {
      return axios.get('http://50.87.249.59/~blackday/wp-json/wp/v2/posts?per_page=100')
        .then(response => {
          this.linkPosts = response.data
        }).catch(function (error) {
          console.error('fetchPosts ' + error)
        })
    },
    setPostCategoryNames: function () {
      const categories = this.categories

      this.linkPosts.forEach(function (post) {
        // add category name to post data, except Uncategorized
        post.categories.forEach(function (category, i) {
          // TODO: if category name has already been found, don't call getCategoryById
          if (category !== 1) {
            post.categories[i] = {
              id: category,
              name: getCategoryById(category)
            }
          }
        })
      })
      this.filterPosts()

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
    filterPosts: function () {
      const linkPosts = []
      const imagePosts = []

      this.linkPosts.forEach(function (post) {
        if (post.format === 'image') {
          imagePosts.push(post)
        } else if (post.format === 'link') {
          linkPosts.push(post)
        }
      })

      this.linkPosts = linkPosts
      this.setBackgroundImages(imagePosts)
    },
    setBackgroundImages: function (posts) {
      posts.forEach(function (post) {
        // find data-medium-file w/in post.content.rendered
        var imgSrc = post.content.rendered.split('"')[17]

        // if name contains a space, replace it with a -
        var id = post.categories[0].name.replace(/\s+/g, '-')

        document.getElementById(id).style.backgroundImage = 'url(' + imgSrc + ')'
      })
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
