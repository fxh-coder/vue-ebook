<template>
  <div class="ebook-reader">
    <div id="read"></div>
    <div class="ebook-reader-mask"
    @touchmove="move"
    @touchend="moveEnd"
    @mousedown.left="onMouseEnter"
    @mousemove.left="onMouseMove"
    @mouseup.left="onMouseEnd"
    @click="onMaskClick"></div>
    <!-- <div class="read-wrapper">
      <div class="mask">
        <div class="left" @click="prevPage"></div>
        <div class="center" @click="toggleTitleAndMenu"></div>
        <div class="right" @click="nextPage"></div>
      </div>
    </div> -->
  </div>
</template>

<script>
import { ebookMinxin } from '../../utils/mixin'
import { flatten } from '../../utils/book'
import {
  getFontFamily,
  saveFontFamily,
  getFontSize,
  saveFontSize,
  getTheme,
  saveTheme,
  getLocation
} from '../../utils/localStorage'
import Epub from 'epubjs'
global.ePub = Epub
export default {
  mixins: [ebookMinxin],
  methods: {
    onMouseEnd(e) {
      if(this.mouseState === 2) {
        this.setOffsetY(0)
        this.firstOffsetY = null
        this.mouseState = 3
      } else {
        this.mouseState = 4
      }
      const time = e.timeStamp - this.mouseStartTime
      if(time < 100) {
        this.mouseState = 4
      }
      e.preventDefault()
      e.stopPropagation()
    },
    onMouseMove(e) {
      if(this.mouseState === 1) {
        this.mouseState = 2
      } else if(this.mouseState === 2) {
        let offsetY = 0
        if(this.firstOffsetY) {
          offsetY = e.clientY - this.firstOffsetY
          this.setOffsetY(offsetY)
        } else {
          this.firstOffsetY = e.clientY
        }
      }
      e.preventDefault()
      e.stopPropagation()
    },
    onMouseEnter(e) {
      this.mouseState = 1
      this.mouseStartTime = e.timeStamp
      e.preventDefault()
      e.stopPropagation()
    },
    move(e) {
      let offsetY = 0
      if(this.firstOffsetY) {
        offsetY = e.changedTouches[0].clientY - this.firstOffsetY
        this.setOffsetY(offsetY)
      } else {
        this.firstOffsetY = e.changedTouches[0].clientY
      }
      e.preventDefault()
      e.stopPropagation()
    },
    moveEnd(e) {
      this.setOffsetY(0)
      this.firstOffsetY = null
    },
    onMaskClick(e) {
      if(this.mouseState && (this.mouseState === 2 || this.mouseState === 3)) {
        return
      }
      const offsetX = e.offsetX
      const width = window.innerWidth
      if(offsetX > 0 && offsetX < width * 0.3) {
        this.prevPage()
      } else if(offsetX > 0 && offsetX > width * 0.7) {
        this.nextPage()
      } else {
        this.toggleTitleAndMenu()
      }
    },
    prevPage() {
      if(this.rendition) {
        this.rendition.prev().then(() => {
          this.refreshLocation()
        })
        this.hideTitleAndMenu()
      }
    },
    nextPage() {
      if(this.rendition) {
        this.rendition.next().then(() => {
          this.refreshLocation()
        })
        this.hideTitleAndMenu()
      }
    },
    toggleTitleAndMenu() {
      if(this.menuVisible) {
        this.setSettingVisible(-1)
        this.setFontFamilyVisible(false)
      }
      this.setMenuVisible(!this.menuVisible)
    },
    initFontSize() {
      const fontSize = getFontSize(this.fileName)
      if(!fontSize) {
        saveFontSize(this.fileName, this.defaultFontSize)
      } else {
        this.setDefaultFontSize(fontSize)
        this.rendition.themes.fontSize(fontSize)
      }
    },
    initFontFamily() {
      const font = getFontFamily(this.fileName)
      if(!font) {
        saveFontFamily(this.fileName, this.defaultFontFamily)
      } else {
        this.rendition.themes.font(font)
        this.setDefaultFontFamily(font)
      }
    },
    initTheme() {
      let defaultTheme = getTheme(this.fileName)
      if(!defaultTheme) {
        defaultTheme = this.themeList[0].name
        saveTheme(this.fileName, defaultTheme)
      }
      this.setDefaultTheme(defaultTheme)
      this.themeList.forEach(theme => {
        this.rendition.themes.register(theme.name, theme.style)
      })
      this.rendition.themes.select(defaultTheme)
    },
    initRendition() {
      this.rendition = this.book.renderTo('read', {
        width: innerWidth,
        height: innerHeight,
        method: 'default'
      })
      const location = getLocation(this.fileName)
      this.display(location, () => {
        this.initTheme()
        this.initFontSize()
        this.initFontFamily()
        this.initGlobalStyle()
      })
      this.rendition.hooks.content.register(contents => {
        Promise.all([
          contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/epub/book/res/fonts/daysOne.css`),
          contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/epub/book/res/fonts/cabin.css`),
          contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/epub/book/res/fonts/montserrat.css`),
          contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/epub/book/res/fonts/tangerine.css`)
        ]).then(() => {})
      })
    },
    initGesture() {
      this.rendition.on('touchstart', event => {
        this.touchStartX = event.changedTouches[0].clientX
        this.touchStartTime = event.timeStamp
      })
      this.rendition.on('touchend', event => {
        const offsetX = event.changedTouches[0].clientX - this.touchStartX
        const time = event.timeStamp - this.touchStartTime
        if (time < 500 && offsetX > 40) {
          this.prevPage()
        } else if (time < 500 && offsetX < -40) {
          this.nextPage()
        } else {
          this.toggleMenuVisible()
        }
        event.preventDefault()
        event.stopPropagation()
      })
    },
    parseBook() {
      this.book.loaded.cover.then(cover => {
        this.book.archive.createUrl(cover).then(url => {
          this.setCover(url)
        })
      })
      this.book.loaded.metadata.then(metadata => {
        this.setMetadata(metadata)
      })
      this.book.loaded.navigation.then(nav => {
        const navItem = flatten(nav.toc)
        function find(item, level = 0) {
          return !item.parent ? level : find(navItem.filter(parentItem => parentItem.id === item.parent)[0], ++level)
        }
        navItem.forEach(item => {
          item.level = find(item)
        })
        this.setNavigation(navItem)
      })
    },
    initEpub() {
      const url = process.env.VUE_APP_RES_URL + '/epub/epub/' + this.fileName + '.epub'
      this.book = new Epub(url)
      this.setCurrentBook(this.book)
      this.initRendition()
      // this.initGesture()
      this.parseBook()
      this.book.ready.then(() => {
        return this.book.locations.generate(750 * (window.innerWidth / 375) * (getFontSize(this.fileName) / 16))
      }).then(locations => {
        this.navigation.forEach(nav => {
          nav.pagelist = []
        })
        locations.forEach(item => {
          const loc = item.match(/\[(.*)\]!/)[1]
          this.navigation.forEach(nav => {
            if(nav.href) {
              const href = nav.href.match(/^(.*)\.html$/)[1]
              if(href === loc) {
                nav.pagelist.push(item)
              }
            }
          })
          let currentPage = 1
          this.navigation.forEach((nav, index) => {
            if(index === 0) {
              nav.page = 1
            } else {
              nav.page = currentPage
            }
            currentPage += nav.pagelist.length + 1
          })
        })
        this.setPagelist(locations)
        this.setBookAvailable(true)
        this.refreshLocation()
      })
    }
  },
  mounted() {
    this.setFileName(
      this.$route.params.fileName.split('|').join('/')).then(() => {
      this.initEpub()
    })
  }
}
</script>

<style lang="scss" scoped>
@import "../../assets/styles/global";
.ebook-reader {
  position: relative;
  width: 100%;
  height: 100%;
  overflow: hidden;
  .ebook-reader-mask {
    position: absolute;
    width: 100%;
    height: 100%;
    z-index: 150;
    background: transparent;
    top: 0;
    left: 0;
  }
  .read-wrapper {
    .mask {
      position: absolute;
      top: 0;
      left: 0;
      z-index: 100;
      display: flex;
      width: 100%;
      height: 100%;
      .left {
        flex: 0 0 px2rem(100);
      }
      .center {
        flex: 1;
      }
      .right {
        flex: 0 0 px2rem(100);
      }
    }
  }
}
</style>
