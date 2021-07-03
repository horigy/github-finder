<template>
  <div class="wrapper-content wrapper-content--fixed">
    <section>
      <div class="container">

        <!-- erorrs -->
        <div class="error" v-if="error" style="margin-bottom: 20px;">
          <p> {{ error }} </p>
        </div>

        <!-- search -->
        <search
          :value="search"
          placeholder="Type username..."
          @search="search = $event"/>

        <!-- buttons -->
        <button v-if="!repos" class="btn btnPrimary" @click="getData">Search!</button>
        <button v-else class="btn btnPrimary" @click="getData">Search Again!</button>

        <!-- user -->
        <div class="user__wrapper" v-if="user">
          <h1>User</h1>
          <div class="user__info">
            <div class="user__main">
              <img :src="user.avatar_url" class="user__avatar">
              <a :href="user.html_url" class="user__name link">{{ user.login }}</a>
            </div>
            <div class="user__repos">{{ reposCount }}</div>
          </div>

        </div>

        <!-- wrapper -->
        <div class="repos__wrapper" v-if="repos">
          <h2>Repositories</h2>

          <!-- items -->
          <table class="repo__table">
            <thead>
              <tr>
                <th @click="sort('name')">Name &#8595;</th>
                <th @click="sort('stargazers_count')">Stars &#8595;</th>
            </tr>
            </thead>
            <tbody>
              <tr v-for="repo in repoSort" :key="repo.id">
                <td>
                  <a class="link" target="_blank" :href="repo.html_url">{{ repo.name }}</a>
                </td>
                <td> {{ repo.stargazers_count }} ⭐ </td>
              </tr>
            </tbody>
          </table>

          <!-- pages -->
          <div class="container">
            <div class="button-list">
              <div class="btn btnPrimary" @click="prevPage"> &#8592; </div>
              <div class="page__number">{{ page.current }}</div>
              <div class="btn btnPrimary" @click="nextPage"> &#8594; </div>
            </div>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
import search from '@/components/Search.vue'
import axios from 'axios'
export default {
  components: { search },
  data () {
    return {
      search: '',
      error: null,
      user: null,
      repos: null,
      reposCount: null,
      reposort: {
        currentSort: 'name',
        currentSortDir: 'asc',
      },
      page: {
        current: 1,
        length: 4
      }
    }
  },
  computed: {
    repoSort() {
      return this.repos.sort((a, b) => {
        let mod = 1
        if (this.reposort.currentSortDir === 'desc') mod = -1
        if (a[this.reposort.currentSort] < b[this.reposort.currentSort]) return -1 * mod
        if (a[this.reposort.currentSort] > b[this.reposort.currentSort]) return 1 * mod
        return 0
      }).filter((row, index) => {
        let start = (this.page.current-1)*this.page.length
        let end = this.page.current * this.page.length
        if (index >= start && index < end) return true
      })
    }
  },
  methods: {
    async getData() {
      try {
        await axios.get(`https://api.github.com/users/${this.search}`)
          .then(res => {
            this.error = null
            this.user = res.data
          })
        await axios.get(`https://api.github.com/users/${this.search}/repos`)
          .then(res2 => {
            this.repos = res2.data
            this.reposCount = this.repos.length
          })
      }
      catch {
        console.log(err)
        this.user = null
        this.repos = null
        this.reposCount = null
        this.error = 'Can`t find this user'
      }
    },
    sort (e) {
      if (e === this.reposort.currentSort) {
        this.reposort.currentSortDir = this.reposort.currentSortDir === 'asc' ? 'desc' : 'asc'
      }
      this.reposort.currentSort = e
    },
    // Pagination
    prevPage () {
      if (this.page.current > 1) this.page.current-=1
    },
    nextPage () {
      if ((this.page.current * this.page.length) < this.repos.length) this.page.current+=1
    }
  }
}
</script>

<style lang="scss" scoped>
.container {
  display: flex;
  align-items: center;
  flex-direction: column;
}
button {
  margin-top: 40px;
}
h1, h2 {
  margin-bottom: 15px;
}
.user__wrapper {
  width: 400px;
  margin: 30px 0;
  margin: 30px 0;
}
.user__info {
  width: 100%;
  display: flex;
  justify-content: space-between;

}
.user__avatar {
  width: 30px;
  height: 30px;
  margin-right: 30px;
}
.repos__wrapper {
  width: 400px;
  margin: 30px 0;
}
.repos-info {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 10px;
  padding: 10px 0;
  border-bottom: 1px solid #dbdbdb;
}
.button-list {
  display: flex;
  align-items: center;
}
.page__number {
  font-weight: bold;
  font-size: 20px;
  margin: 0 20px;
}
</style>
