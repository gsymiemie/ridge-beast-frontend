<template>
  <view class="map-container">
    <map
      id="beastMap"
      class="beast-map"
      :latitude="centerLat"
      :longitude="centerLng"
      :markers="markers"
      :scale="12"
      @markertap="onMarkerTap"
      @tap="closePanel"
    ></map>

    <view class="bottom-panel" :class="{ 'panel-show': selectedArch }">
      <block v-if="selectedArch">
        <view class="panel-header">
          <view>
            <text class="arch-name">{{ selectedArch.name }}</text>
            <text class="arch-dynasty">{{ selectedArch.dynasty }}</text>
          </view>
          <text class="close-btn" @click="closePanel">×</text>
        </view>
        
        <view class="tags">
          <text class="tag-style">{{ selectedArch.architecturalStyle }}</text>
        </view>
        
        <text class="arch-desc">{{ selectedArch.historicalSignificance }}</text>
        <text class="visit-info">?? {{ selectedArch.visitInfo }}</text>
        <text class="location-info">?? {{ selectedArch.location }}</text>

        <view class="nav-btn" @click="goNavigate">
          <text>??️ 唤起本地地图导航</text>
        </view>
      </block>
    </view>
  </view>
</template>

<script setup>
import { ref } from 'vue'
import { onLoad } from '@dcloudio/uni-app'

// 默认中心点定在故宫附近
const centerLat = ref(39.9181) 
const centerLng = ref(116.3974)

const architectures = ref([])
const markers = ref([])
const selectedArch = ref(null)

// 1. 获取后端建筑坐标数据
const fetchMapData = () => {
  uni.request({
    url: 'http://localhost:8080/api/architectures/all',
    method: 'GET',
    success: (res) => {
      if (res.data && res.data.code === 200) {
        architectures.value = res.data.data
        
        // 2. 将建筑数据转换为小程序地图 marker 格式
        markers.value = architectures.value.map(arch => ({
          id: arch.id,
          latitude: arch.latitude,
          longitude: arch.longitude,
          title: arch.name,
          width: 30,
          height: 30,
          // 气泡提示
          callout: {
            content: arch.name,
            color: '#ffffff',
            fontSize: 14,
            borderRadius: 8,
            bgColor: '#d4af37', // 高级金
            padding: 8,
            display: 'ALWAYS'
          }
        }))
      }
    }
  })
}

// 3. 点击地图上的标记点
const onMarkerTap = (e) => {
  const markerId = e.detail.markerId
  // 匹配并展示底部数据
  selectedArch.value = architectures.value.find(a => a.id === markerId)
}

// 4. 点击空白处关闭面板
const closePanel = () => {
  selectedArch.value = null
}

// 5. 核心：调用微信原生导航 API
const goNavigate = () => {
  if(!selectedArch.value) return;
  uni.openLocation({
    latitude: selectedArch.value.latitude,
    longitude: selectedArch.value.longitude,
    name: selectedArch.value.name,
    address: selectedArch.value.location,
    scale: 15
  })
}

onLoad(() => {
  fetchMapData()
})
</script>

<style>
.map-container {
  width: 100vw;
  height: 100vh;
  position: relative;
  overflow: hidden;
}
.beast-map {
  width: 100%;
  height: 100%;
}

/* 底部弹出面板样式 */
.bottom-panel {
  position: absolute;
  bottom: -600rpx; /* 默认隐藏在屏幕下方 */
  left: 0;
  width: 100%;
  background: #ffffff;
  border-radius: 40rpx 40rpx 0 0;
  padding: 40rpx 40rpx 60rpx 40rpx;
  box-sizing: border-box;
  box-shadow: 0 -10rpx 30rpx rgba(0,0,0,0.1);
  transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1);
  z-index: 999;
}
.panel-show {
  bottom: 0; /* 激活时升起 */
}
.panel-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20rpx;
}
.arch-name {
  font-size: 44rpx;
  font-weight: bold;
  color: #333;
  margin-right: 20rpx;
}
.arch-dynasty {
  font-size: 24rpx;
  background: #f0f0f0;
  color: #666;
  padding: 4rpx 16rpx;
  border-radius: 8rpx;
}
.close-btn {
  font-size: 50rpx;
  color: #999;
  padding: 0 20rpx;
}
.tags {
  margin-bottom: 20rpx;
}
.tag-style {
  font-size: 24rpx;
  color: #d4af37;
  border: 1px solid #d4af37;
  padding: 4rpx 12rpx;
  border-radius: 6rpx;
}
.arch-desc {
  font-size: 28rpx;
  color: #666;
  line-height: 1.6;
  display: block;
  margin-bottom: 20rpx;
}
.visit-info, .location-info {
  font-size: 26rpx;
  color: #888;
  display: block;
  margin-bottom: 10rpx;
}
.nav-btn {
  background: #333;
  color: #d4af37;
  text-align: center;
  padding: 24rpx 0;
  border-radius: 40rpx;
  margin-top: 30rpx;
  font-weight: bold;
  font-size: 30rpx;
  box-shadow: 0 10rpx 20rpx rgba(0,0,0,0.2);
}
</style>