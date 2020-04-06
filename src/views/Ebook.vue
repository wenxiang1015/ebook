<template>
  <div class="ebook">
    <title-bar :ifTitleAndMenuShow="ifTitleAndMenuShow"></title-bar>
    <div class="read-warpper">
      <div id="read"></div>
      <div class="mask">
        <div class="left" @click="prevPage"></div>
        <div class="center" @click="toggleTitleAndMenu"></div>
        <div class="right" @click="nextPage"></div>
      </div>
    </div>
    <menu-bar
      :ifTitleAndMenuShow="ifTitleAndMenuShow"
      :selectedFontSize="selectedFontSize"
      @setFontSize="setFontSize"
      :themeList="themeList"
      :selectedThemeIndex="selectedThemeIndex"
      @setTheme="setTheme"
      :bookAvailable="bookAvailable"
      @onProgressChange="onProgressChange"
      :navigation="navigation"
      @jumpTo="jumpTo"
      ref="menuBar"
    ></menu-bar>
  </div>
</template>

<script type="text/ecmascript-6" >
import TitleBar from "@/components/TitleBar";
import MenuBar from "@/components/MenuBar";
import epub from "epubjs";

export default {
  components: {
    TitleBar,
    MenuBar
  },
  data() {
    return {
      ifTitleAndMenuShow: false,
      selectedFontSize: 16,
      selectedThemeIndex: 0,
      themeList: [
        {
          name: "default",
          style: {
            body: {
              color: "#000",
              background: "#fff"
            }
          }
        },
        {
          name: "eye",
          style: {
            body: {
              color: "#000",
              background: "#ceeaba"
            }
          }
        },
        {
          name: "night",
          style: {
            body: {
              color: "#fff",
              background: "#000"
            }
          }
        },
        {
          name: "gold",
          style: {
            body: {
              color: "#000",
              background: "rgb(241, 236, 226)"
            }
          }
        }
      ],
      bookAvailable: false, //epub 的 locations 对象是否加载完毕
      navigation: {}
    };
  },
  methods: {
    // 根据链接(目录)跳转到指定位置
    jumpTo(href) {
      this.rendition.display(href);
      this.ifTitleAndMenuShow = false;
      this.$refs.menuBar.hideSetting();
      this.$refs.menuBar.hideContent();
    },
    // progress 进度条的数值（0-100）
    onProgressChange(progress) {
      const percentage = progress / 100;
      const location = this.locations.cfiFromPercentage(percentage);
      this.rendition.display(location);
    },
    setFontSize(fontSize) {
      this.selectedFontSize = fontSize;
      if (this.themes) {
        this.themes.fontSize(fontSize + "px");
      }
    },
    setTheme(index) {
      this.selectedThemeIndex = index;
      this.themes.select(this.themeList[index].name);
    },
    // 切换标题和菜单的显示状态
    toggleTitleAndMenu() {
      this.ifTitleAndMenuShow = !this.ifTitleAndMenuShow;
      if (!this.ifTitleAndMenuShow) {
        this.$refs.menuBar.hideSetting();
      }
    },
    prevPage() {
      if (this.rendition) {
        this.rendition.prev();
      }
    },
    nextPage() {
      if (this.rendition) {
        this.rendition.next();
      }
    },
    // 电子书的解析和渲染
    async showEpub() {
      const URL = `${process.env.BASE_URL}ebook/惜别.epub`;
      this.book = await epub(URL, {});
      this.rendition = this.book.renderTo("read", {
        width: window.innerWidth,
        height: window.innerHeight
      });
      this.rendition.display();
      this.themes = this.rendition.themes;
      this.setFontSize(this.selectedFontSize);
      // 注册主题
      this.themeList.forEach(theme => {
        this.themes.register(theme.name, theme.style);
      });
      this.setTheme(this.selectedThemeIndex);
      // Book对象的钩子函数ready
      this.book.ready
        .then(() => {
          this.navigation = this.book.navigation;
          // 生成Locations对象
          return this.book.locations.generate();
        })
        .then(result => {
          console.log(result);
          // 保存locations对象
          this.locations = this.book.locations;
          // 标记电子书为解析完毕状态
          this.bookAvailable = true;
        });
    }
  },
  mounted() {
    this.showEpub();
  }
};
</script>
<style lang='scss' rel="stylesheet/scss" scoped>
@import "../assets/styles/global";

.ebook {
  position: relative;
  .read-warpper {
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
