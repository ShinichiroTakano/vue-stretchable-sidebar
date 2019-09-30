<template>
  <div id="sidebar-border">
    <span id="sidebar-border-btn" @click.stop="toggleSidebar">
      <i :class="['fas', isSidebarOpened ? 'fa-chevron-left' : 'fa-bars']" />
    </span>
  </div>
</template>
<script>
export default {
  name: 'sidebar-border',
  props: {
    isSidebarOpened: {
      type: Boolean,
      required: true
    }
  },
  methods: {
    toggleSidebar () {
      this.$emit('toggle-sidebar')
    }
  }
}
</script>
<style scoped>
#sidebar-border {
  width: 3px;
  height: 100vh;
  position: relative;
}
#sidebar-border:hover {
  background-color: #708090;
  cursor: col-resize;
}
#sidebar-border-btn {
  width: 35px;
  height: 35px;
  background-color: #B0C4DE;
  border: 1px solid #708090;
  border-radius: 50%;
  display: inline-block;
  text-align: center;
  box-shadow: .5px .5px .5px rgba(0,0,0,0.6);
  position: absolute;
  top: 17.5px;
  left: -17.5px;
}
.fas {
  line-height: 35px;
  color: #696969;
}
.fa-chevron-left {
  cursor: w-resize;
}
.fa-bars {
  cursor: e-resize;
}
</style>