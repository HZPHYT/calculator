<template>
  <view class="container">
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
        <view class="result-value">{{ result }}</view>
      </view>
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
      dateTimeRange: [[], [], [], [], []],
      dateTimeValue: [0, 0, 0, 0, 0]
    }
  },
  onLoad() {
    this.initDateTimeRange()
  },
  methods: {
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
      // 处理月份变化时更新天数
    },
    timestampToDatetime() {
      if (!this.timestamp) {
        uni.showToast({
          title: '请输入时间戳',
          icon: 'none'
        })
        return
      }
      const date = new Date(this.timestamp * 1000)
      this.result = date.toLocaleString()
      this.saveToHistory()
    },
    datetimeToTimestamp() {
      if (!this.datetime) {
        uni.showToast({
          title: '请选择日期时间',
          icon: 'none'
        })
        return
      }
      const year = parseInt(this.dateTimeRange[0][this.dateTimeValue[0]])
      const month = parseInt(this.dateTimeRange[1][this.dateTimeValue[1]])
      const day = parseInt(this.dateTimeRange[2][this.dateTimeValue[2]])
      const hour = parseInt(this.dateTimeRange[3][this.dateTimeValue[3]])
      const minute = parseInt(this.dateTimeRange[4][this.dateTimeValue[4]])
      
      const date = new Date(year, month - 1, day, hour, minute)
      const timestamp = Math.floor(date.getTime() / 1000)
      this.timestamp = timestamp.toString()
      this.result = timestamp.toString()
      this.saveToHistory()
    },
    saveToHistory() {
      const history = uni.getStorageSync('timestampHistory') || []
      history.unshift({
        timestamp: this.timestamp,
        datetime: this.datetime,
        result: this.result,
        timestamp: new Date().toISOString()
      })
      if (history.length > 10) {
        history.pop()
      }
      uni.setStorageSync('timestampHistory', history)
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
  color: #2196f3;
  word-break: break-all;
}
</style>
