<template>
  <section class="content">
    <div class="title">{{inputing ? 'Ta正在输入' : '天下武学总汇'}}</div>
    <div class="talk" v-show="talkShow" @click="talkShow = false">
      <ul :class="{active: talkShow}">
        <li v-for="(item, index) in options" :key="index" @click="talkData(item)">{{item.value}}</li>
      </ul>
    </div>
    <ul class="chat-list" ref="chat">
      <li v-for="(item, index) in talkArr" :class="item.uid ? item.uid + ' left' : 'right'" :key="index">
        <i v-if="item.uid" class="label">【{{item.label}}】 —— {{item.mark}}</i>
        <span :style="isImg(item.value) ? 'padding: 0;font-size: 0;' : ''" v-html="contentFilter(item.value)"></span>
      </li>
      <li class="left wish" v-show="inputing">
        <span>
          <i class="dot"></i>
          <i class="dot"></i>
          <i class="dot"></i>
        </span>
      </li>
    </ul>
    <div class="input" @click="saySome">{{inputing ? '对方打字中……' : '说点什么吧...'}}</div>
   <textarea id="link" class="link"></textarea>
    <div class="restartAsk" v-show="restart">
      <section>
        <span>欢迎，检测到记录</span>
        <span v-show="scenes">载入分享场景 ?</span>
        <span v-show="anchor">从上次的场景继续 ?</span>
        <button v-show="scenes" type="button" @click="start(1, scenes)">载入分享场景</button>
        <button v-show="anchor" type="button" @click="start(2, anchor)">进入上回进度</button>
        <button type="button" @click="start(0, null)">重新开始</button>
      </section>
    </div>
  </section>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'
const dataJSON: any = require('@/assets/data').default

const initKey = '0000'

@Component
export default class Main extends Vue {
  // data
  private talkShow: boolean = false
  private talkArr: any = []
  private options: any = []
  private inputing: boolean = false
  private restart: boolean = false
  private key: string | null = ''
  private scenes: any = ''
  private anchor: any = ''
  private toastStatus = false

  private mounted() {
    const host = window.location.href
    const msgObj = host.split('?')[1]
    const queryArr = msgObj ? msgObj.split('&') : []
    const data: any = {}
    queryArr.forEach(item => {
      const arr: any = item.split('=')
      data[arr[0]] = arr[1]
    })
    this.anchor = window.localStorage.getItem('#kungfu')
    this.inputing = true
    if (data.anchor) {
      // 如果链接中包含分享key则从分享场景开始
      this.scenes = data.anchor
      this.restart = true
    } else {
      this.key = window.localStorage.getItem('#kungfu')
      if (this.key) {
        // 询问是否重新开始
        this.restart = true
      } else {
        this.awswerList(initKey)
      }
    }
  }

  // methods
  private saySome(): void {
    !this.inputing && (this.talkShow = true)
  }

  private start(n: number, key: any): void {
    this.restart = false
    !!key && (this.key = key)
    if (n) {
      // 继续
      this.awswerList(this.key)
    } else {
      // 重新开始
      this.awswerList(initKey)
    }
  }

  private talkData(data: any): void {
    this.talkShow = false
    this.inputing = true // 输入状态

    // 提问处理
    this.talkArr.push({
      user: null,
      value: data.value
    })
    this.updateScroll()
    this.awswerList(data.next)
  }

  private awswerList(data: any) {
    // 回答处理
    window.localStorage.setItem('#kungfu', data)

    this.setLink(data)

    if (dataJSON[data]) {
      this.options = dataJSON[data].req

      const resData = dataJSON[data].res
      const len = resData.length - 1
      let i = 0
      const timer = setInterval(() => {
        this.pushData(resData[i])
        if (i >= len) {
          clearInterval(timer)
          this.inputing = false // 非输入状态
        }
        this.updateScroll()
        i++
      }, 2500)
    } else {
      // 找不到key
      this.awswerList(initKey)
    }
  }

  private pushData(data: any): void {
    this.talkArr.push(data)
  }

  private setLink(data: string): void {
    // 生成拼接链接
    const state = {
      title: '',
      url: window.location.href.split('?')[0]
    }
    history.pushState(state, '', `?anchor=${data}`)
  }

  private contentFilter(value: string): string {
    const mark = value.split(':')[0]
    switch (mark) {
      case 'img':
        // 添加图片显示功能
        const img = value.substr(4)
        return `<img src="${img}" style="width: 100%;height: auto;" />`
      case 'url':
        // 添加链接显示功能
        const a = value.substr(4)
        return `<a href=${a} style="font-size: 12px;color: #fffffb">${a}</a>`
      default:
        return value
    }
  }

  private isImg(value: string): boolean {
    const mark = value.split(':')[0]
    return mark === 'img'
  }

  private updateScroll(): void {
    const $dom: any = this.$refs.chat
    if (!$dom.scrollHeight) {
      return
    } else {
      const distance = $dom.scrollHeight - $dom.offsetHeight
      const duration = 250
      const startTime = Date.now()

      requestAnimationFrame(function step() {
        const p = Math.min(1, (Date.now() - startTime) / duration)
        $dom.scrollTop = $dom.scrollTop + distance * p
        p < 1 && requestAnimationFrame(step)
      })
    }
  }
}
</script>

<style scoped lang="less">
@themeColor: #7b90d2;
@import '../assets/styles/chat.less';
@import '../assets/styles/peopleChat.less';
@import '../assets/styles/animate.less';
@import '../assets/styles/restartAsk.less';

.content {
  margin: auto;
  height: 100vh;
  width: 100%;
  max-width: 480px;
  box-sizing: border-box;
  position: relative;
  // background: #7db9de; // 勿忘草
  background: #fffffb;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  background-image: url('../assets/images/texture.png');
  .title {
    background: linear-gradient(to right, #7b90d2, #2ea9df, #7b90d2);
    color: #fff;
    height: 45px;
    line-height: 45px;
    padding: 0 10px;
    box-shadow: 0 0 5px #f1f1f1;
    border-radius: 2px;
    text-align: center;
    position: relative;
    font-weight: 800;
    font-size: 16px;
  }
  .link {
    width: 0;
    height: 0;
    overflow: hidden;
    opacity: 0;
    border: none;
    position: absolute;
    z-index: -10;
  }
  .talk {
    position: absolute;
    left: 0;
    right: 0;
    top: 0;
    bottom: 50px;
    background: rgba(0, 0, 0, 0.4);
    z-index: 15;
    ul {
      position: absolute;
      bottom: 0;
      background: #fff;
      padding: 0;
      margin: 0;
      width: 100%;
      transform: translateY(100%);
      transition: 0.3s all ease;
      &.active {
        transform: translateY(0);
        transition: 0.3s all ease;
      }
      li {
        height: 44px;
        line-height: 44px;
        padding: 0 15px;
        margin: 0;
        font-size: 14px;
        list-style: none;
        color: #2760bd;
        display: block;
        position: relative;
        &:after {
          display: block;
          position: absolute;
          bottom: 0;
          content: '';
          bottom: 0;
          height: 1px;
          left: 10px;
          right: 10px;
          background: #e8e8e8;
        }
      }
    }
  }
  .input {
    background: #fff;
    height: 50px;
    line-height: 50px;
    padding: 0 10px;
    box-sizing: border-box;
    color: #888;
    font-size: 13px;
    box-shadow: 0 0 3px @themeColor;
    border-top-left-radius: 4px;
    border-top-right-radius: 4px;
  }
}
</style>
