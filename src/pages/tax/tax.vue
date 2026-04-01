<template>
  <view class="container">
    <view class="input-group">
      <view class="label">月收入 (元)</view>
      <input class="input" type="number" v-model="income" placeholder="请输入月收入" />
    </view>
    
    <view class="input-group">
      <view class="label">五险一金 (元)</view>
      <input class="input" type="number" v-model="insurance" placeholder="请输入五险一金" />
    </view>
    
    <view class="input-group">
      <view class="label">专项扣除 (元)</view>
      <input class="input" type="number" v-model="deduction" placeholder="请输入专项扣除" />
    </view>
    
    <view class="calculate-btn" @click="calculateTax">计算个税</view>
    
    <view class="result-container">
      <view class="result-item">
        <view class="result-label">应纳税所得额：</view>
        <view class="result-value">¥{{ taxableIncome }}</view>
      </view>
      <view class="result-item">
        <view class="result-label">应纳税额：</view>
        <view class="result-value">¥{{ taxAmount }}</view>
      </view>
      <view class="result-item">
        <view class="result-label">税后收入：</view>
        <view class="result-value">¥{{ afterTaxIncome }}</view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      income: '',
      insurance: '',
      deduction: '',
      taxableIncome: 0,
      taxAmount: 0,
      afterTaxIncome: 0
    }
  },
  methods: {
    calculateTax() {
      if (!this.income) {
        uni.showToast({
          title: '请输入月收入',
          icon: 'none'
        })
        return
      }
      const baseDeduction = 5000
      const taxable = this.income - (this.insurance || 0) - (this.deduction || 0) - baseDeduction
      
      this.taxableIncome = Math.max(0, taxable)
      this.taxAmount = this.calculateTaxAmount(this.taxableIncome)
      this.afterTaxIncome = this.income - (this.insurance || 0) - this.taxAmount
      
      this.saveToHistory()
    },
    calculateTaxAmount(taxable) {
      if (taxable <= 0) return 0
      if (taxable <= 3000) return taxable * 0.03
      if (taxable <= 12000) return 3000 * 0.03 + (taxable - 3000) * 0.1
      if (taxable <= 25000) return 3000 * 0.03 + 9000 * 0.1 + (taxable - 12000) * 0.2
      if (taxable <= 35000) return 3000 * 0.03 + 9000 * 0.1 + 13000 * 0.2 + (taxable - 25000) * 0.25
      if (taxable <= 55000) return 3000 * 0.03 + 9000 * 0.1 + 13000 * 0.2 + 10000 * 0.25 + (taxable - 35000) * 0.3
      if (taxable <= 80000) return 3000 * 0.03 + 9000 * 0.1 + 13000 * 0.2 + 10000 * 0.25 + 20000 * 0.3 + (taxable - 55000) * 0.35
      return 3000 * 0.03 + 9000 * 0.1 + 13000 * 0.2 + 10000 * 0.25 + 20000 * 0.3 + 25000 * 0.35 + (taxable - 80000) * 0.45
    },
    saveToHistory() {
      const history = uni.getStorageSync('taxHistory') || []
      history.unshift({
        income: this.income,
        insurance: this.insurance,
        deduction: this.deduction,
        taxableIncome: this.taxableIncome,
        taxAmount: this.taxAmount,
        afterTaxIncome: this.afterTaxIncome,
        timestamp: new Date().toISOString()
      })
      if (history.length > 10) {
        history.pop()
      }
      uni.setStorageSync('taxHistory', history)
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
