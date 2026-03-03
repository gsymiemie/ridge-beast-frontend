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
import { ref, onMounted, reactive } from 'vue'
import * as echarts from 'echarts'

const chartRef = ref(null)
let myChart = null 

// 状态控制
const currentMode = ref('theme') 
const showPopup = ref(false)
const currentNode = ref({})

// 🌟 将静态字典改为响应式对象，等待后端数据注入
const nodeDetailDict = reactive({})

// 存储从后端拉取的真实图谱数据
const backendGraphData = ref({ nodes: [], links: [] })

const goBack = () => uni.navigateBack()
const closePopup = () => showPopup.value = false
const goTo3D = () => {
  uni.showToast({ title: '即将开启法阵...', icon: 'none' })
  setTimeout(() => { uni.redirectTo({ url: '/pages/index/index' }) }, 1000)
}

// 🌐 核心魔法：从 Spring Boot 后端拉取图谱真实数据
const fetchGraphData = () => {
  uni.request({
    url: 'http://localhost:8080/api/graph/data', // 请求咱们刚写的接口
    method: 'GET',
    success: (res) => {
      if (res.data.code === 200) {
        const { nodes, links } = res.data.data;
        
        // 1. 动态生成弹窗字典 (数据驱动弹窗)
        nodes.forEach(node => {
          if (node.briefDesc || node.avatarUrl) {
            nodeDetailDict[node.name] = {
              desc: node.briefDesc || '这只瑞兽十分神秘，暂无过多记载...',
              img: node.avatarUrl || '' // 数据库里存的图片路径
            }
          }
        });

        // 2. 格式化节点数据给 ECharts
        backendGraphData.value.nodes = nodes.map(n => ({
          name: n.name,
          symbolSize: n.symbolSize || 50,
          category: n.category,
          // 将数据库里的颜色应用到 ECharts 节点样式上
          itemStyle: { color: n.color || '#eac56b' },
          label: { color: (n.category === '核心' || n.category === '分类') ? '#fce8b2' : '#1a0f08' }
        }));

        // 3. 格式化连线数据给 ECharts
        backendGraphData.value.links = links.map(l => ({
          source: l.sourceName,
          target: l.targetName,
          value: l.relationValue
        }));

        console.log("🔥 后端图谱数据加载完毕！", backendGraphData.value);
        
        // 如果当前正处于关系图谱模式，自动刷新画布展示最新数据
        if (currentMode.value === 'relation' && myChart) {
          switchMode('relation')
        }
      }
    },
    fail: (err) => {
      console.error("请求后端接口失败，请检查 SpringBoot 是否启动", err);
      uni.showToast({ title: '图谱数据加载失败', icon: 'none' });
    }
  });
}

// 🌟 模式 A：主题图谱 (树状径向展开结构 - 保持不变，作为大类引导)
const getThemeOption = () => {
  return {
    backgroundColor: 'transparent',
    series: [
      {
        type: 'tree',
        layout: 'radial', 
        symbol: 'circle',
        symbolSize: (value, params) => params.data.children ? 50 : 40, 
        initialTreeDepth: 1, 
        animationDurationUpdate: 750,
        itemStyle: { color: '#2c1d11', borderColor: '#d4af37', borderWidth: 2, shadowBlur: 10, shadowColor: 'rgba(212,175,55,0.5)' },
        lineStyle: { color: '#d4af37', width: 2, curveness: 0.3, opacity: 0.6 },
        label: { show: true, position: 'inside', color: '#fce8b2', fontSize: 12, fontFamily: '楷体', fontWeight: 'bold' },
        data: [
          {
            name: '太和脊兽',
            itemStyle: { color: '#9b2226', borderColor: '#fce8b2', borderWidth: 3 }, 
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

// 🌟 模式 B：关系图谱 (力导向网状结构 - 彻底接入后端真实数据)
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
        // 关键点：直接把后端的数据喂给 ECharts！
        data: backendGraphData.value.nodes,
        links: backendGraphData.value.links
      }
    ]
  }
}

// 🌟 切换模式的核心逻辑
const switchMode = (mode) => {
  if (currentMode.value === mode || !myChart) return
  currentMode.value = mode
  
  myChart.clear()
  if (mode === 'theme') {
    myChart.setOption(getThemeOption(), true)
  } else {
    // 切换到关系图谱时，如果后端数据还没回来，可以给个提示
    if (backendGraphData.value.nodes.length === 0) {
      uni.showToast({ title: '图谱羁绊正在构建中...', icon: 'none' })
    }
    myChart.setOption(getRelationOption(), true)
  }
}

onMounted(() => {
  // 页面加载时，立刻向后端请求真实数据
  fetchGraphData();

  setTimeout(async () => {
    if (!chartRef.value) return
    myChart = await chartRef.value.init(echarts)
    
    // 初始化加载【主题模式】
    myChart.setOption(getThemeOption())
    
    // 监听节点点击事件
    myChart.on('click', (params) => {
      const nodeName = params.name
      // 只要后端返回的数据里配了详情，就能弹窗！
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