<template>
  <view class="container">
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
    
    <view class="result-container">
      <view class="result-item">
        <view class="result-label">月供 (等额本息)：</view>
        <view class="result-value">¥{{ monthlyPayment }}</view>
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
      totalPayment: 0
    }
  },
  methods: {
    calculateMortgage() {
      if (!this.loanAmount || !this.loanYears || !this.interestRate) {
        uni.showToast({
          title: '请输入完整信息',
          icon: 'none'
        })
        return
      }
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
      if (history.length > 10) {
        history.pop()
      }
      uni.setStorageSync('mortgageHistory', history)
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
  margin-bottom: 30rpx;
}

.result-label {
  font-size: 32rpx;
  color: #666;
}

.result-value {
  font-size: 40rpx;
  font-weight: bold;
  color: #2196f3;
}
</style>
