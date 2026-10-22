<template>
  <view class="history-page">
    <view class="empty" v-if="historyList.length === 0">
      <text class="icon">🧳</text>
      <text class="text">你还没有开启过盲盒哦</text>
    </view>

    <view class="history-list" v-else>
      <!-- 🌟 增加点击事件 -->
      <view
        class="history-card"
        v-for="(item, index) in historyList"
        :key="index"
        @click="viewHistory(item)"
      >
        <view class="card-top">
          <text class="mood">{{ item.mood }}</text>
          <text class="date">{{ item.date }}</text>
        </view>
        <text class="dest">{{ item.destination }}</text>
        <text class="preview">{{ item.preview }}...</text>
      </view>
    </view>
  </view>
</template>

<script setup>
import { ref } from 'vue'
import { onShow } from '@dcloudio/uni-app'

const historyList = ref([])

onShow(() => {
  const saved = uni.getStorageSync('travelHistory')
  if (saved) {
    historyList.value = JSON.parse(saved)
  }
})

// 🌟 点击查看详情（把整条数据传回结果页）
const viewHistory = (item) => {
  // 如果没有存详细数据（之前的旧数据），提示一下
  if (!item.planData) {
    return uni.showToast({ title: '旧版本数据，请重新生成', icon: 'none' })
  }

  const dataStr = encodeURIComponent(JSON.stringify(item))
  uni.navigateTo({
    url: `/pages/result/result?dataStr=${dataStr}`
  })
}
</script>

<style>
.history-page { min-height: 100vh; background: #1a1a1a; padding: 40rpx; box-sizing: border-box; }
.empty { display: flex; flex-direction: column; align-items: center; margin-top: 200rpx; }
.icon { font-size: 100rpx; margin-bottom: 20rpx; }
.text { color: rgba(255,255,255,0.5); font-size: 30rpx; }
.history-card { background: rgba(255,255,255,0.1); border-radius: 24rpx; padding: 30rpx; margin-bottom: 30rpx; border: 1px solid rgba(255,255,255,0.1); transition: transform 0.2s; }
.history-card:active { transform: scale(0.98); } /* 点击缩放反馈 */
.card-top { display: flex; justify-content: space-between; margin-bottom: 20rpx; }
.mood { color: #FF8E53; font-size: 24rpx; font-weight: bold; }
.date { color: rgba(255,255,255,0.4); font-size: 24rpx; }
.dest { display: block; color: #fff; font-size: 38rpx; font-weight: bold; margin-bottom: 10rpx; }
.preview { color: rgba(255,255,255,0.6); font-size: 26rpx; display: -webkit-box; -webkit-box-orient: vertical; -webkit-line-clamp: 2; overflow: hidden; }
</style>
