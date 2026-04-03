<template>
  <view class="container">
    <view class="header-row">
      <view class="history-btn" @click="toggleHistory">历史</view>
    </view>

    <view class="input-group">
      <view class="label">贷款金额 (万元)</view>
      <input class="input" type="number" v-model="loanAmount" placeholder="请输入贷款金额" />
    </view>
    
    <view class="input-group">
      <view class="label">贷款年限</view>
      <input class="input" type="number" v-model="loanYears" placeholder="请输入贷款年限" />
    </view>
    
    <view class="input-group">
      <view class="label">年利率 (%)</view>
      <input class="input" type="number" v-model="interestRate" placeholder="请输入年利率" />
    </view>
    
    <view class="calculate-btn" @click="calculateMortgage">计算房贷</view>
    
    <view class="formula-tip">
      <view class="formula-title">等额本息公式：</view>
      <view class="formula-content">月供 = 贷款本金 × 月利率 × (1+月利率)^还款月数 / ((1+月利率)^还款月数-1)</view>
      <view class="formula-title" style="margin-top: 16rpx;">其中：</view>
      <view class="formula-content">月利率 = 年利率 ÷ 12 ÷ 100</view>
    </view>
    
    <view class="result-container">
      <view class="result-item">
        <view class="result-label">月供 (等额本息)：</view>
        <view class="result-value" @click="copyResult">¥{{ monthlyPayment }}</view>
      </view>
      <view class="result-item">
        <view class="result-label">总利息：</view>
        <view class="result-value">¥{{ totalInterest }}</view>
      </view>
      <view class="result-item">
        <view class="result-label">还款总额：</view>
        <view class="result-value">¥{{ totalPayment }}</view>
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
          <view class="history-data">{{ item.loanAmount }}万 / {{ item.loanYears }}年 / {{ item.interestRate }}%</view>
          <view class="history-result">月供: ¥{{ item.monthlyPayment }}</view>
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
      loanAmount: '',
      loanYears: '',
      interestRate: '',
      monthlyPayment: 0,
      totalInterest: 0,
      totalPayment: 0,
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
      this.history = uni.getStorageSync('mortgageHistory') || []
    },
    clearHistory() {
      this.vibrate()
      uni.removeStorageSync('mortgageHistory')
      this.history = []
    },
    restoreHistory(item) {
      this.vibrate()
      this.loanAmount = item.loanAmount
      this.loanYears = item.loanYears
      this.interestRate = item.interestRate
      this.monthlyPayment = item.monthlyPayment
      this.totalInterest = item.totalInterest
      this.totalPayment = item.totalPayment
      this.showHistory = false
    },
    copyResult() {
      if (this.monthlyPayment) {
        uni.setClipboardData({
          data: this.monthlyPayment.toString(),
          success: () => {
            uni.showToast({ title: '已复制', icon: 'success' })
          }
        })
      }
    },
    calculateMortgage() {
      if (!this.loanAmount || !this.loanYears || !this.interestRate) {
        uni.showToast({ title: '请输入完整信息', icon: 'none' })
        return
      }
      this.vibrate()
      const principal = this.loanAmount * 10000
      const monthlyRate = this.interestRate / 100 / 12
      const totalMonths = this.loanYears * 12
      
      const payment = principal * monthlyRate * Math.pow(1 + monthlyRate, totalMonths) / 
                     (Math.pow(1 + monthlyRate, totalMonths) - 1)
      
      this.monthlyPayment = payment.toFixed(2)
      this.totalPayment = (payment * totalMonths).toFixed(2)
      this.totalInterest = (payment * totalMonths - principal).toFixed(2)
      
      this.saveToHistory()
    },
    saveToHistory() {
      const history = uni.getStorageSync('mortgageHistory') || []
      history.unshift({
        loanAmount: this.loanAmount,
        loanYears: this.loanYears,
        interestRate: this.interestRate,
        monthlyPayment: this.monthlyPayment,
        totalInterest: this.totalInterest,
        totalPayment: this.totalPayment,
        timestamp: new Date().toISOString()
      })
      if (history.length > 20) {
        history.pop()
      }
      uni.setStorageSync('mortgageHistory', history)
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
  color: #ff9800;
  padding: 10rpx 24rpx;
  background: rgba(255, 152, 0, 0.1);
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
  margin-bottom: 30rpx;
}

.result-label {
  font-size: 32rpx;
  color: #666;
}

.result-value {
  font-size: 40rpx;
  font-weight: bold;
  color: #ff9800;
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
  color: #ff9800;
}

.history-empty {
  text-align: center;
  color: #999;
  padding: 60rpx;
  font-size: 28rpx;
}
</style>
