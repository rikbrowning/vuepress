<template>
  <nav class="nav-links" v-if="userLinks.length || githubLink">
    <!-- user links -->
    <div
      class="nav-item"
      v-for="item in userLinks"
      :key="item.link">
      <DropdownLink v-if="item.type === 'links'" :item="item"/>
      <NavLink v-else :item="item"/>
    </div>
    <!-- github link -->
    <a v-if="githubLink"
      :href="githubLink"
      class="github-link"
      target="_blank"
      rel="noopener noreferrer">
      GitHub
      <OutboundLink/>
    </a>
  </nav>
</template>

<script>
import OutboundLink from './OutboundLink.vue'
import DropdownLink from './DropdownLink.vue'
import { isActive, resolveNavLinkItem } from './util'
import NavLink from './NavLink.vue'

export default {
  components: { OutboundLink, NavLink, DropdownLink },
  computed: {
    userNav () {
      const { nav } = this.$site.themeConfig
      if (Array.isArray(nav)) return nav
      if (typeof nav === 'object') return nav[this.$basepath]
      return []
    },
    nav () {
      if (this.$site.langs && this.$site.langs.length) {
        let currentLink = this.$page.path
        const routes = this.$router.options.routes
        const languageDropdown = {
          text: this.$langConfig.selectText,
          items: this.$site.langs.map(lang => {
            const text = lang.label
            let link
            // Stay on the current page
            if (lang.lang === this.$lang) {
              link = currentLink
            } else {
              // Try to stay on the same page
              link = currentLink.replace(this.$langConfig.path, lang.path)
              // fallback to homepage
              if (!routes.some(route => route.path === link)) {
                link = lang.path
              }
            }
            return { text, link }
          })
        }
        return [...this.userNav, languageDropdown]
      }
      return this.userNav
    },
    userLinks () {
      return (this.nav || []).map((link => {
        return Object.assign(resolveNavLinkItem(link), {
          items: (link.items || []).map(resolveNavLinkItem)
        })
      }))
    },
    githubLink () {
      const { repo } = this.$site.themeConfig
      if (repo) {
        return /^https?:/.test(repo)
          ? repo
          : `https://github.com/${repo}`
      }
    }
  },
  methods: {
    isActive
  }
}
</script>

<style lang="stylus">
@import './styles/config.styl'

.nav-links
  display inline-block
  a
    line-height 1.25rem
    color inherit
    &:hover, &.router-link-active
      color $accentColor
  .nav-item
    cursor: pointer
    position relative
    display inline-block
    margin-left 1.5rem
    font-weight 500
    line-height 2rem
  .github-link
    margin-left 1.5rem

@media (max-width: $MQMobile)
  .nav-links
    .nav-item, .github-link
      margin-left 0

@media (min-width: $MQMobile)
  .nav-links
    a
      &:hover, &.router-link-active
        color $textColor
        margin-bottom -2px
        border-bottom 2px solid lighten($accentColor, 5%)
</style>
