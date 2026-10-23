<template>
  <view class="page-bg">
    <image class="bg-img" src="/static/bg.jpg" mode="aspectFill"></image>
    <view class="bg-mask"></view>

    <view class="content-wrapper">
      <view class="header">
        <text class="brand">Wanderlust</text>
        <text class="title">盲盒旅行</text>
        <text class="subtitle">给灵魂放个假，去哪里交给命运</text>
      </view>

      <view class="glass-card">
        <view class="input-row">
          <text class="label">预算 (元)</text>
          <input type="number" v-model="budget" placeholder="输入金额" placeholder-class="ph" class="input" />
        </view>
        <view class="line"></view>
        <view class="input-row">
          <text class="label">天数</text>
          <input type="number" v-model="days" placeholder="输入天数" placeholder-class="ph" class="input" />
        </view>

        <view class="mood-section">
          <text class="label mb">你想要哪种治愈？</text>
          <view class="moods">
            <view
              v-for="item in moods"
              :key="item"
              class="mood-tag"
              :class="{ active: currentMood === item }"
              @click="currentMood = item"
            >
              {{ item }}
            </view>
          </view>
        </view>
      </view>

      <view class="action-area">
        <button class="magic-btn" @click="openBlindBox">
          <text class="btn-text">开启盲盒</text>
          <text class="btn-icon">✈️</text>
        </button>
      </view>
    </view>
  </view>
</template>

<script setup>
import { ref } from 'vue'

const budget = ref('')
const days = ref('')
const currentMood = ref('去看海')
const moods = ['去看海', '吃遍夜市', '隐居山林', '城市漫游', '特种兵打卡']

const openBlindBox = () => {
  if (!budget.value || !days.value) {
    return uni.showToast({ title: '请补全预算和天数', icon: 'none' })
  }
  uni.navigateTo({
    url: `/pages/result/result?budget=${budget.value}&days=${days.value}&mood=${currentMood.value}`
  })
}
</script>

<style>
/* 和之前一样的唯美样式，这里省略部分，直接复制上面的即可，确保背景图能加载 */
.page-bg { position: relative; min-height: 100vh; background: #1a1a1a; overflow: hidden; padding-bottom: 150rpx; }
.bg-img { position: absolute; top: 0; left: 0; width: 100%; height: 100%; z-index: 1; }
.bg-mask { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: linear-gradient(to bottom, rgba(0,0,0,0.2), rgba(0,0,0,0.8)); z-index: 2; }
.content-wrapper { position: relative; z-index: 3; padding: 120rpx 50rpx 50rpx; display: flex; flex-direction: column; min-height: 100vh; box-sizing: border-box; }
.header { margin-bottom: 60rpx; }
.brand { font-size: 24rpx; color: rgba(255,255,255,0.6); letter-spacing: 4rpx; text-transform: uppercase; }
.title { display: block; font-size: 76rpx; font-weight: 900; color: #ffffff; margin: 10rpx 0; }
.subtitle { font-size: 28rpx; color: rgba(255,255,255,0.8); }
.glass-card { background: rgba(255, 255, 255, 0.15); backdrop-filter: blur(20px); border: 1px solid rgba(255, 255, 255, 0.2); border-radius: 40rpx; padding: 40rpx; margin-bottom: 60rpx; }
.input-row { display: flex; justify-content: space-between; align-items: center; padding: 10rpx 0; }
.label { font-size: 30rpx; color: #ffffff; font-weight: 500; }
.input { text-align: right; font-size: 36rpx; color: #ffffff; font-weight: bold; width: 300rpx; }
.ph { color: rgba(255,255,255,0.4); font-weight: normal; }
.line { height: 1px; background: rgba(255,255,255,0.1); margin: 30rpx 0; }
.mood-section { margin-top: 40rpx; }
.mb { display: block; margin-bottom: 24rpx; }
.moods { display: flex; flex-wrap: wrap; gap: 20rpx; }
.mood-tag { padding: 12rpx 28rpx; background: rgba(255,255,255,0.1); color: rgba(255,255,255,0.8); border-radius: 30rpx; font-size: 26rpx; }
.mood-tag.active { background: #ffffff; color: #000000; font-weight: bold; }
.action-area { margin-top: auto; text-align: center; }
.magic-btn { background: linear-gradient(135deg, #FF6B6B, #FF8E53); border-radius: 60rpx; border: none; padding: 20rpx 0; display: flex; justify-content: center; box-shadow: 0 15rpx 30rpx rgba(255, 107, 107, 0.4); animation: pulse 2s infinite; }
.magic-btn::after { border: none; }
.btn-text { font-size: 34rpx; font-weight: bold; color: white; letter-spacing: 2rpx; }
.btn-icon { margin-left: 10rpx; font-size: 34rpx; }
@keyframes pulse { 0% { transform: scale(1); } 50% { transform: scale(1.02); } 100% { transform: scale(1); } }
</style>
