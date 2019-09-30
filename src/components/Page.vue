<template>
  <div id="page">
    <StretchableSidebar
      :isSidebarOpened="isSidebarOpened"
      :style="stretchableSidebarComputedStyle" />
    <SidebarBorder
      :isSidebarOpened="isSidebarOpened"
      @mousedown.native="startStretch"
      @toggle-sidebar="toggleSidebar" />
  </div>
</template>

<script>
import StretchableSidebar from './StretchableSidebar.vue'
import SidebarBorder from './SidebarBorder.vue'
const TOGGLE_BTN_WIDTH = 35
const DEFAULT_SIDEBAR_WIDTH = 0.2

export default {
  name: 'page',
  components: {
    StretchableSidebar,
    SidebarBorder
  },
  data() {
    return {
      stretchableSidebarStyle: {
        width: DEFAULT_SIDEBAR_WIDTH // 初期表示時の横幅は親要素の20%
      },
      pageRect: {
        width: 0,
        height: 0
      },
      toggleBtnStyle: {
        width: null
      }
    }
  },
  computed: {
    stretchableSidebarComputedStyle () {
      return { width: `${ this.stretchableSidebarStyle.width * 100 }%` }
    },
    isSidebarOpened () {
      return this.stretchableSidebarStyle.width > this.sidebarMinSize
    },
    sidebarMinSize () {
      // トグルボタンの横幅の割合の半分をサイドバーの最小値にする
      return this.toggleBtnStyle.width / 2
    }
  },
  mounted() {
    this.setScreenData()
    this.addResizeEvent()
  },
  beforeDestroy() {
    this.removeResizeEvent()
  },
  methods: {
    setScreenData () {
      this.setPageRect()
      this.setToggleBtnStyle()
    },
    setPageRect () {
      // サイドバーの親要素の横幅と高さを保存。
      const { width, height } = document.getElementById('page').getBoundingClientRect()
      this.pageRect.width = width
      this.pageRect.height = height
    },
    setToggleBtnStyle () {
      // 35px(ボタンの横幅)が親要素の横幅に対してどれぐらいの割合かを保存する。
      this.toggleBtnStyle.width = TOGGLE_BTN_WIDTH / this.pageRect.width
    },
    startStretch () {
      // 画面上でポインターを動かす度に、handleMoveが呼ばれるようにする。
      window.addEventListener('mousemove', this.handleMove)
      window.addEventListener('mouseup', this.finishStretch)
    },
    finishStretch () {
      window.removeEventListener('mousemove', this.handleMove)
      window.removeEventListener('mouseup', this.finishStretch)
    },
    handleMove (event) {
      const { pageX } = event
      const sidebarWidth = pageX / this.pageRect.width // サイドバーの親要素に対する横幅の割合 = 画面最左からポインターまでの距離 / 親要素の横幅
      if (sidebarWidth >= this.sidebarMinSize) {
        this.stretchableSidebarStyle.width = sidebarWidth
      } else {
        this.stretchableSidebarStyle.width = this.sidebarMinSize
        this.finishStretch()
      }
    },
    addResizeEvent () {
      window.addEventListener('resize', this.setScreenData)
    },
    removeResizeEvent () {
      window.removeEventListener('resize', this.setScreenData)
    },
    toggleSidebar () {
      if (this.stretchableSidebarStyle.width === this.sidebarMinSize) {
        this.stretchableSidebarStyle.width = DEFAULT_SIDEBAR_WIDTH
      } else {
        this.stretchableSidebarStyle.width = this.sidebarMinSize
      }
    }
  }
}
</script>

<style scoped>
#page {
  display: flex;
}
</style>
