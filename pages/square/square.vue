<template>
  <view class="square-container">
    <view class="nav-bar">
      <view class="back-btn" @click="goBack"><text class="back-icon">‹</text></view>
      <text class="nav-title">寻踪社区</text>
    </view>

    <scroll-view class="post-list" scroll-y>
      <view class="empty-tip" v-if="postList.length === 0">
        <text>还没有人发布寻踪，快来抢首发吧！</text>
      </view>

      <view class="post-card" v-for="post in postList" :key="post.id">
        <image class="post-image" :src="'http://localhost:8080' + post.imageUrl" mode="aspectFill"></image>
        
        <view class="post-info">
          <view class="post-header">
            <text class="beast-tag">{{ post.beastName }}</text>
            <text class="location">📍 {{ post.location }}</text>
          </view>
          <text class="post-content">{{ post.content }}</text>
          
          <view class="interaction-area">
            <view class="action-btn" @click="likePost(post)">
              <text>❤️ {{ post.likes || 0 }}</text>
            </view>
            <view class="action-btn delete-btn" @click="deletePost(post.id)">
              <text>🗑️ 删除</text>
            </view>
          </view>
        </view> </view> <view style="height: 120rpx;"></view>
    </scroll-view>

    <view class="fab-btn" @click="goToPublish">
      <text class="fab-icon">📷 去发布</text>
    </view>
  </view>
</template>

<script setup>
import { ref } from 'vue'
import { onShow } from '@dcloudio/uni-app'

const postList = ref([])

const goBack = () => { uni.navigateBack() }

// 去发布页面
const goToPublish = () => { 
  uni.navigateTo({ url: '/pages/publish/publish' }) 
}

// 获取帖子列表
const fetchPosts = () => {
  uni.request({
    url: 'http://localhost:8080/api/posts/list',
    method: 'GET',
    success: (res) => {
      if (res.data && res.data.code === 200) {
        postList.value = res.data.data
      }
    }
  })
}

// 关键：使用 onShow 而不是 onLoad，这样每次从发布页回来都能刷新出最新帖子
onShow(() => {
  fetchPosts()
})

// 点赞
const likePost = (post) => {
  uni.request({
    url: `http://localhost:8080/api/posts/${post.id}/like`,
    method: 'POST',
    success: (res) => {
      if(res.data.code === 200) { post.likes = (post.likes || 0) + 1 }
    }
  })
}

// 删除
const deletePost = (id) => {
  uni.showModal({
    title: '确认删除',
    content: '删除了就找不回了哦？',
    success: function (res) {
      if (res.confirm) {
        uni.request({
          url: `http://localhost:8080/api/posts/${id}`,
          method: 'DELETE',
          success: (res) => {
            if(res.data.code === 200) { fetchPosts() } // 重新拉取列表
          }
        })
      }
    }
  })
}
</script>

<style>
page {
  background-color: #0f121a;
  height: 100%;
}
.square-container {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
}

/* 导航栏 */
.nav-bar {
  height: 180rpx;
  padding-top: 80rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  background: rgba(15, 18, 26, 0.95);
  z-index: 10;
}
.back-btn {
  position: absolute;
  left: 30rpx;
  width: 80rpx;
  height: 80rpx;
  display: flex;
  align-items: center;
}
.back-icon { color: #fff; font-size: 60rpx; font-weight: 300; }
.nav-title { color: #d4af37; font-size: 34rpx; font-weight: bold; }

/* 列表区 */
.post-list {
  flex: 1;
  padding: 30rpx;
  box-sizing: border-box;
}
.empty-tip {
  text-align: center;
  color: #666;
  margin-top: 100rpx;
  font-size: 28rpx;
}

/* 帖子卡片 */
.post-card {
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 30rpx;
  margin-bottom: 40rpx;
  overflow: hidden;
  box-shadow: 0 10rpx 30rpx rgba(0,0,0,0.5);
}
.post-image {
  width: 100%;
  height: 400rpx;
  background-color: #1a1a1a;
}
.post-info {
  padding: 30rpx;
}
.post-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20rpx;
}
.beast-tag {
  background: linear-gradient(135deg, #d4af37, #aa8529);
  color: #fff;
  padding: 6rpx 20rpx;
  border-radius: 20rpx;
  font-size: 24rpx;
  font-weight: bold;
}
.location {
  color: rgba(255, 255, 255, 0.6);
  font-size: 24rpx;
}
.post-content {
  color: #ccc;
  font-size: 28rpx;
  line-height: 1.6;
}

/* 悬浮发布按钮 */
.fab-btn {
  position: absolute;
  bottom: 80rpx;
  right: 40rpx;
  background: linear-gradient(135deg, #d4af37, #aa8529);
  padding: 24rpx 40rpx;
  border-radius: 50rpx;
  box-shadow: 0 10rpx 30rpx rgba(212, 175, 55, 0.4);
  z-index: 100;
}
.fab-icon {
  color: #000;
  font-weight: bold;
  font-size: 30rpx;
}

.interaction-area {
  display: flex;
  justify-content: flex-end;
  margin-top: 20rpx;
  border-top: 1px dashed rgba(255,255,255,0.1);
  padding-top: 20rpx;
}
.action-btn {
  margin-left: 30rpx;
  color: #999;
  font-size: 26rpx;
  display: flex;
  align-items: center;
}
.delete-btn { color: #ff6b6b; }
</style>