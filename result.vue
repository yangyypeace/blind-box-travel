<template>
  <view class="page-container">

    <!-- 加载状态（AI 思考时的动画） -->
    <view v-if="isLoading" class="loading-wrapper">
      <view class="globe-animation">🌍</view>
      <text class="loading-title">正在为你探索未知...</text>
      <text class="loading-sub">AI 正在规划行程，请稍候</text>
    </view>

    <!-- 结果展示区 -->
    <view v-else class="result-wrapper">
      <!-- 顶部盲盒揭晓提示 -->
      <view class="reveal-header">
        <text class="reveal-title">盲盒已开启</text>
        <text class="reveal-sub">属于你的 {{ mood }} 之旅</text>
      </view>

      <!-- 路书正文卡片 -->
      <view class="plan-card">
        <text class="plan-text">{{ aiPlan }}</text>
      </view>

      <!-- 底部悬浮按钮 -->
      <view class="floating-actions">
        <button class="action-btn btn-secondary" @click="copyPlan">📋 复制路书</button>
        <button class="action-btn btn-primary" @click="goBack">🔄 换个盲盒</button>
      </view>
    </view>

  </view>
</template>

<script setup>
import { ref } from 'vue'
import { onLoad } from '@dcloudio/uni-app'

const isLoading = ref(true)
const aiPlan = ref('')
const mood = ref('')

// ⚠️⚠️⚠️ 记得换成你真实的 API Key ⚠️⚠️⚠️
const DEEPSEEK_API_KEY = 'sk-这里填你自己的真实密钥'

onLoad((options) => {
  mood.value = options.mood || '惊喜'
  fetchPlanFromAI(options.budget, options.days, options.mood)
})

const fetchPlanFromAI = async (budget, days, mood) => {
  try {
    // 高级提示词工程
    const prompt = `你是一个资深的盲盒旅行策划师。请随机为用户推荐一个国内的宝藏目的地，并为其制定一个${days}天的旅行攻略。
预算限制：${budget}元以内。
偏好风格：${mood}。
输出要求：
1. 格式排版极其整洁，条理清晰。
2. 不要寒暄，直接给干货。
3. 必须包含：目的地（要加粗）、行程亮点（2-3点）、每日行程安排（详细到吃饭和游玩）。`

    const res = await uni.request({
      url: '/api/chat/completions',
      method: 'POST',
      header: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${DEEPSEEK_API_KEY}`
      },
      data: {
        model: 'deepseek-chat',
        messages: [{ role: 'user', content: prompt }]
      }
    })

    if (res.data && res.data.choices) {
      aiPlan.value = res.data.choices[0].message.content
    } else {
      aiPlan.value = 'AI 生成失败，请检查 Key 或余额。'
    }
  } catch (error) {
    aiPlan.value = '网络请求失败，请检查是否配置了代理。'
  } finally {
    isLoading.value = false
  }
}

const copyPlan = () => {
  uni.setClipboardData({
    data: aiPlan.value,
    success: () => uni.showToast({ title: '已复制', icon: 'success' })
  })
}

const goBack = () => {
  uni.navigateBack()
}
</script>

<style>
/* 页面基础 */
.page-container {
  min-height: 100vh;
  background-color: #F8FAFC;
  padding-bottom: 160rpx; /* 给底部按钮留空间 */
}

/* 动画区 */
.loading-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 80vh;
}
.globe-animation {
  font-size: 100rpx;
  animation: spin 4s linear infinite;
  margin-bottom: 40rpx;
}
@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}
.loading-title {
  font-size: 34rpx;
  font-weight: 600;
  color: #1E293B;
  margin-bottom: 12rpx;
}
.loading-sub {
  font-size: 26rpx;
  color: #94A3B8;
}

/* 结果区 */
.result-wrapper {
  padding: 40rpx;
}
.reveal-header {
  text-align: center;
  margin-bottom: 40rpx;
  margin-top: 20rpx;
}
.reveal-title {
  display: block;
  font-size: 44rpx;
  font-weight: 800;
  color: #1E293B;
  margin-bottom: 10rpx;
}
.reveal-sub {
  font-size: 26rpx;
  color: #64748B;
  background: #E2E8F0;
  padding: 6rpx 20rpx;
  border-radius: 20rpx;
}

/* 路书卡片：高级阴影与内边距 */
.plan-card {
  background: #FFFFFF;
  border-radius: 24rpx;
  padding: 40rpx;
  box-shadow: 0 10rpx 40rpx rgba(0,0,0,0.06);
  border: 1px solid #F1F5F9;
}
.plan-text {
  font-size: 30rpx;
  color: #334155;
  line-height: 1.8;
  white-space: pre-wrap; /* 保留 AI 生成的换行 */
}

/* 底部悬浮按钮 */
.floating-actions {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 20rpx 40rpx 60rpx 40rpx;
  background: linear-gradient(to top, #F8FAFC 80%, rgba(248,250,252,0));
  display: flex;
  gap: 20rpx;
}
.action-btn {
  flex: 1;
  border-radius: 50rpx;
  font-size: 30rpx;
  font-weight: 600;
  padding: 10rpx 0;
}
.btn-secondary {
  background-color: #FFFFFF;
  color: #1E293B;
  border: 2rpx solid #E2E8F0;
}
.btn-primary {
  background-color: #1E293B;
  color: #FFFFFF;
  border: none;
}
.action-btn::after {
  border: none;
}
</style>
