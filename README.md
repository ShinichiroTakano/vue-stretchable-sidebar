### ポインターに合わせ伸縮するサイドバー
![デモ](https://github.com/ShinichiroTakano/vue-stretchable-sidebar/blob/images/fbbf620fac7bd46db0e9a14edc15c956.gif)

[ソース](https://github.com/ShinichiroTakano/vue-stretchable-sidebar)

### 動機
Vue.jsでポインターに合わせ伸縮するサイドバーを実装する機会があったので、やり方をメモしておきます。
もっと良い実装方法がありましたら、指摘して頂けましたら嬉しいです。

### 大まかな流れ

サイドバーの横に設置したボーダー要素(<SidebarBorder />)のmousedownイベントが発火する。
↓
mousemoveイベントを画面全体(window)に対して登録し(window.addEventListener('mousemove', this.handleMove))、
画面上でポインターが動く度に、handleMoveメソッドが呼ばれるようにする。
↓
handleMoveメソッドの引数から、移動後のポインターの位置を取得し、
サイドバーの横幅が画面左端からポインターまでの距離(割合)になるように変更する。
↓
ポインターの動きに合わせて、サイドバーが伸び縮みするように見える。

### 細かい点

サイドバーの親要素のwidth(pageRect.width)だけはpx値で持ち、
サイドバーのwidthやトグルボタンのwidthは、
pageRect.widthを1とした時の割合で保持しています。
理由は、割合で持っておけば、画面のリサイズが起こっても、
pageRect.widthだけ変更すれば、リサイズ前と同じ見た目が維持されるからです。

また、pageRect.widthは今回の例で言えば、window.innerWidthと同値になります。
なので、pageRect.widthと書かれているところ、window.innerWidthとしても成り立ちます。
サイドバーの親要素のwidthと画面の横幅が等しいという条件が常に成り立つ場合は、
pageRect.widthとなっているところを、window.innerWidthにした方がシンプルになり、良いかもしれません。

### サンプルコード
```Page.vue
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
```

```StretchableSidebar.vue
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
```

```SidebarBorder.vue
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
```
