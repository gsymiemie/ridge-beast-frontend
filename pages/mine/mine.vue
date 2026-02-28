<template>
  <view class="mine-container">
    
    <view class="header-section">
      <view class="user-info">
        <view class="avatar-box">
          <image src="/static/images/ailogo.png" mode="aspectFill" class="avatar"></image>
          <view class="avatar-ring"></view>
        </view>
        <view class="user-text">
          <text class="user-name">寻踪行者_9527</text>
          <view class="user-tag"><text>Lv.3 御前行走</text></view>
        </view>
      </view>
      <view class="setting-icon">⚙️</view>
    </view>

    <view class="vip-card">
      <view class="card-bg-shine"></view> <view class="card-top">
        <text class="card-title">太和专属·通关文牒</text>
        <text class="card-no">No. 8886 6668</text>
      </view>
      <view class="card-bottom">
        <view class="card-item">
          <text class="item-num">1,280</text>
          <text class="item-label">瑞气 (积分)</text>
        </view>
        <view class="card-item">
          <text class="item-num">12</text>
          <text class="item-label">点亮脊兽</text>
        </view>
        <view class="qr-btn" @click="showQR">
          <text>出示文牒</text>
        </view>
      </view>
    </view>

    <view class="check-in-panel">
      <view class="panel-header">
        <text class="panel-title">七日纳福</text>
        <text class="panel-sub">已连续签到 3 天</text>
      </view>
      <view class="step-axis">
        <view class="step-line"></view>
        <view class="step-item" v-for="(day, index) in 7" :key="index">
          <view class="step-dot" :class="{'dot-active': index < 3, 'dot-today': index === 3}"></view>
          <text class="step-text">{{ index === 3 ? '今日' : index + 1 + '日' }}</text>
        </view>
      </view>
      <view class="btn-checkin" @click="doCheckIn">
        <text>今日祈福</text>
      </view>
    </view>

    <view class="service-grid">
      <view class="grid-item">
        <text class="grid-icon">⭐</text><text class="grid-name">我的收藏</text>
      </view>
      <view class="grid-item">
        <text class="grid-icon">👣</text><text class="grid-name">寻踪足迹</text>
      </view>
      <view class="grid-item">
        <text class="grid-icon">📜</text><text class="grid-name">求签记录</text>
      </view>
      <view class="grid-item">
        <text class="grid-icon">🤖</text><text class="grid-name">AI 对话</text>
      </view>
    </view>

    <view class="custom-tabbar">
      <view class="bar-item" @click="navTo('index')">
        <text class="icon">🌸</text><text class="text">初遇</text>
      </view>
      <view class="bar-item" @click="navTo('humanities')">
        <text class="icon">📜</text><text class="text">人文</text>
      </view>
      <view class="bar-item" @click="navTo('map')">
        <text class="icon">🗺️</text><text class="text">畅游</text>
      </view>
      <view class="bar-item" @click="navTo('square')">
        <text class="icon">🌊</text><text class="text">潮玩</text>
      </view>
      <view class="bar-item active-bar">
        <text class="icon">👤</text><text class="text">我的</text>
      </view>
    </view>
    
  </view>
</template>

<script setup>
import { ref } from 'vue'

const showQR = () => {
  uni.showToast({ title: '文牒二维码生成中...', icon: 'none' })
}

const doCheckIn = () => {
  uni.showToast({ title: '祈福成功！瑞气 +10', icon: 'success' })
}

// 统一的页面路由跳转 (使用 redirectTo 防止页面栈溢出)
const navTo = (page) => {
  if (page === 'index') uni.redirectTo({ url: '/pages/index/index' })
  if (page === 'humanities') uni.redirectTo({ url: '/pages/humanities/humanities' })
  if (page === 'map') uni.navigateTo({ url: '/pages/map/map' })
  if (page === 'square') uni.navigateTo({ url: '/pages/square/square' })
}
</script>

<style>
page { margin: 0; padding: 0; background-color: #160e08; height: 100%; color: #fce8b2; }
.mine-container { width: 100vw; min-height: 100vh; position: relative; padding-bottom: 260rpx; box-sizing: border-box; }

/* 1. 顶部用户信息 */
.header-section {
  padding: 40rpx 40rpx 40rpx 40rpx;
  background: radial-gradient(circle at top right, rgba(212, 175, 55, 0.2), transparent 60%);
  display: flex; justify-content: space-between; align-items: center;
}
.user-info { display: flex; align-items: center; }
.avatar-box { position: relative; width: 120rpx; height: 120rpx; margin-right: 30rpx; }
.avatar { width: 100%; height: 100%; border-radius: 50%; border: 4rpx solid #d4af37; position: relative; z-index: 2; }
.avatar-ring { position: absolute; top: -10rpx; left: -10rpx; width: 140rpx; height: 140rpx; border: 2rpx dashed rgba(212, 175, 55, 0.5); border-radius: 50%; animation: spin 10s linear infinite; }
@keyframes spin { 100% { transform: rotate(360deg); } }
.user-name { font-size: 40rpx; font-weight: bold; font-family: '楷体', 'KaiTi', serif; letter-spacing: 4rpx; display: block; margin-bottom: 10rpx; text-shadow: 0 2rpx 4rpx rgba(0,0,0,0.8); }
.user-tag { background: linear-gradient(to right, #8c2a2a, #521818); padding: 4rpx 16rpx; border-radius: 20rpx; border: 1px solid rgba(252, 232, 178, 0.3); display: inline-block; }
.user-tag text { font-size: 20rpx; color: #fce8b2; }
.setting-icon { font-size: 44rpx; opacity: 0.8; }

/* 2. VIP 金卡 */
.vip-card {
  margin: 0 30rpx; height: 320rpx; border-radius: 24rpx;
  background: linear-gradient(135deg, #dfb962, #fce8b2, #aa8529, #dfb962);
  background-size: 200% 200%;
  box-shadow: 0 10rpx 30rpx rgba(0,0,0,0.6), inset 0 0 20rpx rgba(255,255,255,0.4);
  padding: 40rpx; box-sizing: border-box; display: flex; flex-direction: column; justify-content: space-between;
  position: relative; overflow: hidden;
}
/* 扫光特效 */
.card-bg-shine { position: absolute; top: 0; left: -100%; width: 50%; height: 100%; background: linear-gradient(to right, rgba(255,255,255,0) 0%, rgba(255,255,255,0.4) 50%, rgba(255,255,255,0) 100%); transform: skewX(-25deg); animation: shine 4s infinite; }
@keyframes shine { 0% { left: -100%; } 20% { left: 200%; } 100% { left: 200%; } }
.card-top { display: flex; justify-content: space-between; align-items: center; }
.card-title { color: #521818; font-size: 34rpx; font-weight: bold; font-family: '楷体', 'KaiTi', serif; }
.card-no { color: #521818; font-size: 24rpx; opacity: 0.8; font-family: monospace; }
.card-bottom { display: flex; justify-content: space-between; align-items: flex-end; }
.card-item { display: flex; flex-direction: column; }
.item-num { color: #1a0f08; font-size: 48rpx; font-weight: bold; margin-bottom: 4rpx; }
.item-label { color: #521818; font-size: 22rpx; opacity: 0.9; }
.qr-btn { background: #1a0f08; padding: 12rpx 24rpx; border-radius: 30rpx; border: 1px solid #d4af37; }
.qr-btn text { color: #fce8b2; font-size: 24rpx; font-weight: bold; }

/* 3. 签到轴 */
.check-in-panel {
  margin: 40rpx 30rpx; background: rgba(44, 29, 17, 0.6); border: 1px solid rgba(212, 175, 55, 0.2);
  border-radius: 20rpx; padding: 30rpx; box-shadow: 0 4rpx 16rpx rgba(0,0,0,0.5);
}
.panel-header { display: flex; justify-content: space-between; align-items: flex-end; margin-bottom: 30rpx; }
.panel-title { font-size: 32rpx; font-weight: bold; font-family: '楷体', 'KaiTi', serif; }
.panel-sub { font-size: 24rpx; color: #aaa; }
.step-axis { position: relative; display: flex; justify-content: space-between; margin-bottom: 40rpx; padding: 0 10rpx; }
.step-line { position: absolute; top: 12rpx; left: 20rpx; right: 20rpx; height: 2rpx; background: rgba(212, 175, 55, 0.3); z-index: 1; }
.step-item { display: flex; flex-direction: column; align-items: center; z-index: 2; }
.step-dot { width: 24rpx; height: 24rpx; border-radius: 50%; background: #1a0f08; border: 2rpx solid #555; margin-bottom: 12rpx; transition: all 0.3s; }
.dot-active { background: #d4af37; border-color: #fce8b2; box-shadow: 0 0 10rpx rgba(212, 175, 55, 0.8); }
.dot-today { background: #8c2a2a; border-color: #fce8b2; transform: scale(1.3); }
.step-text { font-size: 20rpx; color: #888; }
.dot-today + .step-text { color: #d4af37; font-weight: bold; }
.btn-checkin { background: linear-gradient(to right, #8c2a2a, #521818); text-align: center; padding: 20rpx 0; border-radius: 40rpx; border: 1px solid rgba(252, 232, 178, 0.5); }
.btn-checkin text { font-size: 28rpx; font-weight: bold; letter-spacing: 4rpx; }

/* 4. 藏宝阁宫格 */
.service-grid {
  margin: 0 30rpx; display: flex; flex-wrap: wrap; justify-content: space-between;
}
.grid-item {
  width: 48%; background: rgba(44, 29, 17, 0.6); border: 1px solid rgba(212, 175, 55, 0.15);
  border-radius: 16rpx; padding: 40rpx 0; margin-bottom: 24rpx; display: flex; flex-direction: column; align-items: center;
  box-shadow: 0 4rpx 10rpx rgba(0,0,0,0.3); transition: all 0.2s;
}
.grid-item:active { transform: scale(0.95); background: rgba(44, 29, 17, 0.9); }
.grid-icon { font-size: 50rpx; margin-bottom: 16rpx; }
.grid-name { font-size: 26rpx; color: #ccc; }

/* 5. 底部导航栏 (直接复用首页的样式代码) */
.custom-tabbar { position: fixed; bottom: 0; left: 0; width: 100%; height: 140rpx; background: #1a0f08; border-top: 1px solid rgba(212, 175, 55, 0.3); display: flex; justify-content: space-around; align-items: center; padding-bottom: constant(safe-area-inset-bottom); padding-bottom: env(safe-area-inset-bottom); z-index: 100; }
.bar-item { display: flex; flex-direction: column; align-items: center; }
.icon { font-size: 44rpx; margin-bottom: 6rpx; filter: grayscale(100%); opacity: 0.5; }
.text { color: #888; font-size: 22rpx; }
.active-bar .icon { filter: none; opacity: 1; }
.active-bar .text { color: #d4af37; font-weight: bold; }
</style>