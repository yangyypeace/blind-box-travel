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
        <view class="line"></view>

        <!-- 省份筛选 -->
        <view class="input-row">
          <text class="label">省份 (可选)</text>
          <picker mode="selector" :range="provinces" @change="onProvinceChange">
            <view class="picker-text" :class="{ 'empty-text': !selectedProvince }">
              {{ selectedProvince || '随机全中国' }} ▼
            </view>
          </picker>
        </view>

        <!-- 城市筛选 -->
        <view class="input-row" v-if="selectedProvince && selectedProvince !== '随机全中国'">
          <text class="label">城市 (可选)</text>
          <picker mode="selector" :range="cities" @change="onCityChange">
            <view class="picker-text" :class="{ 'empty-text': !selectedCity }">
              {{ selectedCity || '不限，该省内随机' }} ▼
            </view>
          </picker>
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

      <view class="inspiration-section">
        <text class="insp-title">✨ 灵感目的地</text>
        <scroll-view class="insp-scroll" scroll-x="true">
          <view class="insp-list">
            <view class="insp-card" v-for="(item, index) in inspirations" :key="index">
              <image :src="item.img" mode="aspectFill" class="insp-img"></image>
              <view class="insp-mask"></view>
              <text class="insp-name">{{ item.name }}</text>
            </view>
          </view>
        </scroll-view>
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

// 全国省级
const provinces = [
  '随机全中国', '北京', '天津', '上海', '重庆', '河北', '山西', '辽宁', '吉林', '黑龙江',
  '江苏', '浙江', '安徽', '福建', '江西', '山东', '河南', '湖北', '湖南', '广东',
  '海南', '四川', '贵州', '云南', '陕西', '甘肃', '青海', '台湾', '内蒙古', '广西',
  '西藏', '宁夏', '新疆', '香港', '澳门'
]
const selectedProvince = ref('随机全中国')
const selectedCity = ref('')
const cities = ref(['不限'])

// 🌟 本地省市大字典（绝对精准）
const cityData = {
  '北京': ['东城', '西城', '朝阳', '丰台', '石景山', '海淀', '门头沟', '房山', '通州', '顺义', '昌平', '大兴', '怀柔', '平谷', '密云', '延庆'],
  '天津': ['和平', '河东', '河西', '南开', '河北', '红桥', '东丽', '西青', '津南', '北辰', '武清', '宝坻', '滨海新区', '宁河', '静海', '蓟州'],
  '上海': ['黄浦', '徐汇', '长宁', '静安', '普陀', '虹口', '杨浦', '闵行', '宝山', '嘉定', '浦东新区', '金山', '松江', '青浦', '奉贤', '崇明'],
  '重庆': ['万州', '涪陵', '渝中', '大渡口', '江北', '沙坪坝', '九龙坡', '南岸', '北碚', '綦江', '大足', '渝北', '巴南', '黔江', '长寿', '江津', '合川', '永川', '南川', '璧山', '铜梁', '潼南', '荣昌', '开州', '梁平', '武隆', '城口', '丰都', '垫江', '忠县', '云阳', '奉节', '巫山', '巫溪', '石柱', '秀山', '酉阳', '彭水'],
  '广东': ['广州', '韶关', '深圳', '珠海', '汕头', '佛山', '江门', '湛江', '茂名', '肇庆', '惠州', '梅州', '汕尾', '河源', '阳江', '清远', '东莞', '中山', '潮州', '揭阳', '云浮'],
  '福建': ['福州', '厦门', '莆田', '三明', '泉州', '漳州', '南平', '龙岩', '宁德'],
  '浙江': ['杭州', '宁波', '温州', '嘉兴', '湖州', '绍兴', '金华', '衢州', '舟山', '台州', '丽水'],
  '江苏': ['南京', '无锡', '徐州', '常州', '苏州', '南通', '连云港', '淮安', '盐城', '扬州', '镇江', '泰州', '宿迁'],
  '四川': ['成都', '自贡', '攀枝花', '泸州', '德阳', '绵阳', '广元', '遂宁', '内江', '乐山', '南充', '眉山', '宜宾', '广安', '达州', '雅安', '巴中', '资阳', '阿坝', '甘孜', '凉山'],
  '云南': ['昆明', '曲靖', '玉溪', '保山', '昭通', '丽江', '普洱', '临沧', '楚雄', '红河', '文山', '西双版纳', '大理', '德宏', '怒江', '迪庆'],
  '贵州': ['贵阳', '六盘水', '遵义', '安顺', '毕节', '铜仁', '黔西南', '黔东南', '黔南'],
  '海南': ['海口', '三亚', '三沙', '儋州', '五指山', '琼海', '文昌', '万宁', '东方', '定安', '屯昌', '澄迈', '临高', '白沙', '昌江', '乐东', '陵水', '保亭', '琼中'],
  '河北': ['石家庄', '唐山', '秦皇岛', '邯郸', '邢台', '保定', '张家口', '承德', '沧州', '廊坊', '衡水'],
  '山西': ['太原', '大同', '阳泉', '长治', '晋城', '朔州', '晋中', '运城', '忻州', '临汾', '吕梁'],
  '辽宁': ['沈阳', '大连', '鞍山', '抚顺', '本溪', '丹东', '锦州', '营口', '阜新', '辽阳', '盘锦', '铁岭', '朝阳', '葫芦岛'],
  '吉林': ['长春', '吉林', '四平', '辽源', '通化', '白山', '松原', '白城', '延边'],
  '黑龙江': ['哈尔滨', '齐齐哈尔', '鸡西', '鹤岗', '双鸭山', '大庆', '伊春', '佳木斯', '七台河', '牡丹江', '黑河', '绥化', '大兴安岭'],
  '安徽': ['合肥', '芜湖', '蚌埠', '淮南', '马鞍山', '淮北', '铜陵', '安庆', '黄山', '滁州', '阜阳', '宿州', '六安', '亳州', '池州', '宣城'],
  '江西': ['南昌', '景德镇', '萍乡', '九江', '新余', '鹰潭', '赣州', '吉安', '宜春', '抚州', '上饶'],
  '山东': ['济南', '青岛', '淄博', '枣庄', '东营', '烟台', '潍坊', '济宁', '泰安', '威海', '日照', '临沂', '德州', '聊城', '滨州', '菏泽'],
  '河南': ['郑州', '开封', '洛阳', '平顶山', '安阳', '鹤壁', '新乡', '焦作', '濮阳', '许昌', '漯河', '三门峡', '南阳', '商丘', '信阳', '周口', '驻马店'],
  '湖北': ['武汉', '黄石', '十堰', '宜昌', '襄阳', '鄂州', '荆门', '孝感', '荆州', '黄冈', '咸宁', '随州', '恩施', '仙桃', '潜江', '天门', '神农架'],
  '湖南': ['长沙', '株洲', '湘潭', '衡阳', '邵阳', '岳阳', '常德', '张家界', '益阳', '郴州', '永州', '怀化', '娄底', '湘西'],
  '陕西': ['西安', '铜川', '宝鸡', '咸阳', '渭南', '延安', '汉中', '榆林', '安康', '商洛'],
  '甘肃': ['兰州', '嘉峪关', '金昌', '白银', '天水', '武威', '张掖', '平凉', '酒泉', '庆阳', '定西', '陇南', '临夏', '甘南'],
  '青海': ['西宁', '海东', '海北', '黄南', '海南', '果洛', '玉树', '海西'],
  '内蒙古': ['呼和浩特', '包头', '乌海', '赤峰', '通辽', '鄂尔多斯', '呼伦贝尔', '巴彦淖尔', '乌兰察布', '兴安盟', '锡林郭勒', '阿拉善'],
  '广西': ['南宁', '柳州', '桂林', '梧州', '北海', '防城港', '钦州', '贵港', '玉林', '百色', '贺州', '河池', '来宾', '崇左'],
  '西藏': ['拉萨', '日喀则', '昌都', '林芝', '山南', '那曲', '阿里'],
  '宁夏': ['银川', '石嘴山', '吴忠', '固原', '中卫'],
  '新疆': ['乌鲁木齐', '克拉玛依', '吐鲁番', '哈密', '昌吉', '博尔塔拉', '巴音郭楞', '阿克苏', '克孜勒苏', '喀什', '和田', '伊犁', '塔城', '阿勒泰'],
  '台湾': ['台北', '新北', '桃园', '台中', '台南', '高雄', '基隆', '新竹', '嘉义'],
  '香港': ['香港岛', '九龙', '新界'],
  '澳门': ['澳门半岛', '氹仔', '路环']
}

const onProvinceChange = (e) => {
  selectedProvince.value = provinces[e.detail.value]
  selectedCity.value = '' // 重置城市

  if (selectedProvince.value !== '随机全中国') {
    const list = cityData[selectedProvince.value] || []
    cities.value = ['不限', ...list]
  }
}

const onCityChange = (e) => {
  selectedCity.value = cities.value[e.detail.value]
}

// 必应搜图
const allInspirations = [
  { name: '大理', img: 'https://tse1.mm.bing.net/th?q=' + encodeURIComponent('大理 风景') + '&w=300&h=400' },
  { name: '川西', img: 'https://tse1.mm.bing.net/th?q=' + encodeURIComponent('川西 风景') + '&w=300&h=400' },
  { name: '厦门', img: 'https://tse1.mm.bing.net/th?q=' + encodeURIComponent('厦门 风景') + '&w=300&h=400' },
  { name: '新疆', img: 'https://tse1.mm.bing.net/th?q=' + encodeURIComponent('新疆 风景') + '&w=300&h=400' },
  { name: '西藏', img: 'https://tse1.mm.bing.net/th?q=' + encodeURIComponent('西藏 风景') + '&w=300&h=400' }
]
const inspirations = ref(allInspirations.sort(() => 0.5 - Math.random()).slice(0, 3))

const openBlindBox = () => {
  if (!budget.value || !days.value) {
    return uni.showToast({ title: '请补全预算和天数', icon: 'none' })
  }

  let areaParam = '随机全中国'
  if (selectedProvince.value !== '随机全中国') {
    if (selectedCity.value && selectedCity.value !== '不限') {
      areaParam = selectedProvince.value + '-' + selectedCity.value
    } else {
      areaParam = selectedProvince.value
    }
  }

  uni.navigateTo({
    url: `/pages/result/result?budget=${budget.value}&days=${days.value}&mood=${currentMood.value}&area=${areaParam}`
  })
}
</script>

<style>
/* 基础样式 */
.page-bg { position: relative; min-height: 100vh; background: #1a1a1a; overflow: hidden; padding-bottom: 50rpx; }
.bg-img { position: absolute; top: 0; left: 0; width: 100%; height: 100%; z-index: 1; }
.bg-mask { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: linear-gradient(to bottom, rgba(0,0,0,0.2), rgba(0,0,0,0.8)); z-index: 2; }
.content-wrapper { position: relative; z-index: 3; padding: 100rpx 40rpx 40rpx; display: flex; flex-direction: column; min-height: 100vh; box-sizing: border-box; }
.header { margin-bottom: 40rpx; }
.brand { font-size: 24rpx; color: rgba(255,255,255,0.6); letter-spacing: 4rpx; text-transform: uppercase; }
.title { display: block; font-size: 70rpx; font-weight: 900; color: #ffffff; margin: 10rpx 0; }
.subtitle { font-size: 26rpx; color: rgba(255,255,255,0.8); }
.glass-card { background: rgba(255, 255, 255, 0.15); backdrop-filter: blur(20px); border: 1px solid rgba(255, 255, 255, 0.2); border-radius: 40rpx; padding: 30rpx; margin-bottom: 30rpx; }
.input-row { display: flex; justify-content: space-between; align-items: center; padding: 10rpx 0; }
.label { font-size: 30rpx; color: #ffffff; font-weight: 500; }
.input { text-align: right; font-size: 36rpx; color: #ffffff; font-weight: bold; width: 300rpx; }
.ph { color: rgba(255,255,255,0.4); font-weight: normal; }
.picker-text { color: #ffffff; font-size: 30rpx; font-weight: bold; }
.empty-text { color: rgba(255,255,255,0.5); font-weight: normal; }
.line { height: 1px; background: rgba(255,255,255,0.1); margin: 20rpx 0; }
.mood-section { margin-top: 30rpx; }
.mb { display: block; margin-bottom: 20rpx; }
.moods { display: flex; flex-wrap: wrap; gap: 15rpx; }
.mood-tag { padding: 10rpx 24rpx; background: rgba(255,255,255,0.1); color: rgba(255,255,255,0.8); border-radius: 30rpx; font-size: 24rpx; }
.mood-tag.active { background: #ffffff; color: #000000; font-weight: bold; }
.inspiration-section { margin-bottom: 40rpx; }
.insp-title { display: block; color: rgba(255,255,255,0.8); font-size: 28rpx; margin-bottom: 20rpx; font-weight: 500; }
.insp-scroll { width: 100%; white-space: nowrap; }
.insp-list { display: flex; gap: 20rpx; padding-bottom: 10rpx; }
.insp-card { position: relative; width: 200rpx; height: 260rpx; border-radius: 20rpx; overflow: hidden; flex-shrink: 0; box-shadow: 0 10rpx 20rpx rgba(0,0,0,0.3); }
.insp-img { width: 100%; height: 100%; display: block; }
.insp-mask { position: absolute; bottom: 0; left: 0; width: 100%; height: 50%; background: linear-gradient(to top, rgba(0,0,0,0.8), transparent); }
.insp-name { position: absolute; bottom: 15rpx; left: 20rpx; color: #fff; font-size: 26rpx; font-weight: bold; letter-spacing: 2rpx; }
.action-area { margin-top: auto; text-align: center; padding-top: 20rpx; }
.magic-btn { background: linear-gradient(135deg, #FF6B6B, #FF8E53); border-radius: 60rpx; border: none; padding: 20rpx 0; display: flex; justify-content: center; box-shadow: 0 15rpx 30rpx rgba(255, 107, 107, 0.4); animation: pulse 2s infinite; }
.magic-btn::after { border: none; }
.btn-text { font-size: 34rpx; font-weight: bold; color: white; letter-spacing: 2rpx; }
.btn-icon { margin-left: 10rpx; font-size: 34rpx; }
@keyframes pulse { 0% { transform: scale(1); } 50% { transform: scale(1.02); } 100% { transform: scale(1); } }
</style>
