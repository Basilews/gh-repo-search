<template lang="pug">
  .search(v-bind:class="{ isFailed: isFailed }")
    .searchBox
      h1
        label(for="search") Search GH repos
      input(
        type="text"
        name="search"
        v-on:input="searchRepos"
        autofocus)
      ul.repos
        li.repo(v-for="repo in repos" v-bind:key="repo.id") {{ repo.name }}
        li.noResults(v-if="repos && !repos.length" v-on:click="showContributors")
          span.emoji 🙁
          span no repositories have been found
        //- li.noResults(v-if="") span.emoji🙁 span this user has no repos
</template>

<script>
  export default {
    data: function() {
      return {
        repos: null,
        isFailed: false,
      }
    },
    name: 'app-search',
    methods: {
      searchRepos: function(e) {
        setTimeout(this.fetchUserRepos(e.target.value), 300);
      },
      fetchUserRepos: function(username) {
        const url = `https://api.github.com/users/${username}/repos?client_id=xxxx&client_secret=yyyy`;
        fetch(url)
          .then(function(response) {
            if (response.status === 200) {
              response.json().then(function(data) {
                this.repos = data;
                this.isFailed = false;
              }.bind(this));
            }
            else {
                this.repos = [];
                this.isFailed = true;
            }
          }.bind(this))
          .catch(function() {
            console.warn('Some error occured');
            this.isFailed = true;
          });
      },
      showContributors: function() {
        //
      },
    }
  }
</script>

<style lang="sass" scoped>
  .search
    position: relative
    width: 100%
    height: 100%
    display: flex
    justify-content: center
    padding: 20px 10px
    background-image: radial-gradient(#e66465, #9198e5)
    z-index: 100

    &:before
      content: ""
      position: absolute
      top: 0
      left: 0
      width: 100%
      height: 100%
      display: block
      background-image: radial-gradient(#e66465, red)
      opacity: 0
      transition: opacity .35s
      z-index: -100

    &.isFailed
      &:before
        opacity: 1

  .searchBox
    position: relative
    display: flex
    flex-direction: column

  h1
    margin-bottom: 10px
    font-size: 20px
    font-weight: 700
    color: white

  input
    width: 252px
    margin-bottom: 5px
    padding-bottom: 5px
    background: none;
    border: none;
    border-bottom: 1px solid white
    outline: 0
    font-size: 16px
    color: white

  .repos
    position: absolute
    top: 70px
    width: 100%
    max-height: calc(100% - 60px)
    overflow: scroll

  .repo
    margin-left: 20px;
    padding: 5px;
    color: white
    cursor: pointer
    list-style-type: disc;

    &:hover
      background-color: rgba(255, 255, 255, 0.1)
      cursor: pointer;

  .noResults
    display: flex;
    align-items: center;
    font-weight: 600

  .emoji
    height: 24px
</style>
