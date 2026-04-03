<template>
  <view class="container">
    <view class="header-row">
      <view class="history-btn" @click="toggleHistory">历史</view>
    </view>

    <view class="input-group">
      <view class="label">开始日期</view>
      <picker mode="date" :value="startDate" @change="onStartDateChange">
        <view class="picker">{{ startDate || '请选择开始日期' }}</view>
      </picker>
    </view>
    
    <view class="input-group">
      <view class="label">结束日期</view>
      <picker mode="date" :value="endDate" @change="onEndDateChange">
        <view class="picker">{{ endDate || '请选择结束日期' }}</view>
      </picker>
    </view>
    
    <view class="calculate-btn" @click="calculateDays">计算天数</view>
    
    <view class="result-container">
      <view class="result-item">
        <view class="result-label">天数（含首尾）：</view>
        <view class="result-value" @click="copyResult">{{ result }} 天</view>
      </view>
    </view>
    <view class="copy-tip">点击结果可复制</view>

    <view class="history-panel" v-if="showHistory">
      <view class="history-header">
        <text>历史记录</text>
        <text class="clear-btn" @click="clearHistory">清空</text>
      </view>
      <scroll-view class="history-list" scroll-y>
        <view class="history-item" v-for="(item, index) in history" :key="index" @click="restoreHistory(item)">
          <view class="history-data">{{ item.startDate }} ~ {{ item.endDate }}</view>
          <view class="history-result">{{ item.result }} 天</view>
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
      startDate: '',
      endDate: '',
      result: 0,
      showHistory: false,
      history: []
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
      this.history = uni.getStorageSync('dateHistory') || []
    },
    clearHistory() {
      this.vibrate()
      uni.removeStorageSync('dateHistory')
      this.history = []
    },
    restoreHistory(item) {
      this.vibrate()
      this.startDate = item.startDate
      this.endDate = item.endDate
      this.result = item.result
      this.showHistory = false
    },
    copyResult() {
      if (this.result) {
        uni.setClipboardData({
          data: this.result.toString(),
          success: () => {
            uni.showToast({ title: '已复制', icon: 'success' })
          }
        })
      }
    },
    onStartDateChange(e) {
      this.startDate = e.detail.value
    },
    onEndDateChange(e) {
      this.endDate = e.detail.value
    },
    calculateDays() {
      if (!this.startDate || !this.endDate) {
        uni.showToast({ title: '请选择日期', icon: 'none' })
        return
      }
      this.vibrate()
      const start = new Date(this.startDate)
      const end = new Date(this.endDate)
      const diffDays = Math.abs(end - start) / (1000 * 60 * 60 * 24) + 1
      this.result = diffDays
      this.saveToHistory()
    },
    saveToHistory() {
      const history = uni.getStorageSync('dateHistory') || []
      history.unshift({
        startDate: this.startDate,
        endDate: this.endDate,
        result: this.result,
        timestamp: new Date().toISOString()
      })
      if (history.length > 20) {
        history.pop()
      }
      uni.setStorageSync('dateHistory', history)
      this.history = history
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
  color: #e91e63;
  padding: 10rpx 24rpx;
  background: rgba(233, 30, 99, 0.1);
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

.picker {
  padding: 24rpx 32rpx;
  background: rgba(255, 255, 255, 0.8);
  border-radius: 20rpx;
  font-size: 32rpx;
  color: #333;
  box-shadow: 0 4rpx 16rpx rgba(0, 0, 0, 0.1);
}

.calculate-btn {
  width: 100%;
  padding: 30rpx;
  background: #2196f3;
  color: white;
  font-size: 32rpx;
  font-weight: 500;
  text-align: center;
  border-radius: 20rpx;
  margin-top: 40rpx;
  box-shadow: 0 8rpx 24rpx rgba(0, 0, 0, 0.1);
}

.calculate-btn:active {
  transform: scale(0.98);
}

.result-container {
  margin-top: 60rpx;
  padding: 40rpx;
  background: rgba(255, 255, 255, 0.7);
  border-radius: 30rpx;
  box-shadow: 0 8rpx 24rpx rgba(0, 0, 0, 0.1);
}

.result-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.result-label {
  font-size: 32rpx;
  color: #666;
}

.result-value {
  font-size: 48rpx;
  font-weight: bold;
  color: #e91e63;
}

.copy-tip {
  text-align: center;
  font-size: 24rpx;
  color: #999;
  margin-top: 10rpx;
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
}

.history-result {
  font-size: 36rpx;
  font-weight: bold;
  color: #e91e63;
}

.history-empty {
  text-align: center;
  color: #999;
  padding: 60rpx;
  font-size: 28rpx;
}
</style>
