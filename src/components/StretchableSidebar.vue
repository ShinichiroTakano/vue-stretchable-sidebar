<template>
  <aside id="strechable-sidebar">
    <div :style="sidebarContentComputedStyle">
      <ul>
        <li>あああああ</li>
        <li>いいいいい</li>
        <li>ううううう</li>
      </ul>
    </div>
  </aside>
</template>
<script>
export default {
  name: 'stretchable-sidebar',
  props: {
    isSidebarOpened: {
      type: Boolean,
      required: true
    }
  },
  computed: {
    sidebarContentComputedStyle () {
      if (this.isSidebarClosed) {
        return { transform: `translateX(-${window.innerWidth}px)` }
      } else {
        return {}
      }
    },
    isSidebarClosed () {
      return !this.isSidebarOpened
    }
  }
}
</script>
<style scoped>
#strechable-sidebar {
  background-color: rgb(244, 245, 247);
  height: 100vh;
  overflow: hidden;
  user-select: none;
}
</style>