<template>
  <view class="container">
<view class="floating-camera" @click="goToSquare">
  <text>👥</text>
</view>
    ...
    <swiper class="beast-swiper" :vertical="false" :indicator-dots="false" @change="onSwiperChange">
      <swiper-item v-for="(beast, index) in beastList" :key="beast.id">
        <view class="swiper-item">
          <image class="bg-image" :src="beast.imageUrl" mode="aspectFill"></image>
          <view class="gradient-overlay"></view>
          
          <view class="info-card">
            <view class="card-header">
              <text class="beast-name">{{ beast.name }}</text>
              <text class="beast-dynasty">{{ beast.dynasty }}</text>
            </view>
            <view class="tags-wrapper">
              <text class="tag">📍 {{ beast.position }}</text>
              <text class="tag">💎 {{ beast.material }}</text>
            </view>
            <text class="beast-desc">{{ beast.stories }}</text>
            
            <view class="action-area">
              <view class="btn-primary" @click="exploreDetail(beast.id)"><text>🌌 图谱</text></view>
              <view class="btn-primary" @click="goToMap"><text>🗺️ 地图</text></view>
              <view class="btn-ai" @click="openAIAgent(beast)"><text>✨ AI</text></view>
              <view class="btn-primary" style="background: rgba(255,255,255,0.1); border: 1px solid rgba(212,175,55,0.5); color: #d4af37;" @click="goToWiki">
                <text>📚 图鉴</text>
              </view>
            </view>
          </view>
        </view>
      </swiper-item>
    </swiper>
  </view>
</template>

<script setup>
import { ref } from 'vue'
import { onLoad } from '@dcloudio/uni-app'

const beastList = ref([])
const currentIndex = ref(0)

const fetchBeastsData = () => {
  uni.request({
    url: 'http://localhost:8080/api/beasts/all',
    method: 'GET',
    success: (res) => {
      if (res.data && res.data.code === 200) {
        beastList.value = res.data.data
      }
    },
    fail: (err) => {
      console.error("数据加载失败", err)
      uni.showToast({ title: '网络请求失败', icon: 'error' })
    }
  })
}

const onSwiperChange = (e) => {
  currentIndex.value = e.detail.current
}

// 跳转到地图页
const goToMap = () => {
  uni.navigateTo({
    url: '/pages/map/map'
  })
}

const openAIAgent = (beast) => {
  uni.navigateTo({
    url: `/pages/ai/ai?name=${beast.name}`
  })
}

const exploreDetail = (id) => {
  uni.navigateTo({
    url: '/pages/graph/graph'
  })
}

onLoad(() => {
  fetchBeastsData()
})

const goToSquare = () => {
  uni.navigateTo({
    url: '/pages/square/square'
  })
}

const goToWiki = () => {
  uni.navigateTo({
    url: '/pages/wiki/wiki'
  })
}
</script>

<style>
/* 沉浸式全屏布局 */
page {
  height: 100%;
  background-color: #1a1a1a;
}
.container {
  height: 100vh;
  width: 100vw;
}
.beast-swiper {
  height: 100%;
  width: 100%;
}
.swiper-item {
  height: 100%;
  position: relative;
}
.bg-image {
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
}
.gradient-overlay {
  position: absolute;
  bottom: 0;
  width: 100%;
  height: 65%;
  background: linear-gradient(to top, rgba(0,0,0,0.9) 0%, rgba(0,0,0,0) 100%);
}

/* 毛玻璃质感信息卡片 */
.info-card {
  position: absolute;
  bottom: 60rpx;
  left: 30rpx;
  right: 30rpx;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border-radius: 30rpx;
  padding: 40rpx;
  border: 1px solid rgba(255, 255, 255, 0.15);
  color: #fff;
  box-shadow: 0 10rpx 30rpx rgba(0,0,0,0.3);
}
.card-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  margin-bottom: 20rpx;
}
.beast-name {
  font-size: 60rpx;
  font-weight: bold;
  letter-spacing: 4rpx;
  text-shadow: 0 4rpx 10rpx rgba(0,0,0,0.6);
}
.beast-dynasty {
  font-size: 26rpx;
  background: linear-gradient(135deg, #d4af37, #aa8529);
  padding: 6rpx 20rpx;
  border-radius: 20rpx;
  font-weight: bold;
}
.tags-wrapper {
  display: flex;
  margin-bottom: 24rpx;
}
.tag {
  font-size: 24rpx;
  background: rgba(0, 0, 0, 0.5);
  padding: 8rpx 20rpx;
  border-radius: 30rpx;
  margin-right: 20rpx;
  border: 1px solid rgba(255,255,255,0.1);
}
.beast-desc {
  font-size: 28rpx;
  line-height: 1.6;
  opacity: 0.85;
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 3;
  overflow: hidden;
  margin-bottom: 40rpx;
}
.action-area {
  display: flex;
  justify-content: space-between;
  gap: 16rpx; /* 控制三个按钮的间距 */
}
.btn-primary {
  background: linear-gradient(135deg, #d4af37, #aa8529);
  padding: 22rpx 0;
  border-radius: 40rpx;
  font-weight: bold;
  font-size: 26rpx;
  flex: 1;
  text-align: center;
  box-shadow: 0 8rpx 20rpx rgba(212, 175, 55, 0.3);
}
.btn-ai {
  background: rgba(255, 255, 255, 0.15);
  padding: 22rpx 0;
  border-radius: 40rpx;
  font-weight: bold;
  font-size: 26rpx;
  flex: 1;
  border: 1px solid rgba(255,255,255,0.3);
  text-align: center;
}
.floating-camera {
  position: absolute;
  top: 100rpx; /* 避开状态栏 */
  right: 40rpx;
  width: 80rpx;
  height: 80rpx;
  background: rgba(0, 0, 0, 0.5);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(212, 175, 55, 0.5);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 40rpx;
  z-index: 100; /* 确保悬浮在最顶层 */
}
</style>