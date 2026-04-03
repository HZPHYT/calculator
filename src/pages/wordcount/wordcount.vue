<template>
  <view class="container">
    <view class="header-row">
      <view class="history-btn" @click="toggleHistory">历史</view>
    </view>

    <view class="input-group">
      <view class="label">输入文本</view>
      <textarea class="textarea" v-model="text" placeholder="请输入文本" />
    </view>

    <view class="save-btn" @click="saveToHistory">保存统计</view>
    
    <view class="result-container">
      <view class="result-item">
        <view class="result-label">字符数（含换行符）：</view>
        <view class="result-value" @click="copyResult('charCount')">{{ charCount }}</view>
      </view>
      <view class="result-item">
        <view class="result-label">行数：</view>
        <view class="result-value" @click="copyResult('lineCount')">{{ lineCount }}</view>
      </view>
    </view>
    <view class="copy-tip">点击结果可复制</view>
    <view class="save-tip">保存后可从历史记录中查看</view>

    <view class="history-panel" v-if="showHistory">
      <view class="history-header">
        <text>历史记录</text>
        <text class="clear-btn" @click="clearHistory">清空</text>
      </view>
      <scroll-view class="history-list" scroll-y>
        <view class="history-item" v-for="(item, index) in history" :key="index" @click="restoreHistory(item)">
          <view class="history-data">{{ item.text.slice(0, 20) }}{{ item.text.length > 20 ? '...' : '' }}</view>
          <view class="history-result">字符: {{ item.charCount }} / 字: {{ item.wordCount }}</view>
        </view>
        <view class="history-empty" v-if="history.length === 0">暂无历史记录</view>
      </scroll-view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      text: '',
      showHistory: false,
      history: []
    }
  },
  computed: {
    charCount() {
      return this.text.length
    },
    wordCount() {
      return this.text.replace(/\s/g, '').length
    },
    wordCountWithSpace() {
      return this.text.length
    },
    lineCount() {
      if (!this.text) return 0
      return this.text.split('\n').length
    }
  },
  onShow() {
    this.loadHistory()
  },
  methods: {
    vibrate() {},
    toggleHistory() {
      this.vibrate()
      this.showHistory = !this.showHistory
    },
    loadHistory() {
      this.history = uni.getStorageSync('wordcountHistory') || []
    },
    clearHistory() {
      this.vibrate()
      uni.removeStorageSync('wordcountHistory')
      this.history = []
    },
    restoreHistory(item) {
      this.vibrate()
      this.text = item.text
      this.showHistory = false
    },
    copyResult(type) {
      let data = ''
      if (type === 'charCount') data = this.charCount.toString()
      else if (type === 'wordCount') data = this.wordCount.toString()
      else if (type === 'lineCount') data = this.lineCount.toString()
      
      if (data) {
        uni.setClipboardData({
          data,
          success: () => {
            uni.showToast({ title: '已复制', icon: 'success' })
          }
        })
      }
    },
    saveToHistory() {
      if (!this.text) {
        uni.showToast({ title: '请输入文本', icon: 'none' })
        return
      }
      this.vibrate()
      const history = uni.getStorageSync('wordcountHistory') || []
      history.unshift({
        text: this.text,
        charCount: this.charCount,
        wordCount: this.wordCount,
        lineCount: this.lineCount,
        timestamp: new Date().toISOString()
      })
      if (history.length > 20) {
        history.pop()
      }
      uni.setStorageSync('wordcountHistory', history)
      this.history = history
      uni.showToast({ title: '已保存', icon: 'success' })
    }
  }
}
</script>

<style lang="scss" scoped>
.container {
  min-height: 100vh;
  background: linear-gradient(135deg, #e3f2fd 0%, #bbdefb 100%);
  padding: 40rpx;
}

.header-row {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 20rpx;
}

.history-btn {
  font-size: 28rpx;
  color: #8bc34a;
  padding: 10rpx 24rpx;
  background: rgba(139, 195, 74, 0.1);
  border-radius: 20rpx;
}

.input-group {
  margin-bottom: 40rpx;
}

.label {
  font-size: 32rpx;
  font-weight: 500;
  color: #333;
  margin-bottom: 20rpx;
}

.textarea {
  width: 100%;
  min-height: 400rpx;
  padding: 24rpx 32rpx;
  background: rgba(255, 255, 255, 0.8);
  border-radius: 20rpx;
  font-size: 32rpx;
  color: #333;
  box-shadow: 0 4rpx 16rpx rgba(0, 0, 0, 0.1);
}

.save-btn {
  width: 100%;
  padding: 24rpx;
  background: #8bc34a;
  color: white;
  font-size: 32rpx;
  font-weight: 500;
  text-align: center;
  border-radius: 20rpx;
  margin-bottom: 30rpx;
  box-shadow: 0 8rpx 24rpx rgba(0, 0, 0, 0.1);
}

.save-btn:active {
  transform: scale(0.98);
}

.result-container {
  margin-top: 40rpx;
  padding: 40rpx;
  background: rgba(255, 255, 255, 0.7);
  border-radius: 30rpx;
  box-shadow: 0 8rpx 24rpx rgba(0, 0, 0, 0.1);
}

.result-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30rpx;
}

.result-label {
  font-size: 32rpx;
  color: #666;
}

.result-value {
  font-size: 40rpx;
  font-weight: bold;
  color: #8bc34a;
}

.copy-tip {
  text-align: center;
  font-size: 24rpx;
  color: #999;
  margin-top: 10rpx;
}

.save-tip {
  text-align: center;
  font-size: 24rpx;
  color: #999;
  margin-top: 5rpx;
}

.history-panel {
  position: fixed;
  top: 0;
  right: 0;
  width: 70%;
  height: 100vh;
  background: white;
  box-shadow: -4rpx 0 24rpx rgba(0, 0, 0, 0.1);
  z-index: 999;
  display: flex;
  flex-direction: column;
}

.history-header {
  padding: 40rpx;
  border-bottom: 1px solid #eee;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 32rpx;
  font-weight: bold;
  color: #333;
}

.clear-btn {
  color: #f44336;
  font-size: 28rpx;
}

.history-list {
  flex: 1;
  padding: 20rpx;
}

.history-item {
  padding: 30rpx;
  border-bottom: 1px solid #f0f0f0;
}

.history-data {
  font-size: 28rpx;
  color: #666;
  margin-bottom: 10rpx;
  word-break: break-all;
}

.history-result {
  font-size: 32rpx;
  font-weight: bold;
  color: #8bc34a;
}

.history-empty {
  text-align: center;
  color: #999;
  padding: 60rpx;
  font-size: 28rpx;
}
</style>
