<template>
  <view class="page-container">
    <!-- 顶部标题区 -->
    <view class="header-section">
      <text class="main-title">盲盒旅行</text>
      <text class="sub-title">给疲惫的生活，抽个上上签</text>
    </view>

    <!-- 核心设置卡片 -->
    <view class="setting-card">

      <!-- 预算输入 -->
      <view class="input-group">
        <view class="label-box">
          <text class="icon">💰</text>
          <text class="label">预算</text>
        </view>
        <view class="input-box">
          <input type="number" v-model="budget" placeholder="请输入金额" placeholder-class="ph-color" />
          <text class="unit">元</text>
        </view>
      </view>

      <view class="divider"></view>

      <!-- 天数输入 -->
      <view class="input-group">
        <view class="label-box">
          <text class="icon">📅</text>
          <text class="label">天数</text>
        </view>
        <view class="input-box">
          <input type="number" v-model="days" placeholder="请输入天数" placeholder-class="ph-color" />
          <text class="unit">天</text>
        </view>
      </view>

      <view class="divider"></view>

      <!-- 心情选择区 -->
      <view class="mood-section">
        <view class="label-box mb-20">
          <text class="icon">✨</text>
          <text class="label">期待的心情</text>
        </view>
        <view class="tags-container">
          <view
            class="tag"
            :class="{ 'tag-active': currentMood === item }"
            v-for="(item, index) in moodList"
            :key="index"
            @click="currentMood = item"
          >
            {{ item }}
          </view>
        </view>
      </view>

    </view>

    <!-- 底部操作区 -->
    <view class="action-section">
      <button class="submit-btn" @click="generatePlan">
        开启盲盒 🎲
      </button>
      <text class="tips">每次生成的目的地和路线均不相同</text>
    </view>

  </view>
</template>

<script setup>
import { ref } from 'vue'

// 表单数据
const budget = ref('')
const days = ref('')
const currentMood = ref('看日落')

// 心情标签数据
const moodList = [
  '看日落', '吃美食', '躺平放松',
  '特种兵', '人文历史', '自然风光'
]

// 点击生成
const generatePlan = () => {
  if (!budget.value) {
    return uni.showToast({ title: '请填写预算', icon: 'none' })
  }
  if (!days.value) {
    return uni.showToast({ title: '请填写天数', icon: 'none' })
  }

  // 跳转到结果页（下一节课做），并传递参数
  uni.navigateTo({
    url: `/pages/result/result?budget=${budget.value}&days=${days.value}&mood=${currentMood.value}`
  })
}
</script>

<style>
/* 基础页面背景 */
.page-container {
  min-height: 100vh;
  background-color: #F8FAFC; /* 高级浅灰蓝 */
  padding: 60rpx 40rpx;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
}

/* 顶部标题 */
.header-section {
  margin-bottom: 60rpx;
  margin-top: 20rpx;
}
.main-title {
  font-size: 64rpx;
  font-weight: 800;
  color: #1E293B;
  letter-spacing: 2rpx;
  display: block;
  margin-bottom: 16rpx;
}
.sub-title {
  font-size: 28rpx;
  color: #94A3B8;
  letter-spacing: 1rpx;
}

/* 卡片容器 */
.setting-card {
  background-color: #FFFFFF;
  border-radius: 32rpx;
  padding: 40rpx;
  box-shadow: 0 12rpx 40rpx rgba(30, 41, 59, 0.04);
  margin-bottom: 60rpx;
}

/* 输入行 */
.input-group {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10rpx 0;
}
.label-box {
  display: flex;
  align-items: center;
}
.mb-20 {
  margin-bottom: 24rpx;
}
.icon {
  font-size: 36rpx;
  margin-right: 16rpx;
}
.label {
  font-size: 32rpx;
  font-weight: 600;
  color: #334155;
}
.input-box {
  display: flex;
  align-items: center;
  background-color: #F1F5F9;
  padding: 16rpx 24rpx;
  border-radius: 16rpx;
  width: 240rpx;
}
.input-box input {
  flex: 1;
  font-size: 30rpx;
  color: #1E293B;
  text-align: right;
  font-weight: bold;
}
.unit {
  font-size: 26rpx;
  color: #64748B;
  margin-left: 12rpx;
}
.ph-color {
  color: #CBD5E1;
  font-weight: normal;
}

/* 分割线 */
.divider {
  height: 1px;
  background-color: #F1F5F9;
  margin: 30rpx 0;
}

/* 心情标签 */
.tags-container {
  display: flex;
  flex-wrap: wrap;
  gap: 20rpx;
}
.tag {
  padding: 16rpx 32rpx;
  background-color: #F8FAFC;
  border: 1px solid #E2E8F0;
  border-radius: 40rpx;
  font-size: 26rpx;
  color: #64748B;
  transition: all 0.2s ease;
}
.tag-active {
  background-color: #1E293B;
  color: #FFFFFF;
  border-color: #1E293B;
  font-weight: 500;
  box-shadow: 0 4rpx 12rpx rgba(30, 41, 59, 0.2);
}

/* 底部按钮 */
.action-section {
  margin-top: auto; /* 把按钮推到最下面 */
  display: flex;
  flex-direction: column;
  align-items: center;
}
.submit-btn {
  width: 100%;
  background-color: #1E293B; /* 深邃蓝 */
  color: #FFFFFF;
  border-radius: 50rpx;
  font-size: 34rpx;
  font-weight: 600;
  padding: 12rpx 0;
  margin-bottom: 24rpx;
  box-shadow: 0 10rpx 24rpx rgba(30, 41, 59, 0.2);
  border: none;
}
.submit-btn::after {
  border: none;
}
.tips {
  font-size: 24rpx;
  color: #94A3B8;
}
</style>
