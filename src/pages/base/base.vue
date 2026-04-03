<template>
  <view class="container">
    <view class="header-row">
      <view class="history-btn" @click="toggleHistory">历史</view>
    </view>

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
          <view class="history-data">{{ item.inputValue }} ({{ item.fromBase }} → {{ item.toBase }})</view>
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
      inputValue: '',
      fromBaseIndex: 2,
      toBaseIndex: 0,
      result: '',
      showHistory: false,
      history: [],
      baseOptions: [
        { name: '二进制', value: 2 },
        { name: '八进制', value: 8 },
        { name: '十进制', value: 10 },
        { name: '十六进制', value: 16 }
      ]
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
      this.history = uni.getStorageSync('baseHistory') || []
    },
    clearHistory() {
      this.vibrate()
      uni.removeStorageSync('baseHistory')
      this.history = []
    },
    restoreHistory(item) {
      this.vibrate()
      this.inputValue = item.inputValue
      this.result = item.result
      for (let i = 0; i < this.baseOptions.length; i++) {
        if (this.baseOptions[i].name === item.fromBase) this.fromBaseIndex = i
        if (this.baseOptions[i].name === item.toBase) this.toBaseIndex = i
      }
      this.showHistory = false
    },
    copyResult() {
      if (this.result && this.result !== '输入格式错误') {
        uni.setClipboardData({
          data: this.result,
          success: () => {
            uni.showToast({ title: '已复制', icon: 'success' })
          }
        })
      }
    },
    onFromBaseChange(e) {
      this.fromBaseIndex = e.detail.value
    },
    onToBaseChange(e) {
      this.toBaseIndex = e.detail.value
    },
    convert() {
      if (!this.inputValue) {
        uni.showToast({ title: '请输入数值', icon: 'none' })
        return
      }
      this.vibrate()
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
      if (history.length > 20) {
        history.pop()
      }
      uni.setStorageSync('baseHistory', history)
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
  color: #607d8b;
  padding: 10rpx 24rpx;
  background: rgba(96, 125, 139, 0.1);
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
  color: #607d8b;
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
  color: #607d8b;
}

.history-empty {
  text-align: center;
  color: #999;
  padding: 60rpx;
  font-size: 28rpx;
}
</style>
