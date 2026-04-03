<template>
  <view class="calculator-container">
    <view class="display-container">
      <view class="header-row">
        <view class="history-btn" @click="toggleHistory">历史</view>
      </view>
      <view class="expression">{{ expression }}</view>
      <view class="result" @click="copyResult">{{ result }}</view>
    </view>
    <view class="copy-tip">点击结果可复制</view>
    
    <view class="keypad">
      <view class="row">
        <view class="key func-key" @click="append('√')">√</view>
        <view class="key func-key" @click="square">x²</view>
        <view class="key func-key" @click="backspace">←</view>
        <view class="key func-key" @click="append('%')">%</view>
      </view>
      <view class="row">
        <view class="key func-key" @click="clear">AC</view>
        <view class="key func-key" @click="append('(')">(</view>
        <view class="key func-key" @click="append(')')">)</view>
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

    <view class="history-panel" v-if="showHistory">
      <view class="history-header">
        <text>计算历史</text>
        <view style="display: flex; gap: 20rpx;">
          <text class="close-btn" @click="toggleHistory">关闭</text>
          <text v-if="!editMode" class="edit-btn" @click="enterEditMode">编辑</text>
          <text v-else class="edit-btn" @click="exitEditMode">完成</text>
          <text class="clear-btn" @click="editMode ? batchDelete() : clearHistory()">{{ editMode ? '删除' : '清空' }}</text>
        </view>
      </view>
      <scroll-view class="history-list" scroll-y>
        <view class="history-item" v-for="(item, index) in history" :key="index" @click="editMode ? toggleSelect(index) : restoreHistory(item)">
          <view v-if="editMode" class="checkbox" :class="{ selected: selectedItems.includes(index) }">✓</view>
          <view class="history-content">
            <view class="history-expression">{{ item.expression }}</view>
            <view class="history-result">= {{ formatNumber(item.result) }}</view>
          </view>
        </view>
        <view class="history-empty" v-if="history.length === 0">暂无历史记录</view>
      </scroll-view>
    </view>

    <view class="copy-modal" v-if="showCopyModal" @click="showCopyModal = false">
      <view class="copy-modal-content" @click.stop>
        <view class="copy-modal-title">复制方式</view>
        <view class="copy-modal-item" @click="copyResultOnly">仅复制结果</view>
        <view class="copy-modal-item" @click="copyFullExpression">复制完整表达式</view>
      </view>
    </view>
  </view>
</template>

<script>
import { create, all } from 'mathjs'

const math = create(all)

export default {
  data() {
    return {
      expression: '',
      result: '0',
      showHistory: false,
      showCopyModal: false,
      history: [],
      editMode: false,
      selectedItems: []
    }
  },
  onShow() {
    this.loadHistory()
  },
  methods: {
    vibrate() {},
    evaluate(expr) {
      let processed = expr
        .replace(/×/g, '*').replace(/÷/g, '/').replace(/−/g, '-')
        .replace(/(\d+)²/g, '($1*$1)')
        .replace(/√(\d+(\.\d+)?)/g, 'sqrt($1)')
        .replace(/π/g, 'pi')
      const percentResult = processed.replace(/(\d+(?:\.\d+)?)%/g, (match, num) => `${num}/100`)
      try {
        return math.evaluate(percentResult)
      } catch (e) {
        return '错误'
      }
    },
    formatNumber(num) {
      if (!num || num === '错误') return num
      const n = parseFloat(num)
      if (isNaN(n)) return num
      return n.toLocaleString('zh-CN', { maximumFractionDigits: 8 })
    },
    toggleHistory() {
      this.showHistory = !this.showHistory
      if (this.showHistory) {
        this.exitEditMode()
      }
    },
    loadHistory() {
      this.history = uni.getStorageSync('standardHistory') || []
    },
    clearHistory() {
      uni.removeStorageSync('standardHistory')
      this.history = []
    },
    enterEditMode() {
      this.editMode = true
      this.selectedItems = []
    },
    exitEditMode() {
      this.editMode = false
      this.selectedItems = []
    },
    toggleSelect(index) {
      const idx = this.selectedItems.indexOf(index)
      if (idx > -1) {
        this.selectedItems.splice(idx, 1)
      } else {
        this.selectedItems.push(index)
      }
    },
    batchDelete() {
      if (this.selectedItems.length === 0) {
        uni.showToast({ title: '请先选择', icon: 'none' })
        return
      }
      const newHistory = this.history.filter((_, i) => !this.selectedItems.includes(i))
      this.history = newHistory
      uni.setStorageSync('standardHistory', newHistory)
      this.exitEditMode()
    },
    restoreHistory(item) {
      this.expression = item.expression
      this.result = item.result
      this.showHistory = false
    },
    copyResult() {
      if (this.result && this.result !== '0' && this.result !== '错误') {
        this.showCopyModal = true
      }
    },
    copyResultOnly() {
      this.showCopyModal = false
      uni.setClipboardData({
        data: this.result,
        success: () => {
          uni.showToast({ title: '已复制', icon: 'success' })
        }
      })
    },
    copyFullExpression() {
      this.showCopyModal = false
      uni.setClipboardData({
        data: this.expression + ' = ' + this.result,
        success: () => {
          uni.showToast({ title: '已复制', icon: 'success' })
        }
      })
    },
    append(value) {
      this.vibrate()
      this.expression += value
      this.calculatePreview()
    },
    clear() {
      this.vibrate()
      this.expression = ''
      this.result = '0'
    },
    backspace() {
      this.vibrate()
      this.expression = this.expression.slice(0, -1)
      this.calculatePreview()
    },
    square() {
      this.vibrate()
      this.expression += '²'
      this.calculatePreview()
    },
    calculatePreview() {
      if (!this.expression) {
        this.result = '0'
        return
      }
      const raw = this.expression.trim()
      if (raw.endsWith('+') || raw.endsWith('-') || raw.endsWith('*') || raw.endsWith('/') || raw.endsWith('^') || raw.endsWith('(') || raw.endsWith('√')) {
        this.result = '0'
        return
      }
      const res = this.evaluate(raw)
      if (res === '错误' || isNaN(res) || !isFinite(res)) {
        this.result = '错误'
      } else {
        this.result = Number.isInteger(res) ? res.toString() : res.toFixed(8).replace(/\.?0+$/, '')
      }
    },
    calculate() {
      if (!this.expression) return
      const raw = this.expression.trim()
      if (raw.endsWith('+') || raw.endsWith('-') || raw.endsWith('*') || raw.endsWith('/') || raw.endsWith('^') || raw.endsWith('(') || raw.endsWith('√')) {
        uni.showToast({ title: '表达式不完整', icon: 'none' })
        return
      }
      const res = this.evaluate(raw)
      if (res === '错误' || isNaN(res) || !isFinite(res)) {
        this.result = '错误'
      } else {
        this.result = Number.isInteger(res) ? res.toString() : res.toFixed(8).replace(/\.?0+$/, '')
      }
      this.saveToHistory()
    },
    saveToHistory() {
      const history = uni.getStorageSync('standardHistory') || []
      const existingIndex = history.findIndex(item => item.expression === this.expression)
      if (existingIndex !== -1) {
        history[existingIndex].result = this.result
        history[existingIndex].timestamp = new Date().toISOString()
      } else {
        history.unshift({
          expression: this.expression,
          result: this.result,
          timestamp: new Date().toISOString()
        })
      }
      if (history.length > 20) {
        history.pop()
      }
      uni.setStorageSync('standardHistory', history)
      this.history = history
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

.header-row {
  width: 100%;
  display: flex;
  justify-content: flex-end;
  margin-bottom: 20rpx;
}

.history-btn {
  font-size: 28rpx;
  color: #2196f3;
  padding: 10rpx 24rpx;
  background: rgba(33, 150, 243, 0.1);
  border-radius: 20rpx;
}

.expression {
  font-size: 40rpx;
  color: #666;
  margin-bottom: 20rpx;
  word-break: break-all;
  text-align: right;
  width: 100%;
}

.result {
  font-size: 80rpx;
  font-weight: bold;
  color: #333;
  word-break: break-all;
  text-align: right;
  width: 100%;
}

.copy-tip {
  text-align: center;
  font-size: 24rpx;
  color: #999;
  padding: 10rpx 0;
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
  display: flex;
  align-items: center;
}

.history-content {
  flex: 1;
}

.edit-btn {
  color: #2196f3;
  font-size: 28rpx;
}

.close-btn {
  color: #999;
  font-size: 28rpx;
}

.checkbox {
  width: 40rpx;
  height: 40rpx;
  border: 2rpx solid #ccc;
  border-radius: 8rpx;
  margin-right: 20rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24rpx;
  color: transparent;
}

.checkbox.selected {
  background: #2196f3;
  border-color: #2196f3;
  color: white;
}

.history-expression {
  font-size: 28rpx;
  color: #666;
  margin-bottom: 10rpx;
}

.history-result {
  font-size: 36rpx;
  font-weight: bold;
  color: #2196f3;
}

.history-empty {
  text-align: center;
  color: #999;
  padding: 60rpx;
  font-size: 28rpx;
}

.copy-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100vh;
  background: rgba(0, 0, 0, 0.5);
  z-index: 1000;
  display: flex;
  align-items: flex-end;
  justify-content: center;
}

.copy-modal-content {
  width: 100%;
  background: white;
  border-radius: 24rpx 24rpx 0 0;
  padding: 40rpx;
}

.copy-modal-title {
  text-align: center;
  font-size: 32rpx;
  font-weight: bold;
  color: #333;
  margin-bottom: 30rpx;
}

.copy-modal-item {
  padding: 30rpx;
  text-align: center;
  font-size: 32rpx;
  color: #2196f3;
  border-bottom: 1px solid #f0f0f0;
}

.copy-modal-item:last-child {
  border-bottom: none;
}
</style>
