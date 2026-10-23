<template>
  <view class="page-bg">
    <image class="bg-img" src="/static/bg.jpg" mode="aspectFill"></image>
    <view class="bg-mask"></view>

    <view class="loading-screen" v-if="isLoading">
      <view class="spinner"></view>
      <text class="load-text">正在探索未知的坐标...</text>
      <text class="load-sub">AI 正在为你规划惊喜</text>
    </view>

    <view class="result-screen" v-else>
      <view class="header-tag"><text class="tag-text">Secret Destination</text></view>

      <view class="postcard">
        <view class="dest-header">
          <text class="dest-title">盲盒已开启</text>
          <view class="divider"></view>
        </view>

        <scroll-view class="scroll-content" scroll-y="true">
          <rich-text :nodes="formattedPlan" class="plan-html"></rich-text>
        </scroll-view>
      </view>

      <view class="bottom-bar">
        <button class="btn-copy" @click="copyPlan">复制路书</button>
        <button class="btn-again" @click="goBack">再抽一次</button>
      </view>
    </view>
  </view>
</template>

<script setup>
import { ref, computed } from 'vue'
import { onLoad } from '@dcloudio/uni-app'

const isLoading = ref(true)
const aiPlan = ref('')
const mood = ref('')

const DEEPSEEK_API_KEY = 'sk-sk-18681990fb48431a904bb99378910a5e'

onLoad((options) => {
  mood.value = options.mood || '惊喜'
  fetchPlan(options.budget, options.days, options.mood)
})

const formattedPlan = computed(() => {
  if (!aiPlan.value) return ''
  let html = aiPlan.value.replace(/\*\*(.*?)\*\*/g, '<span style="font-size:36rpx; font-weight:bold; color:#1E293B; display:block; margin-top:20rpx; margin-bottom:10rpx;">$1</span>')
  html = html.replace(/\n/g, '<br/>')
  return html
})

const fetchPlan = async (budget, days, mood) => {
  try {
    const prompt = `你是一个充满文艺气息的旅行盲盒策划师。请随机推荐一个国内目的地，并制定${days}天攻略。
预算：${budget}元内。风格：${mood}。
要求：
1. 不要废话，直接输出干货。
2. 第一行必须用【地名】单独写出目的地。
3. 采用**加粗**来写每天的行程标题（如 **Day 1：初见大理**）。`

    const res = await uni.request({
      // 注意：这里必须是真实地址，不能是 /api
      url: 'https://api.deepseek.com/chat/completions',
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
      saveToHistory() // AI 生成成功后，自动存入本地历史
    } else {
      aiPlan.value = '生成失败，请检查 Key。'
    }
  } catch (e) {
    aiPlan.value = '网络异常，请重试。'
  } finally {
    isLoading.value = false
  }
}

// 把生成的数据存到手机本地缓存
const saveToHistory = () => {
  let history = uni.getStorageSync('travelHistory') || '[]'
  let list = JSON.parse(history)

  // 提取第一行的地名（去掉星号）
  const firstLine = aiPlan.value.split('\n')[0].replace(/[*#【】]/g, '')

  // 插入到数组最前面
  list.unshift({
    destination: firstLine || '未知目的地',
    mood: mood.value,
    date: new Date().toLocaleDateString(),
    preview: aiPlan.value.substring(firstLine.length, 60)
  })

  // 最多保存 10 条
  if(list.length > 10) list.pop()

  uni.setStorageSync('travelHistory', JSON.stringify(list))
}

const copyPlan = () => {
  uni.setClipboardData({ data: aiPlan.value, success: () => uni.showToast({ title: '已复制', icon: 'none' }) })
}
const goBack = () => uni.navigateBack()
</script>

<style>
/* 这里的样式与之前一致，确保你的滚动、卡片、加载动画好看 */
.page-bg { position: relative; min-height: 100vh; overflow: hidden; background: #1a1a1a; }
.bg-img { position: absolute; top: 0; left: 0; width: 100%; height: 100%; z-index: 1; filter: blur(10px); }
.bg-mask { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.6); z-index: 2; }
.loading-screen { position: relative; z-index: 3; display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100vh; }
.spinner { width: 80rpx; height: 80rpx; border: 4rpx solid rgba(255,255,255,0.2); border-top-color: #FF8E53; border-radius: 50%; animation: spin 1s linear infinite; margin-bottom: 40rpx; }
@keyframes spin { to { transform: rotate(360deg); } }
.load-text { font-size: 34rpx; color: #fff; font-weight: bold; margin-bottom: 10rpx; }
.load-sub { font-size: 26rpx; color: rgba(255,255,255,0.6); }
.result-screen { position: relative; z-index: 3; height: 100vh; display: flex; flex-direction: column; padding: 60rpx 40rpx; box-sizing: border-box; }
.header-tag { align-self: center; border: 1px solid rgba(255,255,255,0.3); padding: 8rpx 30rpx; border-radius: 30rpx; margin-bottom: 30rpx; }
.tag-text { color: #fff; font-size: 22rpx; letter-spacing: 4rpx; }
.postcard { flex: 1; background: #F8FAFC; border-radius: 24rpx; padding: 40rpx; display: flex; flex-direction: column; box-shadow: 0 20rpx 50rpx rgba(0,0,0,0.5); overflow: hidden; }
.dest-header { text-align: center; margin-bottom: 30rpx; }
.dest-title { font-size: 46rpx; font-weight: 900; color: #1E293B; letter-spacing: 2rpx; }
.divider { width: 60rpx; height: 6rpx; background: #FF8E53; margin: 20rpx auto 0; border-radius: 6rpx; }
.scroll-content { flex: 1; overflow-y: auto; padding-bottom: 40rpx; }
.plan-html { font-size: 30rpx; color: #475569; line-height: 1.8; }
.bottom-bar { display: flex; gap: 20rpx; margin-top: 40rpx; }
.btn-copy, .btn-again { flex: 1; border-radius: 50rpx; font-size: 30rpx; font-weight: bold; padding: 10rpx 0; border: none; }
.btn-copy { background: rgba(255,255,255,0.15); color: #fff; border: 1px solid rgba(255,255,255,0.3); }
.btn-again { background: #FF8E53; color: #fff; }
</style>
