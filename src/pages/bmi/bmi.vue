<template>
  <view class="container">
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
        <view class="result-value">{{ bmiValue }}</view>
      </view>
      <view class="result-item">
        <view class="result-label">健康状况：</view>
        <view class="result-value" :class="statusClass">{{ healthStatus }}</view>
      </view>
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
      statusClass: ''
    }
  },
  methods: {
    calculateBMI() {
      if (!this.height || !this.weight) {
        uni.showToast({
          title: '请输入身高和体重',
          icon: 'none'
        })
        return
      }
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
      if (history.length > 10) {
        history.pop()
      }
      uni.setStorageSync('bmiHistory', history)
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

.thin {
  color: #ff9800;
}

.normal {
  color: #4caf50;
}

.overweight {
  color: #ff5722;
}

.obese {
  color: #f44336;
}
</style>
