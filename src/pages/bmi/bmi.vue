<template>
  <view class="container">
    <view class="header-row">
      <view class="history-btn" @click="toggleHistory">历史</view>
    </view>

    <view class="input-group">
      <view class="label">身高 (cm)</view>
      <input class="input" type="number" v-model="height" placeholder="请输入身高" />
    </view>
    
    <view class="input-group">
      <view class="label">体重 (kg)</view>
      <input class="input" type="number" v-model="weight" placeholder="请输入体重" />
    </view>
    
    <view class="calculate-btn" @click="calculateBMI">计算 BMI</view>
    
    <view class="result-container">
      <view class="result-item">
        <view class="result-label">BMI 值：</view>
        <view class="result-value" @click="copyResult">{{ bmiValue }}</view>
      </view>
      <view class="result-item">
        <view class="result-label">健康状况：</view>
        <view class="result-value" :class="statusClass">{{ healthStatus }}</view>
      </view>
    </view>

    <view class="formula-tip">
      <view class="formula-title">BMI 计算公式：</view>
      <view class="formula-content">BMI = 体重(kg) ÷ 身高(m)²</view>
      <view class="formula-title" style="margin-top: 16rpx;">参考标准：</view>
      <view class="formula-content">偏瘦：BMI &lt; 18.5</view>
      <view class="formula-content">正常：18.5 ~ 24</view>
      <view class="formula-content">偏胖：24 ~ 28</view>
      <view class="formula-content">肥胖：BMI &gt;= 28</view>
    </view>

    <view class="copy-tip">点击结果可复制</view>

    <view class="history-panel" v-if="showHistory">
      <view class="history-header">
        <text>历史记录</text>
        <text class="clear-btn" @click="clearHistory">清空</text>
      </view>
      <scroll-view class="history-list" scroll-y>
        <view class="history-item" v-for="(item, index) in history" :key="index" @click="restoreHistory(item)">
          <view class="history-data">{{ item.height }}cm / {{ item.weight }}kg</view>
          <view class="history-result">BMI: {{ item.bmi }} - {{ item.status }}</view>
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
      height: '',
      weight: '',
      bmiValue: 0,
      healthStatus: '',
      statusClass: '',
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
      this.history = uni.getStorageSync('bmiHistory') || []
    },
    clearHistory() {
      this.vibrate()
      uni.removeStorageSync('bmiHistory')
      this.history = []
    },
    restoreHistory(item) {
      this.vibrate()
      this.height = item.height
      this.weight = item.weight
      this.bmiValue = item.bmi
      this.healthStatus = item.status
      this.updateStatusClass(item.status)
      this.showHistory = false
    },
    copyResult() {
      if (this.bmiValue) {
        uni.setClipboardData({
          data: this.bmiValue.toString(),
          success: () => {
            uni.showToast({ title: '已复制', icon: 'success' })
          }
        })
      }
    },
    updateStatusClass(status) {
      if (status === '偏瘦') this.statusClass = 'thin'
      else if (status === '正常') this.statusClass = 'normal'
      else if (status === '超重') this.statusClass = 'overweight'
      else this.statusClass = 'obese'
    },
    calculateBMI() {
      if (!this.height || !this.weight) {
        uni.showToast({ title: '请输入身高和体重', icon: 'none' })
        return
      }
      this.vibrate()
      const heightInM = this.height / 100
      const bmi = this.weight / (heightInM * heightInM)
      this.bmiValue = bmi.toFixed(2)
      this.updateHealthStatus(bmi)
      this.saveToHistory()
    },
    updateHealthStatus(bmi) {
      if (bmi < 18.5) {
        this.healthStatus = '偏瘦'
        this.statusClass = 'thin'
      } else if (bmi < 24) {
        this.healthStatus = '正常'
        this.statusClass = 'normal'
      } else if (bmi < 28) {
        this.healthStatus = '超重'
        this.statusClass = 'overweight'
      } else {
        this.healthStatus = '肥胖'
        this.statusClass = 'obese'
      }
    },
    saveToHistory() {
      const history = uni.getStorageSync('bmiHistory') || []
      history.unshift({
        height: this.height,
        weight: this.weight,
        bmi: this.bmiValue,
        status: this.healthStatus,
        timestamp: new Date().toISOString()
      })
      if (history.length > 20) {
        history.pop()
      }
      uni.setStorageSync('bmiHistory', history)
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
  color: #4caf50;
  padding: 10rpx 24rpx;
  background: rgba(76, 175, 80, 0.1);
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
  margin-bottom: 20rpx;
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

.thin { color: #ff9800; }
.normal { color: #4caf50; }
.overweight { color: #ff5722; }
.obese { color: #f44336; }

.formula-tip {
  margin-top: 40rpx;
  padding: 30rpx;
  background: rgba(255, 255, 255, 0.5);
  border-radius: 20rpx;
  font-size: 26rpx;
  color: #666;
}

.formula-title {
  font-weight: bold;
  color: #333;
  margin-bottom: 10rpx;
}

.formula-content {
  line-height: 1.6;
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
  color: #4caf50;
}

.history-empty {
  text-align: center;
  color: #999;
  padding: 60rpx;
  font-size: 28rpx;
}
</style>
