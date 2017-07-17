
<template>
  <div id="app" v-if="categories.length">
    <h1>blackdaylight</h1>
    <section v-if="category.id !== 1 && category.name !== 'home'" :id="category.slug" v-for="category in categories">
      <category :category="category" :posts="linkPosts"></category>
    </section>
  </div>
</template>

<script>
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
    fetchCategories: function () {
      fetch('http://50.87.249.59/~blackday/wp-json/wp/v2/categories?per_page=100')
        .then(
          function (response) {
            if (response.status !== 200) {
              console.error('Looks like there was a problem. Status Code: ' +
                response.status)
              return
            }

            // Examine the text in the response
            response.json().then(function (data) {
              this.categories = data
              this.fetchPosts()
            }.bind(this))
          }.bind(this)
        )
        .catch(function (error) {
          console.error('Fetch Error :-S', error)
        })
    },
    fetchPosts: function () {
      fetch('http://50.87.249.59/~blackday/wp-json/wp/v2/posts?per_page=100')
        .then(
          function (response) {
            if (response.status !== 200) {
              console.log('Looks like there was a problem. Status Code: ' +
                response.status)
              return
            }

            // Examine the text in the response
            response.json().then(function (data) {
              this.linkPosts = data
              this.setPostCategoryNames()
            }.bind(this))
          }.bind(this)
        )
        .catch(function (error) {
          console.error('Fetch Error :-S', error)
        })
    },
    setPostCategoryNames: function () {
      var categories = this.categories

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
        var numCategories = categories.length

        // return category name, if id exists
        for (let i = 0; i < numCategories; i++) {
          if (categories[i].id === id) {
            return categories[i].name
          }
        }

        console.error('Category ID is not available.')
        return
      }
    },
    filterPosts: function () {
      var linkPosts = []
      var imagePosts = []

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
    this.fetchCategories()
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
