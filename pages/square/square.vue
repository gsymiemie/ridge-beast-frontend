<template>
  <view class="square-container">
    
    <view class="header-banner">
      <text class="banner-title">雅集 · 寻踪掠影</text>
      <text class="banner-sub">汇聚天下行者，共赏瑞兽之姿</text>
      <view class="banner-line"></view>
    </view>

    <scroll-view scroll-y class="post-list">
      <view class="post-card" v-for="post in postList" :key="post.id">
        
        <view class="post-header">
          <view class="user-info">
            <image :src="post.avatar" class="user-avatar" mode="aspectFill"></image>
            <view class="name-box">
              <text class="user-name">{{ post.userName }}</text>
              <text class="post-time">{{ post.time }}</text>
            </view>
          </view>
          <view class="tag-box"><text>{{ post.tag }}</text></view>
        </view>

        <text class="post-content">{{ post.content }}</text>

        <view class="post-image-wrapper" @click="previewImage(post.img)">
          <image :src="post.img" class="post-image" mode="aspectFill"></image>
          <view class="image-inner-frame"></view> 
        </view>

        <view class="post-actions">
          <view class="action-btn" @click="toggleLike(post)">
            <text class="action-icon" :class="{'liked': post.isLiked}">
              {{ post.isLiked ? '❤️' : '🤍' }}
            </text>
            <text class="action-num" :class="{'liked-text': post.isLiked}">{{ post.likes }}</text>
          </view>
          <view class="action-btn">
            <text class="action-icon">💬</text>
            <text class="action-num">{{ post.comments }}</text>
          </view>
        </view>
        
      </view>
      
      <view class="loading-more">
        <text>— 卷轴已至尽头 —</text>
      </view>
    </scroll-view>

    <view class="publish-btn" @click="navToPublish">
      <text class="publish-icon">✍️</text>
      <text class="publish-text">提笔</text>
    </view>

    <view class="custom-tabbar">
      <view class="bar-item" @click="navTo('index')"><text class="icon">🌸</text><text class="text">初遇</text></view>
      <view class="bar-item" @click="navTo('humanities')"><text class="icon">📜</text><text class="text">人文</text></view>
      <view class="bar-item" @click="navTo('map')"><text class="icon">🗺️</text><text class="text">畅游</text></view>
      <view class="bar-item active-bar"><text class="icon">🌊</text><text class="text">潮玩</text></view>
      <view class="bar-item" @click="navTo('mine')"><text class="icon">👤</text><text class="text">我的</text></view>
    </view>
    
  </view>
</template>

<script setup>
import { ref } from 'vue'

// 模拟社区动态数据
const postList = ref([
  {
    id: 1,
    userName: '江南第一深情',
    avatar: '/static/images/ailogo.png',
    time: '甲辰年 春分',
    tag: '古建迷',
    content: '今日登顶太和殿，阳光正好，拍到了这尊绝美的嘲风雕像。不愧是“威慑邪祟”的神兽，太有气势了！',
    img: '/static/images/chaofeng.jpg',
    likes: 128,
    comments: 32,
    isLiked: false
  },
  {
    id: 2,
    userName: '寻踪行者_007',
    avatar: '/static/images/ailogo.png',
    time: '昨日 戌时',
    tag: '摄影控',
    content: '夜游古镇，发现了屋脊上的螭吻。据说它可以避火防灾，古人的智慧与审美真的令人叹服。',
    img: '/static/images/chiwen.jpg',
    likes: 85,
    comments: 16,
    isLiked: true
  }
])

// 点赞交互
const toggleLike = (post) => {
  if (post.isLiked) {
    post.likes--
    post.isLiked = false
  } else {
    post.likes++
    post.isLiked = true
    // 震动反馈增加手感
    uni.vibrateShort() 
  }
}

// 预览大图
const previewImage = (url) => {
  uni.previewImage({
    urls: [url],
    current: url
  })
}

// 跳转到发布页面 (如果之前写了上传页，这里可以填入对应路径)
const navToPublish = () => {
  uni.showToast({ title: '即将展开空白卷轴...', icon: 'none' })
  // uni.navigateTo({ url: '/pages/publish/publish' })
}

// 统一的页面路由跳转
const navTo = (page) => {
  if (page === 'index') uni.redirectTo({ url: '/pages/index/index' })
  if (page === 'humanities') uni.redirectTo({ url: '/pages/humanities/humanities' })
  if (page === 'map') uni.redirectTo({ url: '/pages/map/map' })
  if (page === 'mine') uni.redirectTo({ url: '/pages/mine/mine' })
}
</script>

<style>
page { margin: 0; padding: 0; background-color: #2b1d15; height: 100%; color: #e6d2b5; }
.square-container { width: 100vw; height: 100vh; display: flex; flex-direction: column; position: relative; overflow: hidden; background-color: #2b1d15; }
/* 1. 顶部标题栏 */
.header-banner {
  padding: 100rpx 40rpx 30rpx 40rpx;
  /* 顶部变得更亮更暖，平滑过渡到底部的沉香色 */
  background: linear-gradient(to bottom, rgba(74, 51, 32, 0.95), #2b1d15);
  display: flex; flex-direction: column; align-items: center; z-index: 10;
}
.banner-title { color: #fce8b2; font-size: 42rpx; font-weight: bold; font-family: '楷体', 'KaiTi', serif; letter-spacing: 8rpx; text-shadow: 0 4rpx 10rpx rgba(0,0,0,0.8); }
.banner-sub { color: #888; font-size: 24rpx; margin-top: 10rpx; letter-spacing: 4rpx; }
.banner-line { width: 40%; height: 2rpx; background: linear-gradient(to right, transparent, rgba(212, 175, 55, 0.5), transparent); margin-top: 20rpx; }

/* 2. 动态列表区 */
.post-list { 
  flex: 1; 
  /* 👇 【核心魔法】：强行约束高度，激活内部滚动！ */
  height: 0; 
  padding: 0 30rpx; 
  box-sizing: border-box; 
}
/* 单个动态卡片 */
.post-card {
  /* 卡片底色提亮，增加木质暖调 */
  background: linear-gradient(135deg, rgba(82, 54, 34, 0.8), rgba(44, 29, 17, 0.9));
  border: 1px solid rgba(212, 175, 55, 0.3); border-radius: 16rpx;
  padding: 30rpx; margin-bottom: 40rpx;
  /* 阴影透明度从 0.5 降到 0.3，让卡片显得更轻盈 */
  box-shadow: 0 8rpx 20rpx rgba(0,0,0,0.3);
}

/* 用户信息 */
.post-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20rpx; }
.user-info { display: flex; align-items: center; }
.user-avatar { width: 80rpx; height: 80rpx; border-radius: 50%; border: 2rpx solid #d4af37; margin-right: 20rpx; }
.name-box { display: flex; flex-direction: column; }
.user-name { color: #fce8b2; font-size: 30rpx; font-weight: bold; }
.post-time { color: #666; font-size: 22rpx; margin-top: 6rpx; }
.tag-box { border: 1px solid rgba(212, 175, 55, 0.4); padding: 4rpx 16rpx; border-radius: 8rpx; background: rgba(0,0,0,0.2); }
.tag-box text { color: #d4af37; font-size: 22rpx; }

/* 文案与图片 */
.post-content { 
  /* 从原本刺眼的亮白/冷灰，改为极其护眼的“宣纸黄” */
  color: #f2e4d0; 
  font-size: 28rpx; line-height: 1.6; display: block; margin-bottom: 20rpx; text-align: justify; 
}
.post-image-wrapper { position: relative; width: 100%; height: 360rpx; border-radius: 12rpx; overflow: hidden; padding: 6rpx; background: #2c1d11; box-sizing: border-box; }
.post-image { width: 100%; height: 100%; border-radius: 8rpx; }
.image-inner-frame { position: absolute; top: 12rpx; left: 12rpx; right: 12rpx; bottom: 12rpx; border: 1px dashed rgba(212, 175, 55, 0.6); pointer-events: none; z-index: 2; }

/* 互动区 */
.post-actions { display: flex; justify-content: flex-end; margin-top: 30rpx; border-top: 1px dashed rgba(255,255,255,0.1); padding-top: 20rpx; }
.action-btn { display: flex; align-items: center; margin-left: 40rpx; transition: all 0.2s; }
.action-btn:active { transform: scale(0.9); }
.action-icon { font-size: 36rpx; margin-right: 10rpx; filter: grayscale(100%) brightness(1.5); transition: all 0.3s; }
.action-icon.liked { filter: none; animation: pop 0.3s ease; }
@keyframes pop { 0% { transform: scale(1); } 50% { transform: scale(1.3); } 100% { transform: scale(1); } }
.action-num { color: #888; font-size: 26rpx; }
.liked-text { color: #ff4d4f; font-weight: bold; }

.loading-more { text-align: center; padding: 200rpx 0; color: #555; font-size: 24rpx; font-family: '楷体', 'KaiTi', serif; letter-spacing: 4rpx; }

/* 3. 悬浮提笔按钮 */
.publish-btn {
  position: absolute; bottom: 260rpx; right: 40rpx;
  width: 110rpx; height: 110rpx; border-radius: 50%;
  background: linear-gradient(135deg, #9b2226, #521818);
  border: 4rpx solid #d4af37;
  display: flex; flex-direction: column; align-items: center; justify-content: center;
  box-shadow: 0 8rpx 20rpx rgba(0,0,0,0.8), inset 0 0 10rpx rgba(255,255,255,0.2);
  z-index: 50; transition: all 0.2s;
}
.publish-btn:active { transform: scale(0.95); }
.publish-icon { font-size: 40rpx; margin-bottom: -4rpx; }
.publish-text { color: #fce8b2; font-size: 22rpx; font-weight: bold; font-family: '楷体', 'KaiTi', serif; }

/* 4. 底部导航栏 */
.custom-tabbar { position: absolute; bottom: 0; left: 0; width: 100%; height: 140rpx; background: #1a0f08; border-top: 1px solid rgba(212, 175, 55, 0.3); display: flex; justify-content: space-around; align-items: center; padding-bottom: constant(safe-area-inset-bottom); padding-bottom: env(safe-area-inset-bottom); z-index: 100; }
.bar-item { display: flex; flex-direction: column; align-items: center; }
.icon { font-size: 44rpx; margin-bottom: 6rpx; filter: grayscale(100%); opacity: 0.5; }
.text { color: #888; font-size: 22rpx; }
.active-bar .icon { filter: none; opacity: 1; }
.active-bar .text { color: #d4af37; font-weight: bold; }
</style>