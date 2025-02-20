<template>
  <div class="buttons">
    <transition name="fade">
      <div
        title="返回顶部"
        class="button go-to-top iconfont icon-fanhuidingbu"
        v-show="showToTop"
        @click="scrollToTop"
      />
    </transition>
    <div
      title="去评论"
      class="button go-to-comment iconfont icon-pinglun"
      v-show="showCommentBut"
      @click="scrollToComment"
    />
    <div
      title="主题模式"
      class="button theme-mode-but iconfont icon-zhuti"
      @mouseenter="showModeBox = true"
      @mouseleave="showModeBox = false"
      @click="showModeBox = true"
    >
      <transition name="mode">
        <ul class="select-box" ref="modeBox" v-show="showModeBox" @click.stop>
          <li
          v-for="item in modeList"
          :key="item.KEY"
          class="iconfont"
          :class="[item.icon, {active: item.KEY === currentMode}]"
          @click="toggleMode(item.KEY)"
          >
            {{item.name}}
          </li>
        </ul>
      </transition>
    </div>
  </div>
</template>

<script>
import debounce from 'lodash.debounce'
import storage from 'good-storage' // 本地存储
const MOBILE_DESKTOP_BREAKPOINT = 719 // refer to config.styl

export default {
  data() {
    return {
      threshold: 100,
      scrollTop: null,
      showCommentBut: false,
      commentTop: null,
      currentMode: null,
      showModeBox: false,
      modeList: [
        {
          name: '跟随系统',
          icon: 'icon-zidong',
          KEY: 'auto'
        },
        {
          name: '浅色模式',
          icon: 'icon-rijianmoshi',
          KEY: 'light'
        },
        {
          name: '深色模式',
          icon: 'icon-yejianmoshi',
          KEY: 'dark'
        },
        {
          name: '阅读模式',
          icon: 'icon-yuedu',
          KEY: 'read'
        }
      ],
      _scrollTimer: null,
      _textareaEl: null,
      _recordScrollTop: null,
      COMMENT_SELECTOR: '#vuepress-plugin-comment' // 评论区元素的选择器
    }
  },
  mounted () {
    this.currentMode = storage.get('mode') || 'auto'

    this.scrollTop = this.getScrollTop()
    window.addEventListener('scroll', debounce(() => {
      this.scrollTop = this.getScrollTop()
    }, 100))

    this.handleShowCommentBut()
    window.addEventListener('load', () => {
      this.getCommentTop()
    })

    // 小屏时选择主题模式后关闭选择框
    if (document.documentElement.clientWidth < MOBILE_DESKTOP_BREAKPOINT) {
      const modeBox = this.$refs.modeBox
      modeBox.onclick = () => {
        this.showModeBox = false
      }
      window.addEventListener('scroll', debounce(() => {
        if(this.showModeBox) {
          this.showModeBox = false
        }
      }, 100))
    }
  },
  computed: {
    showToTop () {
      return this.scrollTop > this.threshold
    }
  },
  methods: {
    toggleMode(key){
      this.currentMode = key
      this.$emit('toggle-theme-mode', key)
    },
    getScrollTop () {
      return window.pageYOffset
        || document.documentElement.scrollTop
        || document.body.scrollTop || 0
    },

    scrollToTop () {
      window.scrollTo({ top: 0, behavior: 'smooth' })
      this.scrollTop = 0
    },

    getCommentTop () {
      setTimeout(() => {
        const commentEl = document.querySelector(this.COMMENT_SELECTOR)
        if (commentEl) {
          this.commentTop = commentEl.offsetTop
        } else {
          this.showCommentBut = false
        }
      },500)
    },

    handleShowCommentBut() {
      this.showCommentBut = this.$frontmatter.comment !== false && this.$frontmatter.home !== true
    },

    scrollToComment() {
      window.scrollTo({ top: this.commentTop, behavior: 'smooth' })
      this._textareaEl = document.querySelector(this.COMMENT_SELECTOR + ' textarea')
      if( this._textareaEl && this.getScrollTop() !== this._recordScrollTop) {
        document.addEventListener("scroll", this._handleListener)
      } else if (this._textareaEl && this.getScrollTop() === this._recordScrollTop) {
        this._handleFocus()
      }
    },

    _handleListener() {
      clearTimeout(this._scrollTimer)
      this._scrollTimer = setTimeout(() => {
        document.removeEventListener('scroll', this._handleListener)
        this._recordScrollTop = this.getScrollTop()
        this._handleFocus()
      }, 30)
    },

    _handleFocus() {
      this._textareaEl.focus()
      this._textareaEl.classList.add('yellowBorder')
      setTimeout(() => {
        this._textareaEl.classList.remove('yellowBorder')
      }, 500)
    }
  },
  watch: {
    $route() {
      this.handleShowCommentBut()
      this.getCommentTop()
    }
  }
}
</script>

<style lang='stylus'>
  .yellowBorder
    border: #FFE089 1px solid!important
    box-shadow 0 0 10px #FFE089!important
  .buttons
    position fixed
    right 2rem
    bottom 2.5rem
    z-index 11
    @media (max-width: $MQNarrow)
      right 1rem
      bottom 1.5rem
    .button
      width 2.2rem
      height 2.2rem
      line-height 2.2rem
      border-radius 50%
      box-shadow 0 2px 6px rgba(0,0,0,.25)
      margin-top .9rem
      text-align center
      cursor pointer
      // color var(--textLightenColor)
      background rgba(255,255,255,.1)
      &:hover
        &:before
          color $accentColor
      .select-box
        margin 0
        padding .5rem 0
        position absolute
        bottom 0rem
        right 1.5rem
        background var(--bg)
        border 1px solid var(--borderColor)
        width 100px
        border-radius 3px
        box-shadow 0 2px 6px rgba(0,0,0,.25)
        li 
          list-style none
          line-height 1.8rem
          font-size .95rem
          &:hover
            color $accentColor
          &.active
            background-color rgba(150,150,150,.2)
            color $accentColor

  .mode-enter-active, .mode-leave-active
    transition all .3s
  .mode-enter, .mode-leave-to
    opacity 0
    transform scale(.8)

  .fade-enter-active, .fade-leave-active
    transition opacity .2s
  .fade-enter, .fade-leave-to
    opacity 0
</style>
