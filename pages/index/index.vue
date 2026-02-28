<template>
  <view class="home-container">
    
    <view class="fade-bg-container">
      <view 
        class="fade-slide" 
        v-for="(beast, index) in beastList" 
        :key="beast.id"
        :class="{ 'active-slide': currentIndex === index }"
      >
        <image :src="beast.img" class="slide-img" mode="aspectFill"></image>
        
        <view class="slide-content-wrapper">
           <view class="text-area-left">
             <text class="beast-title">{{ beast.name }}</text>
             <text class="beast-subtitle">{{ beast.desc }}</text>
           </view>
           
           <view class="model-3d-container">
             <canvas type="webgl" id="webgl" canvas-id="webgl"></canvas>
           </view>
        </view>
      </view>
    </view>

<view class="floating-ai" @click="openAI">
      <view class="ai-halo"></view>
      <image src="/static/images/ailogo.png" class="ai-avatar" mode="aspectFill"></image>
      <view class="ai-badge"><text>AI 寻踪</text></view>
    </view>

    <view class="scroll-area">
      <view class="jade-tags">
        <view class="jade-btn" :class="{'active-jade': scrollTab === 'blessing'}" @click="scrollTab = 'blessing'">
          <text>太</text><text>和</text><text>祈</text><text>福</text>
        </view>
        <view class="jade-btn" :class="{'active-jade': scrollTab === 'graph'}" @click="scrollTab = 'graph'">
          <text>文</text><text>化</text><text>图</text><text>谱</text>
        </view>
      </view>
      
      <view class="main-scroll" @click="handleScrollClick">
        <view class="scroll-axis left-axis"></view>
        <view class="scroll-content" v-if="scrollTab === 'blessing'">
          <text class="scroll-title">冬至太和 福瑞吉祥</text>
          <text class="scroll-sub">✨ 点击抽取今日瑞气 ✨</text>
        </view>
        <view class="scroll-content" v-if="scrollTab === 'graph'">
          <text class="scroll-title">星系纵横 脉络古今</text>
          <text class="scroll-sub">🌌 点击探索文化图谱 🌌</text>
        </view>
        <view class="scroll-axis right-axis"></view>
      </view>
    </view>

    <view class="custom-tabbar">
      <view class="bar-item active-bar"><text class="icon">🌸</text><text class="text">初遇</text></view>
      <view class="bar-item" @click="navTo('humanities')"><text class="icon">📜</text><text class="text">人文</text></view>
      <view class="bar-item" @click="navTo('map')"><text class="icon">🗺️</text><text class="text">畅游</text></view>
      <view class="bar-item" @click="navTo('square')"><text class="icon">🌊</text><text class="text">潮玩</text></view>
      <view class="bar-item" @click="navTo('mine')"><text class="icon">👤</text><text class="text">我的</text></view>
    </view>
    <view class="top-left-ornament">
          <view class="red-stamp"><text>太和</text></view>
          <view class="lunar-date">
            <text class="date-text">甲辰年 · 春分</text>
            <view class="date-line"></view> </view>
        </view>
  </view>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { onShow, onHide } from '@dcloudio/uni-app'

// --- 引入 强大的完全体 three-platformize ---
import * as THREE from 'three-platformize'
import { WechatPlatform } from 'three-platformize/src/WechatPlatform'
import { STLLoader } from 'three-platformize/examples/jsm/loaders/STLLoader'

// 轮播数据
const beastList = ref([
  { id: 1, name: '嘲 风', desc: '平生好险，威慑邪祟', img: '/static/images/chaofeng.jpg' },
  { id: 2, name: '螭 吻', desc: '口润嗓粗，避火防灾', img: '/static/images/chiwen.jpg' },
  { id: 3, name: '仙人骑凤', desc: '排头指路，逢凶化吉', img: '/static/images/immortal.jpg' }
])
const currentIndex = ref(0)
let timer = null
const scrollTab = ref('blessing')

// --- Three.js 核心变量 ---
let camera, scene, renderer, modelMesh
let requestAnimationFrameId = null
let canvasNode = null // 提升画布作用域，防止动画报错

// --- 初始化 3D 场景核心函数 ---
const initThree = () => {
  uni.createSelectorQuery()
    .select('#webgl')
    .node()
    .exec((res) => {
      canvasNode = res[0].node
      
      // 【关键修复】使用 WechatPlatform 完美适配微信底层环境
      const platform = new WechatPlatform(canvasNode)
      THREE.PLATFORM.set(platform)
      
      // 1. 创建场景 (大写的 THREE)
      scene = new THREE.Scene()
      
      // 2. 创建透视相机
      camera = new THREE.PerspectiveCamera(45, canvasNode.width / canvasNode.height, 0.1, 1000)
      camera.position.set(0, 2, 10) 
      camera.lookAt(0, 0, 0)

      // 3. 创建渲染器
      renderer = new THREE.WebGLRenderer({
        canvas: canvasNode,
        antialias: true, // 抗锯齿
        alpha: true      // 透明背景
      })
      renderer.setSize(canvasNode.width, canvasNode.height)
      renderer.setPixelRatio(uni.getSystemInfoSync().pixelRatio)

      // 4. 打光 (无光则黑)
      const ambientLight = new THREE.AmbientLight(0xffffff, 0.6)
      scene.add(ambientLight)
      const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8)
      directionalLight.position.set(5, 10, 7.5)
      scene.add(directionalLight)
      
      // 5. 加载 STL 模型
      loadSTLModel('/static/models/chiwen.stl')

      // 开始渲染循环
      animate()
  })
}

// 加载 STL 专用函数
// --- 专门针对微信小程序的本地模型加载方案 ---
// --- 工业级方案：从后端服务器加载 3D 模型 ---
const loadSTLModel = () => {
  const loader = new STLLoader()
  
  // 指向你本地运行的 Spring Boot 后端地址
  const modelUrl = 'http://127.0.0.1:8080/models/chiwen.stl'

  loader.load(modelUrl, (geometry) => {
    // 创建流金色材质
    const material = new THREE.MeshPhongMaterial({ 
      color: 0xd4af37, 
      specular: 0x111111,
      shininess: 100 
    })
    
	
	
// 👇 ---------- 【新增：在这里给模型“正骨”】 ---------- 👇
      // 常见情况 1：如果是“躺着”的，通常是绕 X 轴旋转 90 度
      geometry.rotateX(-Math.PI / 2) 
      
      // 如果上面那行不行，把它注释掉，试试下面这几种：
      // geometry.rotateX(Math.PI / 2)   // 往反方向躺
      // geometry.rotateX(Math.PI)       // 完全倒立的话转180度
      // geometry.rotateZ(-Math.PI / 2)  // 侧翻的话绕Z轴转
      // 👆 ------------------------------------------------ 👆

      geometry.center() // 自动居中
            modelMesh = new THREE.Mesh(geometry, material)
            
            // 这是你目前调整好的倍率
            modelMesh.scale.set(0.15, 0.15, 0.15) 
            
            // 👇 【新增这一行：精准控制上下位置】 👇
            // Y轴坐标：负数代表往下移动，正数代表往上移动。
            // 请盲测调整这个数字（比如 -1, -1.5, -2, -3 等），直到它跟左边文字完美对齐！
            //modelMesh.position.y = -1 
            
            //-scene.add(modelMesh)
			
      // 如果扶正之后发现模型太靠上或者太靠下，可以微调它的 Y 坐标：
      // modelMesh.position.y = -1 // 负数往下走，正数往上走
      
      scene.add(modelMesh)
      console.log("🎉 3D模型从后端加载成功！")
    
  }, undefined, (error) => {
    console.error('模型加载失败，请检查 Spring Boot 是否启动，或路径是否正确:', error)
  })
}

// 动画渲染循环
const animate = () => {
  if (canvasNode) {
    requestAnimationFrameId = canvasNode.requestAnimationFrame(animate)
  }
  // 让模型绕着 Y 轴自转
  if (modelMesh) {
    modelMesh.rotation.y += 0.01 
  }
  renderer.render(scene, camera)
}

// --- 页面生命周期与交互逻辑 ---
onMounted(() => {
  setTimeout(() => { initThree() }, 500)
  startTimer()
})
onUnmounted(() => {
  stopTimer()
  if (requestAnimationFrameId && canvasNode) {
    canvasNode.cancelAnimationFrame(requestAnimationFrameId)
  }
  if (renderer) renderer.dispose()
})

onShow(() => startTimer())
onHide(() => stopTimer())

const startTimer = () => {
  timer = setInterval(() => {
    currentIndex.value = (currentIndex.value + 1) % beastList.value.length
  }, 4000)
}
const stopTimer = () => { if (timer) clearInterval(timer) }

const handleScrollClick = () => {
  if (scrollTab.value === 'graph') uni.navigateTo({ url: '/pages/graph/graph' })
  else uni.showToast({ title: '🌟 抽取成功：嘲风护佑，今日宜登高！', icon: 'none', duration: 3000 })
}

const openAI = () => { uni.navigateTo({ url: '/pages/ai/ai' }) }

// 统一的页面路由跳转
const navTo = (page) => {
  if (page === 'humanities') uni.redirectTo({ url: '/pages/humanities/humanities' })
  if (page === 'map') uni.navigateTo({ url: '/pages/map/map' })
  if (page === 'square') uni.navigateTo({ url: '/pages/square/square' })
  // 👇 【新增这一行，打通前往个人中心的路线】
  if (page === 'mine') uni.redirectTo({ url: '/pages/mine/mine' }) 
}
</script>

<style>
page { margin: 0; padding: 0; background-color: #160e08; height: 100%; }
.home-container { width: 100vw; height: 100vh; position: relative; overflow: hidden; }

/* 背景渐变轮播 */
.fade-bg-container { width: 100%; height: 100%; position: absolute; top: 0; left: 0; }
.fade-slide { position: absolute; top: 0; left: 0; width: 100%; height: 100%; opacity: 0; visibility: hidden; transition: opacity 1.5s ease-in-out, visibility 1.5s ease-in-out; }
.active-slide { opacity: 1; visibility: visible; z-index: 10; }
.slide-img { width: 100%; height: 100%; }

/* --- 核心布局修改：左右分栏 --- */
/* 将蒙层改为 Flex 布局，支撑左右结构 */
.slide-content-wrapper {
  position: absolute; bottom: 0; left: 0; width: 100%; height: 70%;
  background: linear-gradient(to top, #160e08 20%, rgba(22,14,8,0.6) 50%, transparent);
  display: flex;
  justify-content: space-between; /* 左右撑开 */
  align-items: flex-end; /* 底部对齐 */
  padding: 0 30rpx 460rpx 30rpx; /* 底部留出空间给卷轴 */
  box-sizing: border-box;
}

/* 左侧文字区 (左对齐) */
.text-area-left {
  flex: 1; /* 占据左侧空间 */
  display: flex; flex-direction: column; justify-content: flex-end;
  padding-right: 20rpx;
  z-index: 20; /* 确保文字在模型上层 */
}
.beast-title { color: #fce8b2; font-size: 70rpx; font-weight: bold; letter-spacing: 10rpx; margin-bottom: 16rpx; text-shadow: 0 4rpx 10rpx rgba(0,0,0,0.8); text-align: left; }
.beast-subtitle { color: rgba(255,255,255,0.8); font-size: 30rpx; letter-spacing: 4rpx; text-align: left;}

/* 右侧 3D 模型容器 */
.model-3d-container {
  width: 350rpx; /* 控制模型区域宽度 */
  height: 500rpx; /* 控制模型区域高度 */
  position: relative;
  /* 可以微调位置，比如往右下角一点 */
  margin-bottom: -50rpx; 
  margin-right: -20rpx;
}
/* WebGL 画布填满容器 */
#webgl {
  width: 100%;
  height: 100%;
}

/* --- 其他组件样式保持不变 --- */
.floating-ai { position: absolute; right: 30rpx; top: 180rpx; width: 120rpx; height: 120rpx; display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 50; }
.ai-halo { position: absolute; width: 100%; height: 100%; border-radius: 50%; background: #d4af37; animation: breathe 2s infinite ease-in-out; }
@keyframes breathe { 0% { transform: scale(0.9); opacity: 0.3; } 50% { transform: scale(1.3); opacity: 0; } 100% { transform: scale(0.9); opacity: 0.3; } }
.ai-avatar { width: 100rpx; height: 100rpx; border-radius: 50%; border: 4rpx solid #fce8b2; position: relative; z-index: 2; background-color: #2c1d11; }
.ai-badge { position: absolute; bottom: -10rpx; background: linear-gradient(to right, #d4af37, #aa8529); color: #1a0f08; font-size: 20rpx; font-weight: bold; padding: 4rpx 16rpx; border-radius: 20rpx; z-index: 3; box-shadow: 0 4rpx 10rpx rgba(0,0,0,0.5); }
.scroll-area { position: absolute; bottom: 220rpx; left: 0; width: 100%; padding: 0 30rpx; box-sizing: border-box; display: flex; align-items: flex-end; z-index: 30; }
.jade-tags { display: flex; margin-right: 20rpx; gap: 10rpx; }
.jade-btn { 
  width: 54rpx; /* 稍微加宽一点点，让比例更协调 (从50改到54) */
  
  /* 👇 【核心修改1：把上下内边距从 20rpx 改大到 36rpx 甚至 40rpx】 */
  padding: 33rpx 0; 
  
  background: linear-gradient(to bottom, #4a3320, #2c1d11); 
  border: 1px solid rgba(212, 175, 55, 0.3); 
  border-radius: 8rpx; 
  display: flex; 
  flex-direction: column; 
  align-items: center; 
  justify-content: center; 
  
  /* 👇 【核心修改2：把文字之间的垂直间距从 6rpx 改大到 12rpx】 */
  gap: 12rpx; 
  
  box-shadow: 4rpx 4rpx 10rpx rgba(0,0,0,0.5); 
  transition: all 0.3s; 
}
.active-jade { background: linear-gradient(to bottom, #8c2a2a, #521818); border-color: #fce8b2; }
.jade-btn text { color: #ccc; font-size: 24rpx; font-weight: bold; }
.active-jade text { color: #fce8b2; }
.main-scroll { flex: 1; height: 220rpx; display: flex; align-items: center; position: relative; transition: all 0.3s; }
.main-scroll:active { transform: scale(0.98); }
.scroll-axis { width: 24rpx; height: 100%; background: linear-gradient(to right, #3d2b1f, #5e4331, #3d2b1f); border: 1px solid #1a0f08; border-radius: 12rpx; z-index: 2; box-shadow: 2rpx 0 10rpx rgba(0,0,0,0.8); }
.left-axis { margin-right: -10rpx; }
.right-axis { margin-left: -10rpx; }
.scroll-content { flex: 1; height: 190rpx; background: linear-gradient(to right, #eac56b, #fce8b2, #eac56b); display: flex; flex-direction: column; justify-content: center; align-items: center; box-shadow: inset 0 0 20rpx rgba(186, 137, 49, 0.4); position: relative; z-index: 1; }
.scroll-content::before { content: ''; position: absolute; top: 0; left: 0; width: 100%; height: 100%; background-image: repeating-linear-gradient(90deg, transparent, transparent 10rpx, rgba(0,0,0,0.03) 10rpx, rgba(0,0,0,0.03) 12rpx); }
.scroll-title { color: #8c2a2a; font-size: 38rpx; font-weight: bold; font-family: '楷体', 'KaiTi', serif; text-shadow: 1px 1px 0 rgba(255,255,255,0.5); z-index: 2;}
.scroll-sub { color: #521818; font-size: 22rpx; margin-top: 10rpx; font-weight: bold; z-index: 2;}
.custom-tabbar { position: absolute; bottom: 0; left: 0; width: 100%; height: 140rpx; background: #1a0f08; border-top: 1px solid rgba(212, 175, 55, 0.3); display: flex; justify-content: space-around; align-items: center; padding-bottom: constant(safe-area-inset-bottom); padding-bottom: env(safe-area-inset-bottom); z-index: 100; }
.bar-item { display: flex; flex-direction: column; align-items: center; }
.icon { font-size: 44rpx; margin-bottom: 6rpx; filter: grayscale(100%); opacity: 0.5; }
.text { color: #888; font-size: 22rpx; }
.active-bar .icon { filter: none; opacity: 1; }
.active-bar .text { color: #d4af37; font-weight: bold; }
/* --- 左上角国风挂件 --- */
.top-left-ornament {
  position: absolute;
  top: 120rpx; /* 根据你的手机状态栏高度可微调 */
  left: 40rpx;
  display: flex;
  align-items: flex-start;
  z-index: 40; /* 确保在背景之上 */
}

/* 朱砂印章设计 */
/* 朱砂印章设计 (全面放大) */
.red-stamp {
  width: 76rpx;  /* 从 50 放大到 76 */
  height: 76rpx; /* 从 50 放大到 76 */
  background-color: #9b2226; 
  border: 4rpx solid #9b2226; /* 边框加粗 */
  border-radius: 8rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 24rpx; /* 间距稍微拉开 */
  opacity: 0.95; /* 增加不透明度，让红色更实 */
  box-shadow: 4rpx 6rpx 12rpx rgba(0,0,0,0.6);
}
.red-stamp text {
  color: #fce8b2; 
  font-size: 32rpx; /* 字体从 20 放大到 32 */
  font-family: '楷体', 'KaiTi', serif;
  font-weight: bold;
}

/* 竖排日期文字 (清晰度拉满) */
.lunar-date {
  display: flex;
  align-items: center;
}
.date-text {
  writing-mode: vertical-rl; 
  letter-spacing: 12rpx; /* 增加字间距，更有古籍感 */
  color: rgba(255, 255, 255, 0.9); /* 提高白色亮度 */
  font-size: 34rpx; /* 字体从 24 放大到 34 */
  font-family: '楷体', 'KaiTi', serif;
  font-weight: bold; /* 加上粗体 */
  /* 【关键】加深双重阴影，哪怕背景是纯白色，字也能清楚跳出来 */
  text-shadow: 0 4rpx 8rpx rgba(0,0,0,0.9), 0 0 4rpx rgba(0,0,0,0.8); 
}
.date-line {
  width: 4rpx; /* 金线加粗 */
  height: 180rpx; /* 金线拉长匹配大字 */
  background: linear-gradient(to bottom, #d4af37, transparent);
  margin-left: 20rpx;
}
</style>