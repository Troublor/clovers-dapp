<template lang="pug">
  section
    featured.mb2.md-my3(v-if="showFeatured && filters.page === 1")
    //- pagination
    filters-nav(:page="filters.page", :maxPages="maxPage", :canPrev="prevPossible", :canNext="nextPossible", @prev="back", @next="forward")

    section
      //- (list)
      .fade-enter-active(v-if="hasResults", :class="{'opacity-50':loading}")
        album-list-cards(v-if="albums.length", :albums="albums")
        page-nav(:hasResults="hasResults", :canPrev="prevPossible", :canNext="nextPossible", @prev="back", @next="forward")

      //- (empty)
      div(v-else)
        p.center.p2.lg-mt4
          span(v-if="loading") Loading...
          span(v-else) No Albums

</template>

<script>
import AlbumListCards from '@/components/AlbumList--Cards'
import FiltersNav from '@/components/FiltersNav'
import PageNav from '@/components/PageNav'
import Featured from '@/components/Featured'
import { cleanObj } from '@/utils'
import { mapState, mapGetters } from 'vuex'

export default {
  name: 'PagedAlbums',
  props: {
    apiPath: { type: String, default: '/albums' },
    showFeatured: { type: Boolean, default: false }
  },
  data () {
    return {
      loading: false,
      filters: {
        page: 1 // (temp) change back to: undefined
      }
    }
  },
  computed: {
    ...mapGetters(['apiBase']),
    ...mapState(['pagedAlbums']),
    apiUrl () {
      return `${this.apiBase}${this.apiPath}`
    },
    albums () {
      if (!this.results.results) return []
      return this.results.results
    },
    results () {
      return this.pagedAlbums
    },
    hasResults () {
      return this.results.results && !!this.results.results.length
    },
    prevPossible () {
      return this.results.prevPage
    },
    nextPossible () {
      return this.results.nextPage
    },
    maxPage () {
      if (!this.results.allResults) return 0
      return Math.ceil(this.results.allResults / 12)
    }
  },
  methods: {
    query () {
      if (this.loading || !this.apiUrl) return
      this.loading = true

      this.$store.dispatch('getPagedAlbums', {
        url: this.apiUrl,
        filters: this.filters
      }).then(() => {
        this.loading = false
      }).catch((error) => {
        console.error(error)
        this.loading = false
      })
    },
    back () {
      if (!this.prevPossible) return
      this.filters.page = this.results.prevPage
    },
    forward () {
      if (!this.nextPossible) return
      this.filters.page = this.results.nextPage
    },
    setFilters () {
      const { query } = this.$route
      this.filters.page = query.page || 1
      // this.filters.filter = query.filter || undefined
      // this.filters.sort = query.sort || undefined
      // this.filters.asc = query.asc || false
    }
  },
  mounted () {
    this.setFilters()
    this.query()
  },
  watch: {
    filters: {
      deep: true,
      handler ({ filter }) {
        let q = { ...this.filters }
        cleanObj(q)
        const cf = this.$route.query.filter
        if (cf !== filter) {
          delete q.page
        }
        this.$router.push({ name: 'Albums', query: { ...q } })
      }
    },
    $route () {
      window.scroll(0, 0)
      this.setFilters()
      this.query()
    }
  },
  components: { AlbumListCards, FiltersNav, PageNav, Featured }
}
</script>
