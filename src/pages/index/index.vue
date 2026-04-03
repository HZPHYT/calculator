<template>
  <view class="container" :class="{ 'dark-mode': isDark }">
    <view class="header">
      <view class="title-row">
        <view class="title">计算器集合</view>
        <view class="theme-btn" @click="toggleTheme">
          {{ isDark ? '☀️' : '🌙' }}
        </view>
      </view>
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
    
    <view class="sort-tip" v-if="!searchQuery">
      <text class="tip-text">按使用频率排序 · 长按卡片进入排序模式</text>
    </view>
    
    <view class="calculator-grid">
      <view 
        class="calculator-card" 
        v-for="(calculator, index) in filteredCalculators" 
        :key="calculator.id"
        @click="navigateTo(calculator.path)"
        @longpress="editMode = true"
      >
        <view class="edit-mark" v-if="editMode">{{ index + 1 }}</view>
        <view class="calculator-icon">{{ calculator.icon }}</view>
        <view class="calculator-name">{{ calculator.name }}</view>
      </view>
    </view>

    <view class="edit-panel" v-if="editMode">
      <view class="edit-header">
        <view class="edit-tip">长按卡片可进入排序模式，使用↑↓调整顺序</view>
        <view style="display: flex; gap: 20rpx;">
          <text class="reset-btn" @click="resetOrder">恢复默认</text>
          <text class="done-btn" @click="saveOrder">完成</text>
        </view>
      </view>
      <scroll-view class="edit-list" scroll-y>
        <view class="edit-item" 
          v-for="(calculator, index) in calculators" 
          :key="calculator.id"
        >
          <view class="edit-index">{{ index + 1 }}</view>
          <view class="edit-icon">{{ calculator.icon }}</view>
          <view class="edit-name">{{ calculator.name }}</view>
          <view class="edit-move" @click="moveUp(index)" v-if="index > 0">↑</view>
          <view class="edit-move" @click="moveDown(index)" v-if="index < calculators.length - 1">↓</view>
        </view>
      </scroll-view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      searchQuery: '',
      isDark: false,
      editMode: false,
      calculators: []
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
  onLoad() {
    this.loadCalculators()
    this.isDark = uni.getStorageSync('isDarkMode') || false
    uni.$on('themeChange', (isDark) => {
      this.isDark = isDark
    })
  },
  onShow() {
    this.isDark = uni.getStorageSync('isDarkMode') || false
  },
  methods: {
    loadCalculators() {
      const defaultCalcs = [
        { id: 1, name: '标准计算器', icon: '🔢', path: '/pages/standard/standard', order: 1 },
        { id: 2, name: '单位换算器', icon: '🔄', path: '/pages/unit/unit', order: 2 },
        { id: 3, name: '日期天数计算', icon: '📅', path: '/pages/date/date', order: 3 },
        { id: 4, name: 'BMI 健康计算器', icon: '⚖️', path: '/pages/bmi/bmi', order: 4 },
        { id: 5, name: '房贷计算器', icon: '🏠', path: '/pages/mortgage/mortgage', order: 5 },
        { id: 6, name: '个税计算器', icon: '💼', path: '/pages/tax/tax', order: 6 },
        { id: 7, name: '时间戳转换', icon: '⏰', path: '/pages/timestamp/timestamp', order: 7 },
        { id: 8, name: '进制转换', icon: '🔄', path: '/pages/base/base', order: 8 },
        { id: 9, name: '字数统计', icon: '📝', path: '/pages/wordcount/wordcount', order: 9 }
      ]
      
      const savedOrder = uni.getStorageSync('calculatorOrder')
      const usageCount = uni.getStorageSync('calculatorUsage') || {}
      
      if (savedOrder) {
        this.calculators = savedOrder.map(id => {
          const calc = defaultCalcs.find(c => c.id === id)
          if (calc) calc.usage = usageCount[id] || 0
          return calc
        }).filter(Boolean)
      } else {
        this.calculators = defaultCalcs.map(calc => {
          calc.usage = usageCount[calc.id] || 0
          return calc
        })
      }
      
      this.calculators.sort((a, b) => b.usage - a.usage || a.order - b.order)
    },
    navigateTo(path) {
      const calc = this.calculators.find(c => c.path === path)
      if (calc) {
        const usage = uni.getStorageSync('calculatorUsage') || {}
        usage[calc.id] = (usage[calc.id] || 0) + 1
        uni.setStorageSync('calculatorUsage', usage)
      }
      
      if (this.editMode) {
        this.saveOrder()
        return
      }
      
      uni.navigateTo({ url: path })
    },
    toggleTheme() {
      const app = getApp()
      app.toggleTheme()
    },
    moveUp(index) {
      if (index > 0) {
        const temp = this.calculators[index]
        this.calculators[index] = this.calculators[index - 1]
        this.calculators[index - 1] = temp
      }
    },
    moveDown(index) {
      if (index < this.calculators.length - 1) {
        const temp = this.calculators[index]
        this.calculators[index] = this.calculators[index + 1]
        this.calculators[index + 1] = temp
      }
    },
    saveOrder() {
      const order = this.calculators.map(c => c.id)
      uni.setStorageSync('calculatorOrder', order)
      this.editMode = false
      uni.showToast({ title: '排序已保存', icon: 'success' })
    },
    resetOrder() {
      uni.removeStorageSync('calculatorOrder')
      this.loadCalculators()
      this.editMode = false
      uni.showToast({ title: '已恢复默认排序', icon: 'success' })
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
  margin-bottom: 30rpx;
}

.title-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20rpx;
}

.title {
  font-size: 48rpx;
  font-weight: bold;
  color: #2196f3;
}

.theme-btn {
  font-size: 48rpx;
  padding: 10rpx;
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

.sort-tip {
  text-align: center;
  margin-bottom: 20rpx;
}

.tip-text {
  font-size: 24rpx;
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
  position: relative;
}

.calculator-card:active {
  transform: scale(0.95);
}

.edit-mark {
  position: absolute;
  top: 10rpx;
  left: 10rpx;
  width: 36rpx;
  height: 36rpx;
  background: #ff9800;
  color: white;
  font-size: 22rpx;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
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

.edit-panel {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  height: 50vh;
  background: white;
  border-radius: 30rpx 30rpx 0 0;
  box-shadow: 0 -4rpx 24rpx rgba(0, 0, 0, 0.15);
  z-index: 1000;
}

.edit-header {
  padding: 30rpx 40rpx;
  border-bottom: 1px solid #eee;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 28rpx;
  color: #666;
}

.done-btn {
  color: #2196f3;
  font-weight: bold;
}

.reset-btn {
  color: #999;
  font-size: 28rpx;
}

.edit-tip {
  font-size: 24rpx;
  color: #999;
  margin-bottom: 10rpx;
}

.edit-list {
  height: calc(50vh - 100rpx);
  padding: 20rpx 40rpx;
}

.edit-item {
  display: flex;
  align-items: center;
  padding: 20rpx 0;
  border-bottom: 1px solid #f5f5f5;
}

.edit-index {
  width: 40rpx;
  height: 40rpx;
  background: #e3f2fd;
  color: #2196f3;
  font-size: 24rpx;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 20rpx;
}

.edit-icon {
  font-size: 36rpx;
  margin-right: 20rpx;
}

.edit-name {
  flex: 1;
  font-size: 28rpx;
  color: #333;
}

.edit-move {
  width: 60rpx;
  height: 60rpx;
  background: #f5f5f5;
  color: #666;
  font-size: 28rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-left: 10rpx;
  border-radius: 10rpx;
}

.dark-mode {
  background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
  
  .title {
    color: #64b5f6;
  }
  
  .search-input {
    background: rgba(255, 255, 255, 0.1);
    color: #ffffff;
  }
  
  .search-placeholder {
    color: #888;
  }
  
  .sort-tip {
    .tip-text {
      color: #888;
    }
  }
  
  .calculator-card {
    background: rgba(255, 255, 255, 0.1);
  }
  
  .calculator-name {
    color: #e0e0e0;
  }
  
  .edit-panel {
    background: #1a1a2e;
  }
  
  .edit-header {
    color: #e0e0e0;
    border-bottom-color: #333;
  }
  
  .edit-index {
    background: rgba(100, 181, 246, 0.2);
    color: #64b5f6;
  }
  
  .edit-name {
    color: #e0e0e0;
  }
  
  .edit-move {
    background: rgba(255, 255, 255, 0.1);
    color: #e0e0e0;
  }
}
</style>
