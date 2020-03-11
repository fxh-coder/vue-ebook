<template>
  <transition name="slide-up">
    <div class="setting-wrapper" v-show="menuVisible && settingVisible === 0">
      <div class="setting-font-size">
        <div class="preview" :style="{fontSize: fontSizeList[0].fontSize + 'px'}">A</div>
        <div class="select">
          <div
            class="select-wrapper"
            v-for="(item, index) in fontSizeList"
            @click="setFontSize(item.fontSize)"
            :key="index"
          >
            <div class="line"></div>
            <div class="point-wrapper">
              <div class="point" v-show="defaultFontSize === item.fontSize">
                <div class="small-point"></div>
              </div>
            </div>
            <div class="line"></div>
          </div>
        </div>
        <div
          class="preview"
          :style="{fontSize: fontSizeList[fontSizeList.length - 1].fontSize + 'px'}"
        >A</div>
      </div>
      <div class="setting-font-family" @click="showFontFamilyPopup">
        <div class="setting-font-family-text-wrapper">
          <span class="setting-font-family-text">{{defaultFontFamily}}</span>
        </div>
        <div class="setting-font-family-icon-wrapper">
          <span class="icon-forward"></span>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
import { FONT_SIZE_LIST } from '../../utils/book'
import { ebookMinxin } from '../../utils/mixin'
import { saveFontSize } from '../../utils/localStorage'
export default {
    mixins: [ebookMinxin],
    data() {
      return {
        fontSizeList: FONT_SIZE_LIST
      }
    },
    methods: {
      setFontSize(fontSize) {
        this.setDefaultFontSize(fontSize)
        saveFontSize(this.fileName, fontSize)
        this.currentBook.rendition.themes.fontSize(fontSize)
      },
      showFontFamilyPopup() {
        this.setFontFamilyVisible(true)
      }
    }
}
</script>

<style lang="scss" scoped>
@import "../../assets/styles/global";
.setting-wrapper {
    position: absolute;
    bottom: px2rem(48);
    left: 0;
    z-index: 200;
    display: flex;
    flex-direction: column;
    width: 100%;
    height: px2rem(90);
    box-shadow: 0 px2rem(-8) px2rem(8) rgba(0, 0, 0, 0.15);
    background-color: #fff;
    .setting-font-size {
      flex: 2;
      display: flex;
      height: 100%;
      .preview {
        flex: 0 0 px2rem(40);
        @include center;
        box-sizing: border-box;
        &:first-of-type {
          padding-left: calc((100% - 80px) / 14);
        }
        &:last-of-type {
          padding-right: calc((100% - 80px) / 14);
        }
      }
      .select {
        display: flex;
        flex: 1;
        .select-wrapper {
          flex: 1;
          display: flex;
          align-items: center;
          &:first-child {
            .line {
              &:first-child {
                border-top: none;
              }
            }
          }
          &:last-child {
            .line {
              &:last-child {
                border-top: none;
              }
            }
          }
          .line {
            flex: 1;
            height: 0;
            border-top: px2rem(1) solid #ccc;
          }
          .point-wrapper {
            position: relative;
            flex: 0 0 0;
            width: 0;
            height: px2rem(7);
            border-left: px2rem(1) solid #ccc;
            .point {
              position: absolute;
              left: px2rem(-10);
              top: px2rem(-8);
              width: px2rem(20);
              height: px2rem(20);
              border-radius: 50%;
              background-color: #fff;
              border: px2rem(1) solid #ccc;
              box-shadow: 0 px2rem(4) px2rem(4) rgba(0, 0, 0, 0.15);
              @include center;
              .small-point {
                width: px2rem(5);
                height: px2rem(5);
                background-color: #000;
                border-radius: 50%;
              }
            }
          }
        }
      }
    }
    .setting-font-family {
      flex: 1;
      font-size: px2rem(14);
      @include center;
      .setting-font-family-text-wrapper {
        @include center;
      }
      .setting-font-family-icon-wrapper {
        @include center;
      }
    }
  }
</style>
