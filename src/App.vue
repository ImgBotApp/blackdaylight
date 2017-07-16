
<template>
  <div id="app" v-if="categoryResponse.length">
    <h1>blackdaylight</h1>
    <section v-if="category.id !== 1 && category.name !== 'home'" :class="category.slug" v-for="category in categoryResponse">
      <category :category="category" :posts="postResponse"></category>
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
      categoryResponse: [],
      postResponse: []
    }
  },
  methods: {
    fetchCategories: function () {
      fetch('http://50.87.249.59/~blackday/wp-json/wp/v2/categories?per_page=100')
      .then(
        function (response) {
          if (response.status !== 200) {
            console.log('Looks like there was a problem. Status Code: ' +
              response.status)
            return
          }

          // Examine the text in the response
          response.json().then(function (data) {
            this.categoryResponse = data
          }.bind(this))
        }.bind(this)
      )
      .catch(function (err) {
        console.log('Fetch Error :-S', err)
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
            console.log(data)
            this.postResponse = data
          }.bind(this))
        }.bind(this)
      )
      .catch(function (err) {
        console.error('Fetch Error :-S', err)
      })
    }
  },
  mounted: function () {
    this.fetchCategories()
    this.fetchPosts()
  }
}
</script>

<style lang="sass">
#app
  font-family: 'Avenir', Helvetica, Arial, sans-serif
  -webkit-font-smoothing: antialiased
  -moz-osx-font-smoothing: grayscale
  text-align: center
  color: #2c3e50
  display: flex;
  flex-flow: row wrap;
  justify-content: space-around;
#app > section
  border: 1px solid #eee
  box-sizing: border-box
  margin: 1%
  padding: 1%
  width: 48%
h1
  flex: 0 1 100%
  font-size: 2em
  margin: 2%
</style>
