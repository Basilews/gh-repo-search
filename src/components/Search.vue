<template lang="pug">
  .search(
    v-bind:class="{ isFailed: isFailed || hasNoRepos }"
    role="search")
    .searchBox
      h1
        label(for="search") Search GH repos
      input(
        type="text"
        name="search"
        v-on:input="searchRepos"
        v-on:focus="handleInputFocus"
        v-on:blur="handleInputBlur"
        autofocus)
      ul(v-bind:class="['repos', isOpened ? 'isOpened' : '']")
        li.repo(
          v-for="repo in repos"
          v-bind:key="repo.id"
          v-on:mousedown="fetchContributors"
        ) {{ repo.name }}
      div.noResults(v-if="isFailed")
        span.emoji 🙁
        span no user have been found
      div.noResults(v-if="hasNoRepos")
        span.emoji ☹️
        span this user has no repos
    Chart(v-if="contributors" :contributors="contributors" class="chart")
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
        isOpened: false,
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
                  this.isOpened = true;
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
        console.log('fetching...')
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
      handleInputFocus: function() {
        this.isOpened = true;
      },
      handleInputBlur: function() {
        this.isOpened = false;
      }
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
    display: none
    width: 100%
    max-height: 305px
    overflow: scroll

    &.isOpened
      display: block

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

  .chart
    margin-left: 10px
</style>
