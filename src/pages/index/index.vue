<template>
  <view class="container">
    <view class="header">
      <view class="title">计算器集合</view>
      <view class="search-box">
        <input 
          class="search-input" 
          type="text" 
          v-model="searchQuery" 
          placeholder="搜索计算器..."
          placeholder-class="search-placeholder"
        />
      </view>
    </view>
    
    <view class="calculator-grid">
      <view 
        class="calculator-card" 
        v-for="calculator in filteredCalculators" 
        :key="calculator.id"
        @click="navigateTo(calculator.path)"
      >
        <view class="calculator-icon">{{ calculator.icon }}</view>
        <view class="calculator-name">{{ calculator.name }}</view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      searchQuery: '',
      calculators: [
        { id: 1, name: '标准计算器', icon: '🔢', path: '/pages/standard/standard' },
        { id: 2, name: '科学计算器', icon: '🔬', path: '/pages/scientific/scientific' },
        { id: 3, name: '日期天数计算', icon: '📅', path: '/pages/date/date' },
        { id: 4, name: 'BMI 健康计算器', icon: '⚖️', path: '/pages/bmi/bmi' },
        { id: 5, name: '房贷计算器', icon: '🏠', path: '/pages/mortgage/mortgage' },
        { id: 6, name: '个税计算器', icon: '💼', path: '/pages/tax/tax' },
        { id: 7, name: '时间戳转换', icon: '⏰', path: '/pages/timestamp/timestamp' },
        { id: 8, name: '进制转换', icon: '🔄', path: '/pages/base/base' },
        { id: 9, name: '字数统计', icon: '📝', path: '/pages/wordcount/wordcount' }
      ]
    }
  },
  computed: {
    filteredCalculators() {
      if (!this.searchQuery) {
        return this.calculators
      }
      return this.calculators.filter(calc => 
        calc.name.toLowerCase().includes(this.searchQuery.toLowerCase())
      )
    }
  },
  methods: {
    navigateTo(path) {
      uni.navigateTo({
        url: path
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.container {
  min-height: 100vh;
  background: linear-gradient(135deg, #e3f2fd 0%, #bbdefb 100%);
  padding: 20rpx;
}

.header {
  margin-bottom: 40rpx;
}

.title {
  font-size: 48rpx;
  font-weight: bold;
  color: #2196f3;
  text-align: center;
  margin-bottom: 30rpx;
}

.search-box {
  width: 100%;
}

.search-input {
  width: 100%;
  height: 80rpx;
  padding: 0 30rpx;
  border-radius: 40rpx;
  background: rgba(255, 255, 255, 0.8);
  font-size: 28rpx;
  box-shadow: 0 4rpx 16rpx rgba(0, 0, 0, 0.1);
}

.search-placeholder {
  color: #999;
}

.calculator-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20rpx;
}

.calculator-card {
  background: rgba(255, 255, 255, 0.9);
  border-radius: 20rpx;
  padding: 40rpx 20rpx;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  box-shadow: 0 4rpx 16rpx rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
}

.calculator-card:active {
  transform: scale(0.95);
}

.calculator-icon {
  font-size: 60rpx;
  margin-bottom: 20rpx;
}

.calculator-name {
  font-size: 24rpx;
  color: #333;
  text-align: center;
  font-weight: 500;
}
</style>
