
<template>
  <div id="app" v-if="categoryResponse.length">
    <h1>blackdaylight</h1>
    <div v-if="category.id !== 1 && category.name !== 'home'" v-bind:class="category.slug" v-for="category in categoryResponse">
      <h2 v-on:click="toggle(clicked)">
        {{ category.name }}
      </h2>
      <ul :class="{selected: clicked}" >
        <li v-if="category.id === post.categories[0]" v-for="post in postResponse" v-html="post.content.rendered"></li>
      </ul>
    </div>
  </div>
</template>

<script>

export default {
  name: 'app',
  components: {
  },
  data () {
    return {
      categoryResponse: [],
      postResponse: [],
      clicked: false
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
        console.log('Fetch Error :-S', err)
      })
    },
    toggle: function (clicked) {
      this.clicked = !clicked
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
#app > div
  border: 1px solid #eee
  box-sizing: border-box
  margin: 1%
  padding: 1%
  width: 48%
h1
  flex: 0 1 100%
  font-size: 2em
  margin: 2%
h2
  font-size: 1.5em
  margin: 1%
</style>
