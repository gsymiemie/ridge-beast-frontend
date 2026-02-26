<template>
  <view class="wiki-container">
    <view class="nav-bar">
      <view class="back-btn" @click="goBack"><text class="back-icon">‹</text></view>
      <text class="nav-title">脊兽结构化知识库</text>
    </view>

    <view class="search-area">
      <view class="search-box">
        <text class="search-icon">🔍</text>
        <input class="search-input" placeholder="搜索脊兽、朝代或建筑..." placeholder-style="color: rgba(255,255,255,0.3);" />
      </view>
    </view>

    <view class="wiki-body">
      <scroll-view class="sidebar" scroll-y>
        <view 
          class="menu-item" 
          v-for="(item, index) in categoryList" 
          :key="index"
          :class="{ 'active-menu': currentCategory === index }"
          @click="switchCategory(index)"
        >
          <text>{{ item.name }}</text>
          <view v-if="currentCategory === index" class="active-indicator"></view>
        </view>
      </scroll-view>

      <scroll-view class="content-area" scroll-y>
        <view v-if="currentBeast" class="detail-card">
          <image class="banner-img" :src="currentBeast.image" mode="aspectFill"></image>
          
          <view class="header-info">
            <text class="beast-name">{{ currentBeast.name }}</text>
            <view class="tags-row">
              <text class="tag gold-tag">{{ currentBeast.dynasty }}</text>
              <text class="tag blue-tag">{{ currentBeast.type }}</text>
            </view>
          </view>

          <view class="structured-grid">
            <view class="grid-item">
              <text class="grid-label">📍 放置位置</text>
              <text class="grid-value">{{ currentBeast.position }}</text>
            </view>
            <view class="grid-item">
              <text class="grid-label">✨ 核心寓意</text>
              <text class="grid-value">{{ currentBeast.meaning }}</text>
            </view>
          </view>

          <view class="section">
            <view class="section-title"><text class="dot"></text>历史典故与文化</view>
            <text class="section-text">{{ currentBeast.story }}</text>
          </view>

          <view class="section">
            <view class="section-title"><text class="dot"></text>建筑学功能</view>
            <text class="section-text">{{ currentBeast.function }}</text>
          </view>
        </view>
        
        <view style="height: 100rpx;"></view>
      </scroll-view>
    </view>
  </view>
</template>

<script setup>
import { ref, computed } from 'vue'

// 模拟的结构化数据库 (后续可以轻松替换为 Spring Boot 接口获取)
const database = [
  {
    categoryName: '太和殿十兽',
    beasts: [
      {
        name: '嘲风',
        image:'/static/images/chaofeng.jpg', // 占位图
        dynasty: '明清',
        type: '走兽',
        position: '殿角、戗脊',
        meaning: '威严、震慑邪祟',
        story: '龙生九子之第三子，平生好险又好望，常饰于殿角。它不仅象征着皇权的威严，还有着清除灾祸、辟邪安宅的寓意。',
        function: '实际上起到了固定屋脊瓦片、防止雨水渗漏的实用建筑学作用。'
      },
      {
        name: '螭吻',
        image: 'https://images.unsplash.com/photo-1584347209536-24861b589de6?auto=format&fit=crop&q=80&w=800',
        dynasty: '唐至清',
        type: '正吻',
        position: '正脊两端',
        meaning: '避火防灾',
        story: '龙生九子之一，口润嗓粗而好吞，遂成殿脊两端的吞脊兽。古人认为它能喷水降雨，因此用来取其灭火消灾之意。',
        function: '卡住正脊两端，防止正脊散架，并在早期兼具避雷针的雏形功能。'
      }
    ]
  },
  {
    categoryName: '其他等级走兽',
    beasts: [
      {
        name: '仙人骑凤',
        image: 'https://images.unsplash.com/photo-1528360354687-83a388b1fcb4?auto=format&fit=crop&q=80&w=800',
        dynasty: '清代定制',
        type: '领头兽',
        position: '檐角最前端',
        meaning: '绝处逢生、逢凶化吉',
        story: '相传战国时期齐湣王被燕将乐毅击败，走投无路时有凤凰飞来助其渡过大河。作为脊兽的排头兵，寓意着祈求吉祥。',
        function: '固定最前端的一块瓦片（滴水），防止瓦片滑落。'
      }
    ]
  }
]

// 状态控制
const currentCategory = ref(0)
const categoryList = ref(database.map(d => ({ name: d.categoryName })))

// 计算当前选中的脊兽（默认取该分类下的第一个）
const currentBeast = computed(() => {
  return database[currentCategory.value]?.beasts[0] || null
})

// 切换左侧目录
const switchCategory = (index) => {
  currentCategory.value = index
}

const goBack = () => { uni.navigateBack() }
</script>

<style>
page {
  background-color: #0f121a;
  height: 100%;
}
.wiki-container {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
}

/* 导航栏与搜索区 */
.nav-bar {
  height: 180rpx;
  padding-top: 80rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  background: #0f121a;
  z-index: 10;
}
.back-btn { position: absolute; left: 30rpx; width: 80rpx; height: 80rpx; display: flex; align-items: center; }
.back-icon { color: #fff; font-size: 60rpx; font-weight: 300; }
.nav-title { color: #d4af37; font-size: 34rpx; font-weight: bold; }

.search-area { padding: 10rpx 30rpx 30rpx; background: #0f121a; border-bottom: 1px solid rgba(255,255,255,0.05); }
.search-box { background: rgba(255, 255, 255, 0.08); height: 72rpx; border-radius: 36rpx; display: flex; align-items: center; padding: 0 30rpx; }
.search-icon { font-size: 28rpx; margin-right: 16rpx; opacity: 0.6; }
.search-input { flex: 1; color: #fff; font-size: 26rpx; }

/* 主体布局：左 25%，右 75% */
.wiki-body { flex: 1; display: flex; overflow: hidden; }

/* 左侧目录 */
.sidebar { width: 220rpx; background: rgba(255, 255, 255, 0.02); height: 100%; }
.menu-item { height: 100rpx; display: flex; align-items: center; justify-content: center; color: rgba(255,255,255,0.5); font-size: 28rpx; position: relative; transition: all 0.3s; }
.active-menu { color: #d4af37; font-weight: bold; background: rgba(212, 175, 55, 0.05); }
.active-indicator { position: absolute; left: 0; top: 25rpx; width: 6rpx; height: 50rpx; background: #d4af37; border-radius: 0 6rpx 6rpx 0; }

/* 右侧内容 */
.content-area { flex: 1; padding: 30rpx; box-sizing: border-box; height: 100%; }
.detail-card { background: rgba(255,255,255,0.03); border-radius: 24rpx; overflow: hidden; border: 1px solid rgba(255,255,255,0.05); }
.banner-img { width: 100%; height: 320rpx; background: #1a1a1a; }

.header-info { padding: 30rpx; border-bottom: 1px dashed rgba(255,255,255,0.1); }
.beast-name { color: #fff; font-size: 40rpx; font-weight: bold; display: block; margin-bottom: 16rpx; letter-spacing: 2rpx; }
.tags-row { display: flex; gap: 16rpx; }
.tag { font-size: 22rpx; padding: 6rpx 20rpx; border-radius: 8rpx; font-weight: bold; }
.gold-tag { background: rgba(212, 175, 55, 0.15); color: #d4af37; border: 1px solid rgba(212, 175, 55, 0.3); }
.blue-tag { background: rgba(79, 172, 254, 0.15); color: #4facfe; border: 1px solid rgba(79, 172, 254, 0.3); }

.structured-grid { display: flex; padding: 30rpx; gap: 20rpx; border-bottom: 1px dashed rgba(255,255,255,0.1); }
.grid-item { flex: 1; background: rgba(255,255,255,0.05); padding: 20rpx; border-radius: 16rpx; display: flex; flex-direction: column; }
.grid-label { font-size: 24rpx; color: rgba(255,255,255,0.5); margin-bottom: 10rpx; }
.grid-value { font-size: 26rpx; color: #fff; font-weight: 500; }

.section { padding: 30rpx; }
.section-title { color: #d4af37; font-size: 28rpx; font-weight: bold; margin-bottom: 20rpx; display: flex; align-items: center; }
.dot { width: 10rpx; height: 10rpx; background: #d4af37; border-radius: 50%; margin-right: 12rpx; }
.section-text { color: #ccc; font-size: 26rpx; line-height: 1.8; text-align: justify; }
</style>