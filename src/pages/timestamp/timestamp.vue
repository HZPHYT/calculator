<template>
  <view class="container">
    <view class="header-row">
      <view class="history-btn" @click="toggleHistory">历史</view>
    </view>

    <view class="input-group">
      <view class="label">时间戳 (秒)</view>
      <input class="input" type="number" v-model="timestamp" placeholder="请输入时间戳" />
    </view>
    
    <view class="input-group">
      <view class="label">日期时间</view>
      <picker mode="multiSelector" :range="dateTimeRange" :value="dateTimeValue" @change="onDateTimeChange" @columnchange="onColumnChange">
        <view class="picker">{{ datetime || '请选择日期时间' }}</view>
      </picker>
    </view>
    
    <view class="button-group">
      <view class="convert-btn" @click="timestampToDatetime">时间戳 → 日期</view>
      <view class="convert-btn" @click="datetimeToTimestamp">日期 → 时间戳</view>
    </view>
    
    <view class="result-container">
      <view class="result-item">
        <view class="result-label">转换结果：</view>
        <view class="result-value" @click="copyResult">{{ result }}</view>
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
          <view class="history-data">{{ item.inputType }}: {{ item.input }}</view>
          <view class="history-result">{{ item.result }}</view>
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
      timestamp: '',
      datetime: '',
      result: '',
      showHistory: false,
      history: [],
      dateTimeRange: [[], [], [], [], []],
      dateTimeValue: [0, 0, 0, 0, 0]
    }
  },
  onLoad() {
    this.initDateTimeRange()
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
      this.history = uni.getStorageSync('timestampHistory') || []
    },
    clearHistory() {
      this.vibrate()
      uni.removeStorageSync('timestampHistory')
      this.history = []
    },
    restoreHistory(item) {
      this.vibrate()
      if (item.inputType === '时间戳') {
        this.timestamp = item.input
      } else {
        this.datetime = item.input
        this.result = item.result
      }
      this.showHistory = false
    },
    copyResult() {
      if (this.result) {
        uni.setClipboardData({
          data: this.result,
          success: () => {
            uni.showToast({ title: '已复制', icon: 'success' })
          }
        })
      }
    },
    initDateTimeRange() {
      const years = []
      const months = []
      const days = []
      const hours = []
      const minutes = []
      
      for (let i = 1970; i <= 2100; i++) {
        years.push(i + '年')
      }
      for (let i = 1; i <= 12; i++) {
        months.push(i + '月')
      }
      for (let i = 1; i <= 31; i++) {
        days.push(i + '日')
      }
      for (let i = 0; i < 24; i++) {
        hours.push(i + '时')
      }
      for (let i = 0; i < 60; i++) {
        minutes.push(i + '分')
      }
      
      this.dateTimeRange = [years, months, days, hours, minutes]
    },
    onDateTimeChange(e) {
      const val = e.detail.value
      this.dateTimeValue = val
      this.datetime = this.dateTimeRange[0][val[0]] + 
                     this.dateTimeRange[1][val[1]] + 
                     this.dateTimeRange[2][val[2]] + 
                     this.dateTimeRange[3][val[3]] + 
                     this.dateTimeRange[4][val[4]]
    },
    onColumnChange(e) {
    },
    timestampToDatetime() {
      if (!this.timestamp) {
        uni.showToast({ title: '请输入时间戳', icon: 'none' })
        return
      }
      const date = new Date(this.timestamp * 1000)
      const year = date.getFullYear()
      const month = date.getMonth() + 1
      const day = date.getDate()
      const hour = date.getHours()
      const minute = date.getMinutes()
      const second = date.getSeconds()
      this.result = year + '年' + month + '月' + day + '日' + hour + '时' + minute + '分' + second + '秒'
      this.saveToHistory('时间戳', this.timestamp)
    },
    datetimeToTimestamp() {
      if (!this.datetime) {
        uni.showToast({ title: '请选择日期时间', icon: 'none' })
        return
      }
      const year = parseInt(this.dateTimeRange[0][this.dateTimeValue[0]])
      const month = parseInt(this.dateTimeRange[1][this.dateTimeValue[1]])
      const day = parseInt(this.dateTimeRange[2][this.dateTimeValue[2]])
      const hour = parseInt(this.dateTimeRange[3][this.dateTimeValue[3]])
      const minute = parseInt(this.dateTimeRange[4][this.dateTimeValue[4]])
      
      const date = new Date(year, month - 1, day, hour, minute)
      const timestamp = Math.floor(date.getTime() / 1000)
      this.result = timestamp.toString()
      this.saveToHistory('日期', this.datetime)
    },
    saveToHistory(inputType, input) {
      const history = uni.getStorageSync('timestampHistory') || []
      history.unshift({
        inputType,
        input,
        result: this.result,
        timestamp: new Date().toISOString()
      })
      if (history.length > 20) {
        history.pop()
      }
      uni.setStorageSync('timestampHistory', history)
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
  color: #00bcd4;
  padding: 10rpx 24rpx;
  background: rgba(0, 188, 212, 0.1);
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

.input {
  width: 100%;
  padding: 24rpx 32rpx;
  background: rgba(255, 255, 255, 0.8);
  border-radius: 20rpx;
  font-size: 32rpx;
  color: #333;
  box-shadow: 0 4rpx 16rpx rgba(0, 0, 0, 0.1);
}

.picker {
  padding: 24rpx 32rpx;
  background: rgba(255, 255, 255, 0.8);
  border-radius: 20rpx;
  font-size: 32rpx;
  color: #333;
  box-shadow: 0 4rpx 16rpx rgba(0, 0, 0, 0.1);
}

.button-group {
  display: flex;
  gap: 20rpx;
  margin-top: 40rpx;
}

.convert-btn {
  flex: 1;
  padding: 24rpx;
  background: #2196f3;
  color: white;
  font-size: 28rpx;
  font-weight: 500;
  text-align: center;
  border-radius: 20rpx;
  box-shadow: 0 8rpx 24rpx rgba(0, 0, 0, 0.1);
}

.convert-btn:active {
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
  flex-direction: column;
  gap: 10rpx;
}

.result-label {
  font-size: 32rpx;
  color: #666;
}

.result-value {
  font-size: 40rpx;
  font-weight: bold;
  color: #00bcd4;
  word-break: break-all;
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
  color: #00bcd4;
}

.history-empty {
  text-align: center;
  color: #999;
  padding: 60rpx;
  font-size: 28rpx;
}
</style>
