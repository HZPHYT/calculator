<template>
  <view class="container">
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
        <view class="result-label">天数差：</view>
        <view class="result-value">{{ result }} 天</view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      startDate: '',
      endDate: '',
      result: 0
    }
  },
  methods: {
    onStartDateChange(e) {
      this.startDate = e.detail.value
    },
    onEndDateChange(e) {
      this.endDate = e.detail.value
    },
    calculateDays() {
      if (!this.startDate || !this.endDate) {
        uni.showToast({
          title: '请选择日期',
          icon: 'none'
        })
        return
      }
      const start = new Date(this.startDate)
      const end = new Date(this.endDate)
      const diffTime = Math.abs(end - start)
      const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24))
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
      if (history.length > 10) {
        history.pop()
      }
      uni.setStorageSync('dateHistory', history)
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
  color: #2196f3;
}
</style>
