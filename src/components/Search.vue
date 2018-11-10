<template lang="pug">
  .search(v-bind:class="{ isFailed: isFailed || hasNoRepos }")
    .searchBox
      h1
        label(for="search") Search GH repos
      input(
        type="text"
        name="search"
        v-on:input="searchRepos"
        autofocus)
      ul.repos
        li.repo(
          v-for="repo in repos"
          v-bind:key="repo.id"
          v-on:click="fetchContributors"
        ) {{ repo.name }}
        li.noResults(v-if="isFailed")
          span.emoji 🙁
          span no user have been found
        li.noResults(v-if="hasNoRepos")
          span.emoji ☹️
          span this user has no repos
    Chart(v-if="contributors" :contributors="contributors")
</template>

<script>
  import Chart from './Chart';

  export default {
    data: function() {
      return {
        repos: null,
        contributors: null,
        isFailed: false,
        hasNoRepos: false,
        timer: null,
        username: '',
      }
    },
    name: 'app-search',
    components: {
      Chart,
    },
    methods: {
      searchRepos: function(e) {
        const { value } = e.target;
        this.username = value;

        if (this.timer) clearTimeout(this.timer);
        if (!value) {
          this.isFailed = this.hasNoRepos = false;
          return;
        }
        this.timer = setTimeout(() => this.fetchUserRepos(value), 350);
      },
      fetchUserRepos: function(username) {
        const url = `https://api.github.com/users/${username}/repos`;
        fetch(url)
          .then(function(response) {
            if (response.status === 200) {
              response.json().then(function(data) {
                this.repos = data;
                if (data.length) {
                  this.isFailed = this.hasNoRepos = false;
                }
                else {
                  this.isFailed = false;
                  this.hasNoRepos = true;
                }
              }.bind(this));
            }
            else {
                this.repos = [];
                this.isFailed = true;
                this.hasNoRepos = false;
            }
          }.bind(this))
          .catch(function() {
            console.warn('Some error occured');
            this.isFailed = true;
          });
      },
      fetchContributors: function(e) {
        const repo = e.target.innerHTML;
        const url = `https://api.github.com/repos/${this.username}/${repo}/contributors`;
        fetch(url)
          .then(function(response) {
            response.json().then(function(data) {
              this.contributors = data;
            }.bind(this));
          }.bind(this))
          .catch(function() {
            console.warn('Something went wrong...');
          })
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
    align-items: center
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
    margin-top: -80px

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
    max-height: 305px
    overflow: scroll

  .repo
    margin-left: 20px
    padding: 5px
    color: white
    cursor: pointer
    list-style-type: disc

    &:hover
      background-color: rgba(255, 255, 255, 0.1)
      cursor: pointer

  .noResults
    display: flex
    align-items: center
    font-weight: 600

  .emoji
    height: 24px
</style>
