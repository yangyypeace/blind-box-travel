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
        <!-- 图片充满 -->
        <view class="img-wrapper">
          <image class="postcard-header-img" :src="coverImage" mode="aspectFill"></image>
        </view>

        <view class="dest-header">
          <text class="dest-title">盲盒已开启</text>
          <view class="divider"></view>
        </view>

        <scroll-view class="scroll-content" scroll-y="true">
          <view class="dest-name" v-if="planData.destination">
            {{ planData.destination }}
          </view>

          <view class="day-block" v-for="(day, index) in planData.itinerary" :key="index">
            <view class="day-title">
              <text class="day-icon">📍</text>
              <text class="day-text">{{ day.dayTitle }}</text>
            </view>

            <view class="time-line">
              <view class="trip-item" v-for="(item, i) in day.items" :key="i">
                <view class="dot"></view>
                <view class="item-content">
                  <text class="item-time">{{ item.time }}</text>
                  <text class="item-desc">{{ item.desc }}</text>
                </view>
              </view>
            </view>
          </view>
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
import { ref } from 'vue'
import { onLoad } from '@dcloudio/uni-app'

const isLoading = ref(true)
const mood = ref('')
const coverImage = ref('https://tse1.mm.bing.net/th?q=' + encodeURIComponent('旅行 风景') + '&w=600&h=300')

const planData = ref({
  destination: '',
  itinerary: []
})

// ⚠️ 填入你的真实 API Key
const DEEPSEEK_API_KEY = 'sk-18681990fb48431a904bb99378910a5e'

onLoad((options) => {
  // 如果是历史记录点进来的
  if (options.dataStr) {
    const historyData = JSON.parse(decodeURIComponent(options.dataStr))
    planData.value = historyData.planData
    coverImage.value = historyData.coverImage
    isLoading.value = false
    return
  }

  mood.value = options.mood || '惊喜'

  // 🌟 解析首页传过来的 area 参数
  let areaRule = ''
  if (options.area && options.area !== '随机全中国') {
    if (options.area.includes('-')) {
      // 用户选了省和市 (例如: 广东-广州)
      const parts = options.area.split('-')
      areaRule = `必须在【${parts[0]}省·${parts[1]}市】范围内`
    } else {
      // 用户只选了省 (例如: 广东)
      areaRule = `必须在【${options.area}省】范围内，请随机挑选一个地级市或县区`
    }
  } else {
    // 用户选了随机全中国
    areaRule = `请随机推荐全中国范围内的小众目的地（精确定位到地级市或县区）`
  }

  fetchPlan(options.budget, options.days, mood.value, areaRule)
})

const fetchPlan = async (budget, days, mood, areaRule) => {
  try {
    const prompt = `你是一个旅行策划师。${areaRule}。请制定${days}天攻略，预算${budget}元，风格${mood}。
请严格输出以下JSON格式（不要输出任何其他多余文字，不要带\`\`\`json这种代码块标签）：
{
  "destination": "【广东省·汕头市】",
  "itinerary": [
    {
      "dayTitle": "Day 1：南澳岛环岛追风",
      "items": [
        { "time": "08:00", "desc": "🚌 交通：乘车前往南澳岛" }
      ]
    }
  ]
}`

    const res = await uni.request({
      url: 'https://api.deepseek.com/chat/completions',
      method: 'POST',
      header: { 'Content-Type': 'application/json', 'Authorization': `Bearer ${DEEPSEEK_API_KEY}` },
      data: { model: 'deepseek-chat', response_format: { type: 'json_object' }, messages: [{ role: 'user', content: prompt }] }
    })

    if (res.data && res.data.choices) {
      const parsedData = JSON.parse(res.data.choices[0].message.content)
      planData.value = parsedData

      // 精准提取地名搜图
      let searchKeyword = parsedData.destination.replace(/[【】\[\]]/g, '')
      if (searchKeyword.includes('·')) {
        searchKeyword = searchKeyword.split('·')[1]
      }

      coverImage.value = 'https://tse1.mm.bing.net/th?q=' + encodeURIComponent(searchKeyword + ' 风景') + '&w=600&h=300'

      saveToHistory(parsedData)
    }
  } catch (e) {
    console.error(e)
  } finally {
    isLoading.value = false
  }
}

const saveToHistory = (data) => {
  let history = uni.getStorageSync('travelHistory') || '[]'
  let list = JSON.parse(history)
  list.unshift({
    destination: data.destination || '未知目的地',
    mood: mood.value,
    date: new Date().toLocaleDateString(),
    coverImage: coverImage.value,
    planData: data
  })
  if(list.length > 10) list.pop()
  uni.setStorageSync('travelHistory', JSON.stringify(list))
}

const copyPlan = () => {
  let textToCopy = planData.value.destination + '\n\n'
  planData.value.itinerary.forEach(day => {
    textToCopy += `【${day.dayTitle}】\n`
    day.items.forEach(item => textToCopy += `${item.time} ${item.desc}\n`)
    textToCopy += '\n'
  })
  uni.setClipboardData({ data: textToCopy, success: () => uni.showToast({ title: '已复制', icon: 'none' }) })
}

const goBack = () => uni.navigateBack()
</script>

<style>
/* 样式保持不变 */
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
.postcard { flex: 1; background: #F8FAFC; border-radius: 24rpx; padding: 30rpx; display: flex; flex-direction: column; box-shadow: 0 20rpx 50rpx rgba(0,0,0,0.5); overflow: hidden; margin-bottom: 20rpx; }

.img-wrapper {
  width: 100%;
  height: 200rpx;
  border-radius: 16rpx;
  overflow: hidden;
  margin-bottom: 20rpx;
  box-shadow: 0 4rpx 10rpx rgba(0,0,0,0.1);
}
.postcard-header-img { width: 100%; height: 100%; display: block; }

.dest-header { text-align: center; margin-bottom: 20rpx; }
.dest-title { font-size: 40rpx; font-weight: 900; color: #1E293B; letter-spacing: 2rpx; }
.divider { width: 60rpx; height: 6rpx; background: #FF8E53; margin: 15rpx auto 0; border-radius: 6rpx; }
Result
.scroll-content { flex: 1; overflow-y: auto; padding-bottom: 20rpx; }
::-webkit-scrollbar { display: none; width: 0 !important; height: 0 !important; -webkit-appearance: none; background: transparent; }

.dest-name { font-size: 40rpx; font-weight: 900; color: #1E293B; text-align: center; margin: 10rpx 0 30rpx 0; letter-spacing: 2rpx; }
.day-block { margin-bottom: 40rpx; }
.day-title { display: flex; align-items: center; background: #FFF7ED; padding: 12rpx 20rpx; border-radius: 12rpx; margin-bottom: 20rpx; }
.day-icon { margin-right: 10rpx; }
.day-text { font-size: 32rpx; font-weight: bold; color: #EA580C; }
.time-line { padding-left: 20rpx; }
.trip-item { display: flex; margin-bottom: 24rpx; position: relative; }
.dot { width: 14rpx; height: 14rpx; background-color: #FF8E53; border-radius: 50%; margin-top: 12rpx; margin-right: 20rpx; flex-shrink: 0; position: relative; z-index: 2; }
.trip-item::before { content: ''; position: absolute; left: 6rpx; top: 26rpx; bottom: -30rpx; width: 2rpx; background-color: #E2E8F0; z-index: 1; }
.trip-item:last-child::before { display: none; }
.item-content { flex: 1; display: flex; flex-direction: column; }
.item-time { font-size: 24rpx; color: #94A3B8; font-weight: bold; margin-bottom: 6rpx; }
.item-desc { font-size: 30rpx; color: #334155; line-height: 1.5; }

.bottom-bar { display: flex; gap: 20rpx; margin-top: 10rpx; }
.btn-copy, .btn-again { flex: 1; border-radius: 50rpx; font-size: 30rpx; font-weight: bold; padding: 10rpx 0; border: none; }
.btn-copy { background: rgba(255,255,255,0.15); color: #fff; border: 1px solid rgba(255,255,255,0.3); }
.btn-again { background: #FF8E53; color: #fff; }
</style>
