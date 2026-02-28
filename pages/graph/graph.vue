<template>
  <view class="graph-container">
    
    <view class="nav-header">
      <view class="back-btn" @click="goBack"><text>⬅ 返回</text></view>
      <text class="title">太和 · 寻踪纪</text>
      <view class="placeholder"></view>
    </view>

    <view class="mode-tabs">
      <view class="tab-item" :class="{'tab-active': currentMode === 'theme'}" @click="switchMode('theme')">
        <text>主题图谱</text>
        <view class="tab-line" v-if="currentMode === 'theme'"></view>
      </view>
      <view class="tab-item" :class="{'tab-active': currentMode === 'relation'}" @click="switchMode('relation')">
        <text>关系图谱</text>
        <view class="tab-line" v-if="currentMode === 'relation'"></view>
      </view>
    </view>

    <view class="desc-box">
      <text class="subtitle">{{ currentMode === 'theme' ? '按类索骥，点击展开枝叶' : '万物有灵，探寻错综羁绊' }}</text>
    </view>

    <view class="chart-wrapper" :class="{'chart-hidden': showPopup}">
      <l-echart ref="chartRef" style="width: 100%; height: 100%;"></l-echart>
    </view>

    <view class="detail-mask" :class="{'mask-active': showPopup}" @click="closePopup"></view>
    <view class="detail-popup" :class="{'popup-active': showPopup}">
      <view class="popup-inner">
        <view class="close-btn" @click="closePopup"><text>✕</text></view>
        <view class="popup-header">
          <view class="ornament-line"></view>
          <text class="beast-title">{{ currentNode.name }}</text>
          <view class="ornament-line"></view>
        </view>
        <scroll-view scroll-y class="popup-content">
          <text class="beast-desc">{{ currentNode.desc }}</text>
          <view class="img-frame" v-if="currentNode.img">
            <image :src="currentNode.img" mode="aspectFill" class="beast-img"></image>
          </view>
        </scroll-view>
        <view class="action-btn" @click="goTo3D">
          <text>鉴赏 3D 瑞兽</text>
        </view>
      </view>
    </view>

  </view>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import * as echarts from 'echarts'

const chartRef = ref(null)
let myChart = null // 存储图表实例，用于切换模式

// 状态控制
const currentMode = ref('theme') // 默认主题模式
const showPopup = ref(false)
const currentNode = ref({})

// 节点详情字典
const nodeDetailDict = {
  '嘲风': { desc: '平生好险，威慑邪祟。嘲风以其无畏的姿态镇守于太和殿殿脊之上，不仅增添了殿宇的威严，更寄托了古人辟邪挡灾的美好愿景。', img: '/static/images/chaofeng.jpg' },
  '螭吻': { desc: '口润嗓粗，避火防灾。相传螭吻乃龙之九子之一，喜好吞火，故被安置于建筑正脊两端，作为镇火的守护神。', img: '/static/images/chiwen.jpg' },
  '仙人骑凤': { desc: '排头指路，逢凶化吉。传说齐国国君战败被追，遇水阻拦，危急时刻飞来一只凤凰载他渡水。此后仙人骑凤便成了绝处逢生的象征。', img: '/static/images/immortal.jpg' }
}

const goBack = () => uni.navigateBack()
const closePopup = () => showPopup.value = false
const goTo3D = () => {
  uni.showToast({ title: '即将开启法阵...', icon: 'none' })
  setTimeout(() => { uni.redirectTo({ url: '/pages/index/index' }) }, 1000)
}

// 🌟 模式 A：主题图谱 (树状径向展开结构)
const getThemeOption = () => {
  return {
    backgroundColor: 'transparent',
    series: [
      {
        type: 'tree',
        layout: 'radial', // 径向布局 (环形辐射)
        symbol: 'circle',
        symbolSize: (value, params) => params.data.children ? 50 : 40, // 有子节点的圆大一点
        initialTreeDepth: 1, // 默认只展开第一层级！
        animationDurationUpdate: 750,
        itemStyle: { color: '#2c1d11', borderColor: '#d4af37', borderWidth: 2, shadowBlur: 10, shadowColor: 'rgba(212,175,55,0.5)' },
        lineStyle: { color: '#d4af37', width: 2, curveness: 0.3, opacity: 0.6 },
        label: { show: true, position: 'inside', color: '#fce8b2', fontSize: 12, fontFamily: '楷体', fontWeight: 'bold' },
        data: [
          {
            name: '太和脊兽',
            itemStyle: { color: '#9b2226', borderColor: '#fce8b2', borderWidth: 3 }, // 中心点朱红色
            children: [
              {
                name: '神兽图鉴',
                itemStyle: { color: '#8c2a2a' },
                children: [
                  { name: '仙人骑凤' }, { name: '龙' }, { name: '凤' }, { name: '狮子' }, 
                  { name: '嘲风' }, { name: '螭吻' }, { name: '行什' }
                ]
              },
              {
                name: '文化寓意',
                itemStyle: { color: '#8c2a2a' },
                children: [
                  { name: '逢凶化吉' }, { name: '威慑邪祟' }, { name: '避火防灾' }, { name: '皇权至尊' }
                ]
              },
              {
                name: '建筑建制',
                itemStyle: { color: '#8c2a2a' },
                children: [
                  { name: '最高建制' }, { name: '垂脊前段' }, { name: '正脊两端' }
                ]
              }
            ]
          }
        ]
      }
    ]
  }
}

// 🌟 模式 B：关系图谱 (力导向网状结构，就是我们之前的代码)
const getRelationOption = () => {
  return {
    backgroundColor: 'transparent',
    series: [
      {
        type: 'graph', layout: 'force',
        force: { repulsion: 600, edgeLength: [60, 150], gravity: 0.1 },
        roam: true, draggable: true,
        label: { show: true, position: 'inside', color: '#1a0f08', fontWeight: 'bold', fontFamily: '楷体' },
        edgeLabel: { show: true, fontSize: 10, color: 'rgba(212, 175, 55, 0.8)', formatter: '{c}' },
        lineStyle: { color: '#d4af37', width: 1.5, opacity: 0.6, curveness: 0.15 },
        itemStyle: { borderColor: '#fce8b2', borderWidth: 2, shadowBlur: 15, shadowColor: 'rgba(212, 175, 55, 0.6)' },
        data: [
          { name: '太和殿脊兽', symbolSize: 80, itemStyle: { color: '#9b2226' }, label: { color: '#fce8b2' } },
          { name: '最高建制', symbolSize: 55, itemStyle: { color: '#8c2a2a' }, label: { color: '#fce8b2' } },
          { name: '仙人骑凤', symbolSize: 60, itemStyle: { color: '#d4af37' } },
          { name: '嘲风', symbolSize: 50, itemStyle: { color: '#eac56b' } },
          { name: '螭吻', symbolSize: 50, itemStyle: { color: '#eac56b' } },
          { name: '避火防灾', symbolSize: 35, itemStyle: { color: '#523620' }, label: { color: '#ccc' } },
          { name: '逢凶化吉', symbolSize: 35, itemStyle: { color: '#523620' }, label: { color: '#ccc' } }
        ],
        links: [
          { source: '太和殿脊兽', target: '最高建制', value: '所属' },
          { source: '最高建制', target: '仙人骑凤', value: '排头' },
          { source: '最高建制', target: '嘲风', value: '顺位3' },
          { source: '最高建制', target: '螭吻', value: '正脊' },
          { source: '仙人骑凤', target: '逢凶化吉', value: '寓意' },
          { source: '螭吻', target: '避火防灾', value: '技能' }
        ]
      }
    ]
  }
}

// 🌟 切换模式的核心逻辑
const switchMode = (mode) => {
  if (currentMode.value === mode || !myChart) return
  currentMode.value = mode
  
  // 清除旧图表，重新注入新配置，开启 true 表示不合并配置
  myChart.clear()
  if (mode === 'theme') {
    myChart.setOption(getThemeOption(), true)
  } else {
    myChart.setOption(getRelationOption(), true)
  }
}

onMounted(() => {
  setTimeout(async () => {
    if (!chartRef.value) return
    myChart = await chartRef.value.init(echarts)
    
    // 初始化加载【主题模式】
    myChart.setOption(getThemeOption())
    
    // 监听节点点击事件
    myChart.on('click', (params) => {
      const nodeName = params.name
      // 只有我们在字典里配置了详情的节点，才会弹出卡片
      // 如果点击的是 "神兽图鉴" 这种分类节点，Echarts 会自动处理展开/折叠动画
      if (nodeDetailDict[nodeName]) {
        currentNode.value = { name: nodeName, ...nodeDetailDict[nodeName] }
        showPopup.value = true
      }
    })
  }, 300)
})
</script>

<style>
page { margin: 0; padding: 0; background-color: #1a0f08; height: 100%; color: #fce8b2; overflow: hidden; }
.graph-container { width: 100vw; height: 100vh; display: flex; flex-direction: column; background: radial-gradient(circle at center, #2c1d11, #1a0f08); position: relative; }

.nav-header { padding: 100rpx 30rpx 10rpx 30rpx; display: flex; justify-content: space-between; align-items: center; }
.back-btn { color: #d4af37; font-size: 28rpx; padding: 10rpx 20rpx; border: 1px solid #d4af37; border-radius: 30rpx; z-index: 50;}
.title { font-size: 36rpx; font-weight: bold; font-family: '楷体', 'KaiTi', serif; letter-spacing: 4rpx; }
.placeholder { width: 100rpx; }

/* 🌟 双模式 Tab 样式 */
.mode-tabs { display: flex; justify-content: center; margin-top: 20rpx; border-bottom: 1px solid rgba(212, 175, 55, 0.2); padding-bottom: 20rpx; }
.tab-item { margin: 0 40rpx; font-size: 30rpx; color: #888; display: flex; flex-direction: column; align-items: center; transition: all 0.3s; padding: 10rpx; }
.tab-active text { color: #d4af37; font-weight: bold; font-family: '楷体', 'KaiTi', serif; font-size: 34rpx; }
.tab-line { width: 40rpx; height: 4rpx; background: #9b2226; margin-top: 10rpx; border-radius: 4rpx; }

.desc-box { display: flex; flex-direction: column; align-items: center; margin-top: 30rpx; }
.subtitle { color: rgba(212, 175, 55, 0.6); font-size: 24rpx; letter-spacing: 4rpx; }

/* Canvas 容器与隐藏魔法 */
.chart-wrapper { flex: 1; width: 100%; position: relative; z-index: 10; height: 0; margin-top: 10rpx; transition: opacity 0.3s ease; }
.chart-hidden { position: fixed !important; left: -9999px !important; top: -9999px !important; opacity: 0; }

/* 详情弹窗样式 */
.detail-mask { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.6); z-index: 90; opacity: 0; pointer-events: none; transition: opacity 0.3s; }
.mask-active { opacity: 1; pointer-events: auto; }

.detail-popup { position: absolute; bottom: 0; left: 0; width: 100%; height: 75vh; background: #eac56b; border-radius: 40rpx 40rpx 0 0; z-index: 100; transform: translateY(100%); transition: transform 0.4s cubic-bezier(0.25, 1, 0.5, 1); box-shadow: 0 -10rpx 40rpx rgba(0,0,0,0.8); }
.popup-active { transform: translateY(0); }

/* 修复了上一轮图片丢失的报错，改用纯 CSS 径向渐变模拟宣纸光泽 */
.popup-inner { width: 100%; height: 100%; background: radial-gradient(circle at center, rgba(255,255,255,0.3) 0%, transparent 80%); padding: 60rpx 40rpx; box-sizing: border-box; display: flex; flex-direction: column; align-items: center; position: relative; }
.close-btn { position: absolute; top: 30rpx; right: 40rpx; width: 60rpx; height: 60rpx; display: flex; align-items: center; justify-content: center; color: #521818; font-size: 40rpx; border: 2rpx solid rgba(82,24,24,0.3); border-radius: 50%; }

.popup-header { display: flex; align-items: center; justify-content: center; width: 100%; margin-bottom: 40rpx; }
.ornament-line { flex: 1; height: 2rpx; background: linear-gradient(to right, transparent, #8c2a2a, transparent); }
.beast-title { font-size: 56rpx; color: #521818; font-weight: bold; font-family: '楷体', 'KaiTi', serif; margin: 0 40rpx; letter-spacing: 8rpx; text-shadow: 2rpx 2rpx 4rpx rgba(255,255,255,0.5); }

.popup-content { flex: 1; width: 100%; display: flex; flex-direction: column; align-items: center; }
.beast-desc { font-size: 30rpx; color: #2c1d11; line-height: 1.8; text-align: justify; text-indent: 60rpx; display: block; margin-bottom: 40rpx; }
.img-frame { width: 100%; padding: 10rpx; background: #d4af37; border-radius: 16rpx; box-shadow: 0 10rpx 20rpx rgba(0,0,0,0.3); margin-bottom: 40rpx; }
.beast-img { width: 100%; height: 360rpx; border-radius: 8rpx; }

.action-btn { width: 80%; padding: 24rpx 0; background: linear-gradient(to right, #8c2a2a, #521818); border-radius: 50rpx; text-align: center; box-shadow: 0 8rpx 20rpx rgba(82,24,24,0.6); border: 2rpx solid #fce8b2; margin-top: auto; transition: all 0.2s; }
.action-btn:active { transform: scale(0.95); }
.action-btn text { color: #fce8b2; font-size: 32rpx; font-weight: bold; letter-spacing: 4rpx; }
</style>