<template>
  <view class="container">
    <view class="header-row">
      <view class="history-btn" @click="toggleHistory">历史</view>
    </view>

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
    
    <view class="formula-tip">
      <view class="formula-title">计算公式：</view>
      <view class="formula-content">应纳税所得额 = 月收入 - 五险一金 - 专项扣除 - 5000</view>
      <view class="formula-title" style="margin-top: 16rpx;">税率表（月应纳税所得额）：</view>
      <view class="formula-content">≤ 3000：3%</view>
      <view class="formula-content">3000 ~ 12000：10%</view>
      <view class="formula-content">12000 ~ 25000：20%</view>
      <view class="formula-content">25000 ~ 35000：25%</view>
      <view class="formula-content">35000 ~ 55000：30%</view>
      <view class="formula-content">55000 ~ 80000：35%</view>
      <view class="formula-content">> 80000：45%</view>
    </view>
    
    <view class="result-container">
      <view class="result-item">
        <view class="result-label">应纳税所得额：</view>
        <view class="result-value" @click="copyResult">¥{{ taxableIncome }}</view>
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
    <view class="copy-tip">点击结果可复制</view>

    <view class="history-panel" v-if="showHistory">
      <view class="history-header">
        <text>历史记录</text>
        <text class="clear-btn" @click="clearHistory">清空</text>
      </view>
      <scroll-view class="history-list" scroll-y>
        <view class="history-item" v-for="(item, index) in history" :key="index" @click="restoreHistory(item)">
          <view class="history-data">收入: {{ item.income }} / 税后: {{ item.afterTaxIncome }}</view>
          <view class="history-result">应纳税: ¥{{ item.taxAmount }}</view>
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
      income: '',
      insurance: '',
      deduction: '',
      taxableIncome: 0,
      taxAmount: 0,
      afterTaxIncome: 0,
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
      this.history = uni.getStorageSync('taxHistory') || []
    },
    clearHistory() {
      this.vibrate()
      uni.removeStorageSync('taxHistory')
      this.history = []
    },
    restoreHistory(item) {
      this.vibrate()
      this.income = item.income
      this.insurance = item.insurance
      this.deduction = item.deduction
      this.taxableIncome = item.taxableIncome
      this.taxAmount = item.taxAmount
      this.afterTaxIncome = item.afterTaxIncome
      this.showHistory = false
    },
    copyResult() {
      if (this.taxableIncome) {
        uni.setClipboardData({
          data: this.taxableIncome.toString(),
          success: () => {
            uni.showToast({ title: '已复制', icon: 'success' })
          }
        })
      }
    },
    calculateTax() {
      if (!this.income) {
        uni.showToast({ title: '请输入月收入', icon: 'none' })
        return
      }
      this.vibrate()
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
      if (history.length > 20) {
        history.pop()
      }
      uni.setStorageSync('taxHistory', history)
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
  color: #9c27b0;
  padding: 10rpx 24rpx;
  background: rgba(156, 39, 176, 0.1);
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
  color: #9c27b0;
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
  color: #9c27b0;
}

.history-empty {
  text-align: center;
  color: #999;
  padding: 60rpx;
  font-size: 28rpx;
}
</style>
