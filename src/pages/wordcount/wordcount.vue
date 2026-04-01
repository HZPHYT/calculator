<template>
  <view class="container">
    <view class="input-group">
      <view class="label">输入文本</view>
      <textarea class="textarea" v-model="text" placeholder="请输入文本" />
    </view>
    
    <view class="result-container">
      <view class="result-item">
        <view class="result-label">字符数：</view>
        <view class="result-value">{{ charCount }}</view>
      </view>
      <view class="result-item">
        <view class="result-label">字数（不含空格）：</view>
        <view class="result-value">{{ wordCount }}</view>
      </view>
      <view class="result-item">
        <view class="result-label">字数（含空格）：</view>
        <view class="result-value">{{ wordCountWithSpace }}</view>
      </view>
      <view class="result-item">
        <view class="result-label">行数：</view>
        <view class="result-value">{{ lineCount }}</view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      text: ''
    }
  },
  computed: {
    charCount() {
      return this.text.length
    },
    wordCount() {
      return this.text.replace(/\s/g, '').length
    },
    wordCountWithSpace() {
      return this.text.length
    },
    lineCount() {
      return this.text.split('\n').length
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

.textarea {
  width: 100%;
  min-height: 400rpx;
  padding: 24rpx 32rpx;
  background: rgba(255, 255, 255, 0.8);
  border-radius: 20rpx;
  font-size: 32rpx;
  color: #333;
  box-shadow: 0 4rpx 16rpx rgba(0, 0, 0, 0.1);
}

.result-container {
  margin-top: 40rpx;
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
