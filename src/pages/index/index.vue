<template>
  <view class="content">
    <image class="logo" src="/static/logo.png" @click="onclick_startQuiz"></image>
    <uni-title title="二十大答题小程序" type="h2" />
    <uni-title title="- 中国科大本科生院 -" type="h6" />
    <button type="primary" class="start-button" @click="onclick_startQuiz">开始答题</button>
    <button type="normal" class="start-button" @click="onclick_showResult">查看结果</button>
    <button type="warn" class="start-button" @click="onclick_clearHistory">清空本地记录</button>
  </view>
</template>

<script>
export default {
  data() {
    return {
      title: '点击上方进入问卷',
    }
  },
  onLoad() { },
  methods: {
    onclick_startQuiz() {
      uni.navigateTo({
        url: '/pages/quiz/quiz?id=null',
      })
    },
    onclick_clearHistory() {
      uni.showModal({
        title: '确认清空答题历史?', content: '清空后无法恢复',
        success: function (res) {
          if (res.confirm) {
            uni.clearStorage()
            uni.setStorageSync('answers', '[]')
            uni.showToast({ title: '清空成功', icon: 'success', duration: 2000 })
          } else if (res.cancel) {
            uni.showToast({ title: '取消清空', icon: 'none', duration: 2000 })
          }
        }
      });
    },
    onclick_showResult() {
      uni.navigateTo({
        url: '/pages/result/result',
      })
    }
  },
}
</script>

<style>
.content {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.logo {
  height: 200rpx;
  width: 200rpx;
  margin-top: 200rpx;
  margin-left: auto;
  margin-right: auto;
  margin-bottom: 50rpx;
}

.text-area {
  display: flex;
  justify-content: center;
}

.start-button {
  /* width: 80%; */
  /* height: 100rpx; */
  margin: 20rpx;
  width: 250rpx;
  font-size: 30rpx;
}

.title {
  font-size: 36rpx;
  color: #8f8f94;
}
</style>
