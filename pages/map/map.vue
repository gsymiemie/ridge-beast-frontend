<template>
  <view class="map-container">
    
    <map 
      class="main-map"
      id="beastMap"
      :latitude="centerLat"
      :longitude="centerLng"
      :scale="16"
      :markers="markerList"
      @markertap="onMarkerTap"
      @tap="closeCard"
    ></map>

    <view class="map-mask"></view>

    <view class="floating-plaque">
      <view class="plaque-inner">
        <text class="frame-title">御览 · 寻踪纪</text>
        <view class="frame-line"></view>
        <text class="frame-subtitle">按图索骥 点亮神兽</text>
      </view>
    </view>

    <view class="info-card" :class="{'card-show': showCard}">
      <view class="card-inner">
        <view class="img-wrapper">
          <image :src="currentBeast.img" mode="aspectFill" class="beast-img"></image>
        </view>
        
        <view class="beast-info">
          <view class="info-header">
            <text class="beast-name">{{ currentBeast.name }}</text>
            <view class="status-tag" :class="{'tag-done': currentBeast.isCheckIn}">
              <text>{{ currentBeast.isCheckIn ? '福瑞已至' : '未寻踪' }}</text>
            </view>
          </view>
          <text class="beast-location">📍 {{ currentBeast.location }}</text>
          <text class="beast-desc">{{ currentBeast.desc }}</text>
          
          <view class="btn-row">
            <view class="checkin-btn" :class="{'btn-disabled': currentBeast.isCheckIn}" @click="doCheckIn">
              <text>{{ currentBeast.isCheckIn ? '已点亮' : '祈福打卡' }}</text>
            </view>
          </view>
        </view>
      </view>
    </view>

    <view class="custom-tabbar">
      <view class="bar-item" @click="navTo('index')"><text class="icon">🌸</text><text class="text">初遇</text></view>
      <view class="bar-item" @click="navTo('humanities')"><text class="icon">📜</text><text class="text">人文</text></view>
      <view class="bar-item active-bar"><text class="icon">🗺️</text><text class="text">畅游</text></view>
      <view class="bar-item" @click="navTo('square')"><text class="icon">🌊</text><text class="text">潮玩</text></view>
      <view class="bar-item" @click="navTo('mine')"><text class="icon">👤</text><text class="text">我的</text></view>
    </view>
    
  </view>
</template>

<script setup>
import { ref } from 'vue'

const centerLat = ref(39.916344)
const centerLng = ref(116.397155)
const showCard = ref(false)
const currentBeast = ref({})

const markerList = ref([
  {
    id: 1,
    latitude: 39.916344,
    longitude: 116.397155,
    iconPath: '/static/images/ailogo.png', // ⚠️ 这里强烈建议换成一张背景透明的“金色图钉/古建” PNG 图！
    width: 36, // 稍微放大一点标点
    height: 36,
    beastData: { name: '嘲 风', location: '太和殿西北角', desc: '平生好险，威慑邪祟。', img: '/static/images/chaofeng.jpg', isCheckIn: false }
  },
  {
    id: 2,
    latitude: 39.915500,
    longitude: 116.397000,
    iconPath: '/static/images/ailogo.png',
    width: 36,
    height: 36,
    beastData: { name: '螭 吻', location: '保和殿正脊两端', desc: '口润嗓粗，避火防灾。', img: '/static/images/chiwen.jpg', isCheckIn: true }
  }
])

const onMarkerTap = (e) => {
  const marker = markerList.value.find(m => m.id === e.detail.markerId)
  if (marker) {
    currentBeast.value = marker.beastData
    showCard.value = true
  }
}

const closeCard = () => { showCard.value = false }

const doCheckIn = () => {
  if (currentBeast.value.isCheckIn) return
  uni.showLoading({ title: '祈福中...' })
  setTimeout(() => {
    uni.hideLoading()
    currentBeast.value.isCheckIn = true
    uni.showToast({ title: '点亮成功！瑞气+20', icon: 'success' })
  }, 1000)
}

const navTo = (page) => {
  if (page === 'index') uni.redirectTo({ url: '/pages/index/index' })
  if (page === 'humanities') uni.redirectTo({ url: '/pages/humanities/humanities' })
  if (page === 'square') uni.navigateTo({ url: '/pages/square/square' })
  if (page === 'mine') uni.redirectTo({ url: '/pages/mine/mine' })
}
</script>

<style>
page { margin: 0; padding: 0; background-color: #160e08; height: 100%; }
.map-container { width: 100vw; height: 100vh; position: relative; overflow: hidden; }

/* 1. 地图全屏沉浸式 */
.main-map { position: absolute; top: 0; left: 0; width: 100%; height: 100%; z-index: 1; }

/* 🌟 2. 核心滤镜魔法 (边缘暗化，中心泛黄，自带古画光影) */
/* 🌟 修改后的复古滤镜遮罩 (大幅降低黑度，提升通透感) */
.map-mask {
  position: absolute; top: 0; left: 0; width: 100%; height: 100%;
  /* 魔法解析：中心 40% 完全透明，边缘过渡到极淡的金黄色，最外圈用柔和的深棕色收边 */
  background: radial-gradient(circle at center, transparent 40%, rgba(212, 175, 55, 0.08) 75%, rgba(44, 29, 17, 0.5) 100%);
  pointer-events: none; 
  z-index: 5;
}

/* 3. 悬浮的古风牌匾 */
.floating-plaque {
  position: absolute; top: 80rpx; left: 50%; transform: translateX(-50%);
  width: 400rpx; z-index: 10;
  background: rgba(44, 29, 17, 0.85); /* 半透明木质底色 */
  border: 2rpx solid #d4af37; border-radius: 12rpx;
  box-shadow: 0 8rpx 20rpx rgba(0,0,0,0.6), inset 0 0 10rpx rgba(212, 175, 55, 0.3);
  backdrop-filter: blur(4px); /* 毛玻璃特效 */
}
.plaque-inner {
  padding: 20rpx; display: flex; flex-direction: column; align-items: center; justify-content: center;
  border: 1px dashed rgba(212, 175, 55, 0.4); margin: 6rpx; border-radius: 8rpx;
}
.frame-title { color: #fce8b2; font-size: 36rpx; font-family: '楷体', 'KaiTi', serif; font-weight: bold; letter-spacing: 6rpx; text-shadow: 2rpx 2rpx 4rpx rgba(0,0,0,0.8); }
.frame-line { width: 80%; height: 2rpx; background: linear-gradient(to right, transparent, #d4af37, transparent); margin: 12rpx 0; }
.frame-subtitle { color: #d4af37; font-size: 20rpx; letter-spacing: 6rpx; opacity: 0.8; }

/* 4. 底部信息卡片 (悬浮样式，不顶边) */
.info-card {
  position: absolute; bottom: 180rpx; left: 40rpx; right: 40rpx;
  background: linear-gradient(135deg, rgba(44, 29, 17, 0.95), rgba(26, 15, 8, 0.98));
  border: 2rpx solid rgba(212, 175, 55, 0.5); border-radius: 20rpx;
  box-shadow: 0 10rpx 40rpx rgba(0,0,0,0.8);
  transform: translateY(150%); opacity: 0; transition: all 0.4s cubic-bezier(0.34, 1.56, 0.64, 1);
  z-index: 20; padding: 24rpx; backdrop-filter: blur(10px);
}
.card-show { transform: translateY(0); opacity: 1; }
.card-inner { display: flex; align-items: stretch; }
.img-wrapper { position: relative; padding: 4rpx; background: #d4af37; border-radius: 12rpx; margin-right: 24rpx; flex-shrink: 0; height: 160rpx; }
.beast-img { width: 160rpx; height: 160rpx; border-radius: 8rpx; }
.beast-info { flex: 1; display: flex; flex-direction: column; justify-content: space-between; }
.info-header { display: flex; justify-content: space-between; align-items: flex-start; }
.beast-name { color: #fce8b2; font-size: 34rpx; font-weight: bold; font-family: '楷体', 'KaiTi', serif; letter-spacing: 4rpx; }
.status-tag { padding: 4rpx 14rpx; border-radius: 8rpx; background: rgba(255,255,255,0.05); border: 1px solid #666; }
.status-tag text { color: #aaa; font-size: 20rpx; }
.tag-done { background: rgba(212, 175, 55, 0.15); border-color: #d4af37; }
.tag-done text { color: #dce8b2; font-weight: bold; }
.beast-location { color: rgba(255,255,255,0.8); font-size: 22rpx; margin-top: 10rpx; }
.beast-desc { color: rgba(255,255,255,0.5); font-size: 20rpx; margin-top: 6rpx; display: -webkit-box; -webkit-box-orient: vertical; -webkit-line-clamp: 2; overflow: hidden; line-height: 1.4; }
.btn-row { display: flex; justify-content: flex-end; margin-top: 10rpx; }
.checkin-btn { background: linear-gradient(to bottom, #9b2226, #521818); padding: 10rpx 36rpx; border-radius: 30rpx; border: 1px solid #fce8b2; box-shadow: 0 4rpx 10rpx rgba(0,0,0,0.5); }
.checkin-btn text { color: #fce8b2; font-size: 24rpx; font-weight: bold; }
.btn-disabled { background: #2c1d11; border-color: #555; box-shadow: none; }
.btn-disabled text { color: #888; }

/* 5. 底部导航栏 */
.custom-tabbar { position: absolute; bottom: 0; left: 0; width: 100%; height: 140rpx; background: #1a0f08; border-top: 1px solid rgba(212, 175, 55, 0.3); display: flex; justify-content: space-around; align-items: center; padding-bottom: constant(safe-area-inset-bottom); padding-bottom: env(safe-area-inset-bottom); z-index: 100; }
.bar-item { display: flex; flex-direction: column; align-items: center; }
.icon { font-size: 44rpx; margin-bottom: 6rpx; filter: grayscale(100%); opacity: 0.5; }
.text { color: #888; font-size: 22rpx; }
.active-bar .icon { filter: none; opacity: 1; }
.active-bar .text { color: #d4af37; font-weight: bold; }
</style>