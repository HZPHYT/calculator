<template>
  <view class="container">
    <view class="input-group">
      <view class="label">输入数值</view>
      <input class="input" type="text" v-model="inputValue" placeholder="请输入数值" />
    </view>
    
    <view class="input-group">
      <view class="label">源进制</view>
      <picker mode="selector" :range="baseOptions" range-key="name" :value="fromBaseIndex" @change="onFromBaseChange">
        <view class="picker">{{ baseOptions[fromBaseIndex].name }}</view>
      </picker>
    </view>
    
    <view class="input-group">
      <view class="label">目标进制</view>
      <picker mode="selector" :range="baseOptions" range-key="name" :value="toBaseIndex" @change="onToBaseChange">
        <view class="picker">{{ baseOptions[toBaseIndex].name }}</view>
      </picker>
    </view>
    
    <view class="calculate-btn" @click="convert">转换</view>
    
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
      inputValue: '',
      fromBaseIndex: 2,
      toBaseIndex: 0,
      result: '',
      baseOptions: [
        { name: '二进制', value: 2 },
        { name: '八进制', value: 8 },
        { name: '十进制', value: 10 },
        { name: '十六进制', value: 16 }
      ]
    }
  },
  methods: {
    onFromBaseChange(e) {
      this.fromBaseIndex = e.detail.value
    },
    onToBaseChange(e) {
      this.toBaseIndex = e.detail.value
    },
    convert() {
      if (!this.inputValue) {
        uni.showToast({
          title: '请输入数值',
          icon: 'none'
        })
        return
      }
      try {
        const fromBase = this.baseOptions[this.fromBaseIndex].value
        const toBase = this.baseOptions[this.toBaseIndex].value
        
        const decimal = parseInt(this.inputValue, fromBase)
        this.result = decimal.toString(toBase)
        
        this.saveToHistory()
      } catch (e) {
        this.result = '输入格式错误'
      }
    },
    saveToHistory() {
      const history = uni.getStorageSync('baseHistory') || []
      history.unshift({
        inputValue: this.inputValue,
        fromBase: this.baseOptions[this.fromBaseIndex].name,
        toBase: this.baseOptions[this.toBaseIndex].name,
        result: this.result,
        timestamp: new Date().toISOString()
      })
      if (history.length > 10) {
        history.pop()
      }
      uni.setStorageSync('baseHistory', history)
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
  flex-direction: column;
  gap: 10rpx;
}

.result-label {
  font-size: 32rpx;
  color: #666;
}

.result-value {
  font-size: 48rpx;
  font-weight: bold;
  color: #2196f3;
  word-break: break-all;
}
</style>
