<script>
export default {
  globalData: {
    isDark: false
  },
  onLaunch: function() {
    const isDark = uni.getStorageSync('isDarkMode') || false
    this.globalData.isDark = isDark
    this.applyTheme(isDark)
  },
  methods: {
    applyTheme(isDark) {
      if (isDark) {
        uni.setNavigationBarColor({
          frontColor: '#ffffff',
          backgroundColor: '#1a1a2e'
        })
        uni.setTabBarStyle({
          backgroundColor: '#1a1a2e',
          color: '#888888',
          selectedColor: '#2196f3'
        })
      } else {
        uni.setNavigationBarColor({
          frontColor: '#000000',
          backgroundColor: '#e3f2fd'
        })
        uni.setTabBarStyle({
          backgroundColor: '#ffffff',
          color: '#666666',
          selectedColor: '#2196f3'
        })
      }
    },
    toggleTheme() {
      this.globalData.isDark = !this.globalData.isDark
      uni.setStorageSync('isDarkMode', this.globalData.isDark)
      this.applyTheme(this.globalData.isDark)
      uni.$emit('themeChange', this.globalData.isDark)
    }
  }
}
</script>

<style lang="scss">
@import './uni.scss';

page {
  background-color: #f8f8f8;
}

.dark-mode {
  page {
    background-color: #1a1a2e;
  }
  .container {
    background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%) !important;
  }
  .label, .result-label, .result-value, .history-data {
    color: #e0e0e0 !important;
  }
  .input, .textarea, .picker {
    background: rgba(255, 255, 255, 0.1) !important;
    color: #ffffff !important;
  }
  .result-container, .history-panel {
    background: rgba(255, 255, 255, 0.05) !important;
  }
  .history-item {
    border-bottom-color: #333 !important;
  }
  .history-header {
    border-bottom-color: #333 !important;
  }
}
</style>
