<template>
  <div id="app">
    <StyleEditor ref="styleEditor" :code="currentStyle"></StyleEditor>
    <ResumeEditor ref="resumeEditor" :markdown="currentMarkdown" :enableHtml="enableHtml"></ResumeEditor>
  </div>
</template>

<script>
import StyleEditor from './components/StyleEditor'
import ResumeEditor from './components/ResumeEditor'
import './assets/reset.css'

export default {
  name: 'app',
  components: {
    StyleEditor,
    ResumeEditor
  },
  data () {
    return {
      interval: 40,
      currentStyle: '',
      enableHtml: false,
      fullStyle: [
        `/*
* Inspired by http://strml.net/
* 大家好，我是一个不愿意说出姓名的陈姓男子
* 新的一年马上就到了
* 而我看到一个有趣的网页效果想要和你们分享。
* 参考了很多源码（其实就是抄）
*/

/* 第一步元素加上过渡效果 */
* {
  transition: all .5s;
}
/* 白色背景太单调了，我们来点背景 */
html {
  color: #005659; background: #EFEFE5
}
/* 第二步让边框离得远点 */
.styleEditor {
  padding: 1em;
  border: 1px solid;
  margin: 1em;
  overflow: auto;
  width: 45vw; height: 90vh;
}
/* 代码高亮 */
.token.selector{ color: #0CBCF2; }
.token.property{ color: #5F2711; }
.token.punctuation{ color: #5F2711; }
.token.function{ color: #A10A31; }

/* 加点特效 */
html{
  perspective: 1000px;
}
.styleEditor {
  position: fixed; left: 0; top: 0;
  -webkit-transition: none;
  transition: none;
  /* 
  *转一个？
  */
   transition: All 0.6s ease-in-out;
    -webkit-transition: All 0.6s ease-in-out;
   transform: rotate(360deg);
    -webkit-transform: rotate(360deg);
   transform: rotateY(10deg) translateZ(-100px);
    -webkit-transform: rotateY(10deg) translateZ(-100px);
         
}

/*
* 最后一步写上祝福的话 
* 先弄个框出来吧
*/
.resumeEditor{
  position: fixed; right: 0; top: 0;
  padding: .5em;  margin: .5em;
  width: 45vw; height: 90vh;
  border: 1px solid;
  background: white; color: #222;
  overflow: auto;
  transform: translate(-20px,20px);
    -webkit-transform: translate(-20px,20px);
  transform: rotateY(-10deg) translateZ(-100px);
    -webkit-transform: rotateY(-10deg) translateZ(-100px);
 
}
/* 祝大家！！ */


`,
        `
/* 这段话好像又有点难看
 * 不如
 * 调整一下
 */
`,
        `
/* 再对 HTML 加点样式 */
.resumeEditor{
  padding: 2em;
}
/* 再加个背景 */
.resumeEditor{
  background: url('../static/fu.jpg');
  background-size: 100% auto;
}
.resumeEditor h2{
  display: inline-block;
  border-bottom: 1px solid;
  margin: 1em 0 .5em;
}
.resumeEditor ul,.resumeEditor ol{
  list-style: none;
}
.resumeEditor ul> li::before{
  content: '•';
  margin-right: .5em;
}
.resumeEditor ol {
  counter-reset: section;
}
.resumeEditor ol li::before {
  counter-increment: section;
  content: counters(section, ".") " ";
  margin-right: .5em;
}
.resumeEditor blockquote {
  margin: 1em;
  padding: .5em;
  background: #ddd;
}
/* 就这样结束啦 希望能把快乐带给你*/
`],
      currentMarkdown: '',
      fullMarkdown: `大哥大嫂过年好
        
----
新的一年祝各位
----
* 所有的霉运都随着新年到来而消逝
* 出任CEO迎娶白富美
* 在新的一年事事顺遂！

祝自己
----
* 身体健康，心理已近很变态了身体可不能再有问题了
* 在前端领域有长足的进步
* 千行代码过，BUG不沾身！


链接
----

* [GitHub](https://github.com/czkm)
* [个人blog](https://czkm.github.io/)
`

    }
  },
  created () {
    this.makeResume()
  },

  methods: {
    makeResume: async function () {
      await this.progressivelyShowStyle(0)
      await this.progressivelyShowResume()
      await this.progressivelyShowStyle(1)
      await this.showHtml()
      await this.progressivelyShowStyle(2)
    },
    showHtml: function () {
      return new Promise((resolve, reject) => {
        this.enableHtml = true
        resolve()
      })
    },
    progressivelyShowStyle (n) {
      return new Promise((resolve, reject) => {
        let interval = this.interval
        let showStyle = async function () {
          let style = this.fullStyle[n]
          if (!style) { return }
          // 计算前 n 个 style 的字符总数
          let length = this.fullStyle.filter((_, index) => index <= n).map((item) => item.length).reduce((p, c) => p + c, 0)
          let prefixLength = length - style.length
          if (this.currentStyle.length < length) {
            let l = this.currentStyle.length - prefixLength
            let char = style.substring(l, l + 1) || ' '
            this.currentStyle += char
            if (style.substring(l - 1, l) === '\n' && this.$refs.styleEditor) {
              this.$nextTick(() => {
                this.$refs.styleEditor.goBottom()
              })
            }
            setTimeout(showStyle, interval)
          } else {
            resolve()
          }
        }.bind(this)
        showStyle()
      })
    },
    progressivelyShowResume () {
      return new Promise((resolve, reject) => {
        let length = this.fullMarkdown.length
        let interval = this.interval
        let showResume = () => {
          if (this.currentMarkdown.length < length) {
            this.currentMarkdown = this.fullMarkdown.substring(0, this.currentMarkdown.length + 1)
            // eslint-disable-next-line no-unused-vars
            let lastChar = this.currentMarkdown[this.currentMarkdown.length - 1]
            let prevChar = this.currentMarkdown[this.currentMarkdown.length - 2]
            if (prevChar === '\n' && this.$refs.resumeEditor) {
              this.$nextTick(() => this.$refs.resumeEditor.goBottom())
            }
            setTimeout(showResume, interval)
          } else {
            resolve()
          }
        }
        showResume()
      })
    }
  }
}

</script>

<style scoped>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }

  html {
    min-height: 100vh;
  }
  *{
    box-sizing: border-box;
  }
</style>
