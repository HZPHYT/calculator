<template>
  <view class="calculator-container">
    <view class="display-container">
      <view class="expression">{{ expression }}</view>
      <view class="result">{{ result }}</view>
    </view>
    
    <view class="keypad">
      <view class="row">
        <view class="key func-key" @click="clear">AC</view>
        <view class="key func-key" @click="backspace">←</view>
        <view class="key func-key" @click="append('%')">%</view>
        <view class="key operator-key" @click="append('/')">÷</view>
      </view>
      <view class="row">
        <view class="key num-key" @click="append('7')">7</view>
        <view class="key num-key" @click="append('8')">8</view>
        <view class="key num-key" @click="append('9')">9</view>
        <view class="key operator-key" @click="append('*')">×</view>
      </view>
      <view class="row">
        <view class="key num-key" @click="append('4')">4</view>
        <view class="key num-key" @click="append('5')">5</view>
        <view class="key num-key" @click="append('6')">6</view>
        <view class="key operator-key" @click="append('-')">−</view>
      </view>
      <view class="row">
        <view class="key num-key" @click="append('1')">1</view>
        <view class="key num-key" @click="append('2')">2</view>
        <view class="key num-key" @click="append('3')">3</view>
        <view class="key operator-key" @click="append('+')">+</view>
      </view>
      <view class="row">
        <view class="key num-key zero-key" @click="append('0')">0</view>
        <view class="key num-key" @click="append('.')">.</view>
        <view class="key equals-key" @click="calculate">=</view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      expression: '',
      result: '0'
    }
  },
  methods: {
    append(value) {
      this.expression += value
      this.calculatePreview()
    },
    clear() {
      this.expression = ''
      this.result = '0'
    },
    backspace() {
      this.expression = this.expression.slice(0, -1)
      this.calculatePreview()
    },
    calculatePreview() {
      if (!this.expression) {
        this.result = '0'
        return
      }
      try {
        const expr = this.expression.replace(/×/g, '*').replace(/÷/g, '/')
        const res = eval(expr)
        this.result = res.toString()
      } catch (e) {
        this.result = '错误'
      }
    },
    calculate() {
      if (!this.expression) return
      try {
        const expr = this.expression.replace(/×/g, '*').replace(/÷/g, '/')
        const res = eval(expr)
        this.result = res.toString()
        this.saveToHistory()
      } catch (e) {
        this.result = '错误'
      }
    },
    saveToHistory() {
      const history = uni.getStorageSync('calculatorHistory') || []
      history.unshift({
        expression: this.expression,
        result: this.result,
        timestamp: new Date().toISOString()
      })
      if (history.length > 10) {
        history.pop()
      }
      uni.setStorageSync('calculatorHistory', history)
    }
  }
}
</script>

<style lang="scss" scoped>
.calculator-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #e3f2fd 0%, #bbdefb 100%);
  display: flex;
  flex-direction: column;
}

.display-container {
  flex: 1;
  padding: 60rpx 40rpx;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  align-items: flex-end;
  background: rgba(255, 255, 255, 0.7);
  margin: 40rpx;
  border-radius: 30rpx;
  box-shadow: 0 8rpx 24rpx rgba(0, 0, 0, 0.1);
}

.expression {
  font-size: 40rpx;
  color: #666;
  margin-bottom: 20rpx;
  word-break: break-all;
  text-align: right;
}

.result {
  font-size: 80rpx;
  font-weight: bold;
  color: #333;
  word-break: break-all;
  text-align: right;
}

.keypad {
  padding: 40rpx;
  background: rgba(255, 255, 255, 0.9);
  border-radius: 40rpx 40rpx 0 0;
  box-shadow: 0 -8rpx 24rpx rgba(0, 0, 0, 0.1);
}

.row {
  display: flex;
  justify-content: space-between;
  margin-bottom: 30rpx;
}

.key {
  flex: 1;
  margin: 0 16rpx;
  padding: 40rpx;
  border-radius: 30rpx;
  font-size: 48rpx;
  font-weight: 500;
  text-align: center;
  box-shadow: 0 4rpx 16rpx rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
}

.key:active {
  transform: scale(0.95);
}

.num-key {
  background: rgba(255, 255, 255, 0.8);
  color: #333;
}

.func-key {
  background: rgba(245, 245, 245, 0.8);
  color: #666;
}

.operator-key {
  background: rgba(33, 150, 243, 0.2);
  color: #2196f3;
}

.equals-key {
  background: #2196f3;
  color: white;
}

.zero-key {
  flex: 2;
}
</style>
