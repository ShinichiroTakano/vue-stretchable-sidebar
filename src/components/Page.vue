<template>
  <div id="page">
    <StretchableSidebar
      :isSidebarOpened="isSidebarOpened"
      :style="stretchableSidebarComputedStyle" />
    <SidebarBorder
      :isSidebarOpened="isSidebarOpened"
      @mousedown.native="startStretch"
      @mouseup.native="finishStretch"
      @toggle-sidebar="toggleSidebar" />
  </div>
</template>

<script>
import StretchableSidebar from './StretchableSidebar.vue'
import SidebarBorder from './SidebarBorder.vue'
const TOGGLE_BTN_WIDTH = 35

export default {
  name: 'page',
  components: {
    StretchableSidebar,
    SidebarBorder
  },
  data() {
    return {
      stretchableSidebarStyle: {
        width: 0.2
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
      return this.stretchableSidebarStyle.width !== this.sidebarMinSize
    },
    sidebarMinSize () {
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
      // ページ全体を包む要素の横幅と高さを保存。
      const { width, height } = document.getElementById('page').getBoundingClientRect()
      this.pageRect.width = width
      this.pageRect.height = height
    },
    setToggleBtnStyle () {
      this.toggleBtnStyle.width = TOGGLE_BTN_WIDTH / this.pageRect.width
    },
    startStretch () {
      window.addEventListener('mousemove', this.handleStretch)
    },
    finishStretch () {
      window.removeEventListener('mousemove', this.handleStretch)
    },
    handleStretch (event) {
      const { pageX } = event
      // 画面最左からポインターまでの距離 / 画面全体の横幅
      const sidebarWidth = pageX / this.pageRect.width
      if (this.checkIsSidebarEnoughWide(sidebarWidth)) {
        this.stretchableSidebarStyle.width = sidebarWidth
      } else {
        this.stretchableSidebarStyle.width = this.sidebarMinSize
        window.removeEventListener('mousemove', this.handleStretch)
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
        this.stretchableSidebarStyle.width = 0.2
      } else {
        this.stretchableSidebarStyle.width = this.sidebarMinSize
      }
    },
    checkIsSidebarEnoughWide(sidebarWidth) {
      return sidebarWidth > this.sidebarMinSize
    }
  }
}
</script>

<style scoped>
#page {
  display: flex;
}
</style>
