<template>
  <view class="converter-container">
    <view class="header">
      <view class="title">单位换算</view>
    </view>

    <view class="category-tabs">
      <view 
        class="tab" 
        :class="{ active: category === 'length' }"
        @click="category = 'length'"
      >长度</view>
      <view 
        class="tab" 
        :class="{ active: category === 'weight' }"
        @click="category = 'weight'"
      >重量</view>
      <view 
        class="tab" 
        :class="{ active: category === 'temperature' }"
        @click="category = 'temperature'"
      >温度</view>
      <view 
        class="tab" 
        :class="{ active: category === 'area' }"
        @click="category = 'area'"
      >面积</view>
      <view 
        class="tab" 
        :class="{ active: category === 'volume' }"
        @click="category = 'volume'"
      >体积</view>
    </view>

    <view class="converter-content">
      <view class="input-group">
        <input 
          class="value-input" 
          type="digit" 
          v-model="inputValue" 
          @input="convert"
          placeholder="请输入数值"
        />
        <picker 
          class="unit-picker"
          :value="fromIndex"
          :range="units[category]"
          @change="onFromChange"
        >
          <view class="picker-value">{{ units[category][fromIndex] }}</view>
        </picker>
      </view>

      <view class="swap-btn" @click="swap">
        <text>⇅ 交换</text>
      </view>

      <view class="input-group">
        <input 
          class="value-input result-value" 
          type="text" 
          :value="result" 
          disabled
        />
        <picker 
          class="unit-picker"
          :value="toIndex"
          :range="units[category]"
          @change="onToChange"
        >
          <view class="picker-value">{{ units[category][toIndex] }}</view>
        </picker>
      </view>
    </view>

    <view class="copy-tip">点击结果可复制</view>

    <view class="quick-convert">
      <view class="quick-title">常用换算</view>
      <view class="quick-list" v-if="category === 'length'">
        <view class="quick-item" @click="quickConvert('1', '米', '厘米')">1米 = 100厘米</view>
        <view class="quick-item" @click="quickConvert('1', '公里', '英里')">1公里 ≈ 0.62英里</view>
        <view class="quick-item" @click="quickConvert('1', '英寸', '厘米')">1英寸 = 2.54厘米</view>
      </view>
      <view class="quick-list" v-if="category === 'weight'">
        <view class="quick-item" @click="quickConvert('1', '千克', '克')">1千克 = 1000克</view>
        <view class="quick-item" @click="quickConvert('1', '千克', '磅')">1千克 ≈ 2.2磅</view>
        <view class="quick-item" @click="quickConvert('1', '克', '毫克')">1克 = 1000毫克</view>
      </view>
      <view class="quick-list" v-if="category === 'temperature'">
        <view class="quick-item" @click="quickConvert('100', '摄氏度', '华氏度')">100°C = 212°F</view>
        <view class="quick-item" @click="quickConvert('0', '摄氏度', '华氏度')">0°C = 32°F</view>
        <view class="quick-item" @click="quickConvert('37', '摄氏度', '华氏度')">37°C = 98.6°F</view>
      </view>
      <view class="quick-list" v-if="category === 'area'">
        <view class="quick-item" @click="quickConvert('100', '平方米', '平方英尺')">100㎡ ≈ 1076平方英尺</view>
        <view class="quick-item" @click="quickConvert('1', '公顷', '平方米')">1公顷 = 10000㎡</view>
        <view class="quick-item" @click="quickConvert('1', '亩', '平方米')">1亩 ≈ 666.67㎡</view>
        <view class="quick-item" @click="quickConvert('1', '平方公里', '平方米')">1平方公里 = 1000000㎡</view>
        <view class="quick-item" @click="quickConvert('1', '平方英尺', '平方英寸')">1平方英尺 = 144平方英寸</view>
        <view class="quick-item" @click="quickConvert('1', '英亩', '平方米')">1英亩 ≈ 4046.9㎡</view>
      </view>
      <view class="quick-list" v-if="category === 'volume'">
        <view class="quick-item" @click="quickConvert('1', '升', '毫升')">1升 = 1000毫升</view>
        <view class="quick-item" @click="quickConvert('1', '立方米', '升')">1立方米 = 1000升</view>
        <view class="quick-item" @click="quickConvert('1', '加仑', '升')">1加仑 ≈ 3.79升</view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      category: 'length',
      inputValue: '',
      fromIndex: 0,
      toIndex: 1,
      result: '0',
      units: {
        length: ['米', '厘米', '毫米', '分米', '公里', '英里', '英寸', '英尺', '码'],
        weight: ['千克', '克', '毫克', '吨', '磅', '盎司', '克拉'],
        temperature: ['摄氏度', '华氏度', '开尔文'],
        area: ['平方米', '平方厘米', '平方毫米', '公顷', '平方千米', '亩', '平方英尺', '平方码', '英亩'],
        volume: ['升', '毫升', '立方米', '立方厘米', '加仑', '夸脱', '品脱', '杯']
      },
      rates: {
        length: {
          '米': 1,
          '厘米': 100,
          '毫米': 1000,
          '分米': 10,
          '公里': 0.001,
          '英里': 0.000621371,
          '英寸': 39.3700787,
          '英尺': 3.2808399,
          '码': 1.0936133
        },
        weight: {
          '千克': 1,
          '克': 1000,
          '毫克': 1000000,
          '吨': 0.001,
          '磅': 2.20462262,
          '盎司': 35.2739619,
          '克拉': 5000
        },
        area: {
          '平方米': 1,
          '平方厘米': 10000,
          '平方毫米': 1000000,
          '公顷': 0.0001,
          '平方千米': 0.000001,
          '亩': 0.0015,
          '平方英尺': 10.7639104,
          '平方码': 1.19599005,
          '英亩': 0.000247105
        },
        volume: {
          '升': 1,
          '毫升': 1000,
          '立方米': 0.001,
          '立方厘米': 1000,
          '加仑': 0.264172,
          '夸脱': 1.056688,
          '品脱': 2.113376,
          '杯': 4.226754
        }
      }
    }
  },
  methods: {
    convert() {
      if (!this.inputValue || isNaN(parseFloat(this.inputValue))) {
        this.result = '0'
        return
      }
      const val = parseFloat(this.inputValue)
      if (this.category === 'temperature') {
        this.result = this.convertTemperature(val, this.units.temperature[this.fromIndex], this.units.temperature[this.toIndex])
      } else {
        const fromUnit = this.units[this.category][this.fromIndex]
        const toUnit = this.units[this.category][this.toIndex]
        const baseValue = val / this.rates[this.category][fromUnit]
        const result = baseValue * this.rates[this.category][toUnit]
        this.result = this.formatResult(result)
      }
    },
    convertTemperature(val, from, to) {
      let celsius
      if (from === '摄氏度') celsius = val
      else if (from === '华氏度') celsius = (val - 32) * 5 / 9
      else if (from === '开尔文') celsius = val - 273.15

      let result
      if (to === '摄氏度') result = celsius
      else if (to === '华氏度') result = celsius * 9 / 5 + 32
      else if (to === '开尔文') result = celsius + 273.15

      return this.formatResult(result)
    },
    formatResult(num) {
      if (Math.abs(num) < 0.0001 || Math.abs(num) > 999999999) {
        return num.toExponential(4)
      }
      if (Number.isInteger(num)) return num.toString()
      return parseFloat(num.toFixed(8)).toString()
    },
    onFromChange(e) {
      this.fromIndex = e.detail.value
      this.convert()
    },
    onToChange(e) {
      this.toIndex = e.detail.value
      this.convert()
    },
    swap() {
      const tempFrom = this.fromIndex
      this.fromIndex = this.toIndex
      this.toIndex = tempFrom
      this.convert()
    },
    quickConvert(val, from, to) {
      this.inputValue = val
      const cats = ['length', 'weight', 'temperature', 'area', 'volume']
      for (const cat of cats) {
        const fromIdx = this.units[cat].indexOf(from)
        const toIdx = this.units[cat].indexOf(to)
        if (fromIdx !== -1 && toIdx !== -1) {
          this.category = cat
          this.fromIndex = fromIdx
          this.toIndex = toIdx
          break
        }
      }
      this.convert()
    },
    copyResult() {
      if (this.result && this.result !== '0') {
        uni.setClipboardData({
          data: this.result,
          success: () => {
            uni.showToast({ title: '已复制', icon: 'success' })
          }
        })
      }
    }
  }
}
</script>

<style scoped>
.converter-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #e3f2fd 0%, #bbdefb 100%);
  padding: 40rpx 30rpx;
}

.header {
  text-align: center;
  margin-bottom: 40rpx;
}

.title {
  font-size: 48rpx;
  font-weight: bold;
  color: #1976d2;
}

.category-tabs {
  display: flex;
  background: rgba(255, 255, 255, 0.7);
  border-radius: 20rpx;
  padding: 10rpx;
  margin-bottom: 40rpx;
}

.tab {
  flex: 1;
  text-align: center;
  padding: 20rpx 0;
  color: #666;
  font-size: 28rpx;
  border-radius: 15rpx;
}

.tab.active {
  background: #2196f3;
  color: white;
  font-weight: bold;
}

.converter-content {
  background: rgba(255, 255, 255, 0.9);
  border-radius: 30rpx;
  padding: 40rpx;
  margin-bottom: 30rpx;
}

.input-group {
  display: flex;
  align-items: center;
  border: 2rpx solid #e0e0e0;
  border-radius: 20rpx;
  overflow: hidden;
}

.value-input {
  flex: 1;
  padding: 30rpx;
  font-size: 36rpx;
  text-align: left;
}

.result-value {
  color: #2196f3;
  font-weight: bold;
  background: rgba(33, 150, 243, 0.1);
}

.unit-picker {
  background: rgba(255, 255, 255, 0.8);
  padding: 30rpx 20rpx;
  border-left: 2rpx solid #e0e0e0;
}

.picker-value {
  font-size: 28rpx;
  color: #333;
  min-width: 120rpx;
  text-align: center;
}

.swap-btn {
  text-align: center;
  padding: 30rpx 0;
}

.swap-btn text {
  color: #2196f3;
  font-size: 28rpx;
  background: rgba(33, 150, 243, 0.1);
  padding: 15rpx 40rpx;
  border-radius: 30rpx;
}

.copy-tip {
  text-align: center;
  font-size: 24rpx;
  color: #999;
  margin-top: 20rpx;
}

.quick-convert {
  background: rgba(255, 255, 255, 0.8);
  border-radius: 30rpx;
  padding: 30rpx;
}

.quick-title {
  color: #333;
  font-size: 28rpx;
  margin-bottom: 20rpx;
  font-weight: bold;
}

.quick-list {
  display: flex;
  flex-wrap: wrap;
  gap: 20rpx;
}

.quick-item {
  background: rgba(33, 150, 243, 0.1);
  color: #1976d2;
  padding: 20rpx 30rpx;
  border-radius: 30rpx;
  font-size: 24rpx;
}
</style>
