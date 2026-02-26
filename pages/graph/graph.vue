<template>
  <view class="graph-container">
    <view class="nav-bar">
      <view class="back-btn" @click="goBack">
        <text class="back-icon">‹</text>
      </view>
      <text class="nav-title">脊兽文化全景图谱</text>
    </view>

    <view class="chart-wrapper">
      <l-echart ref="chartRef" style="width: 100%; height: 100%; display: block;"></l-echart>
    </view>
    
    <view class="tips">
      <text>?? 提示：支持双指缩放、单指拖拽节点</text>
    </view>
  </view>
</template>

<script setup>
import { ref, nextTick } from 'vue'
import { onLoad, onReady } from '@dcloudio/uni-app' 
import * as echarts from 'echarts'

const chartRef = ref(null)

const goBack = () => {
  uni.navigateBack()
}

const renderGraph = () => {
  uni.showLoading({ title: '引力计算中...' })

  uni.request({
    url: 'http://localhost:8080/api/graph/data',
    method: 'GET',
    success: (res) => {
      uni.hideLoading()
      if (res.data && res.data.code === 200) {
        const graphData = res.data.data
        console.log("✅ 成功获取图谱数据:", graphData) 

        // 增强版：给视图渲染多一点时间
        nextTick(() => {
          if (!chartRef.value) {
            console.error("❌ 依然找不到 ECharts 容器！请检查 lime-echart 插件是否在 uni_modules 目录下。")
            return
          }

          chartRef.value.init(echarts, (chart) => {
            const option = {
              backgroundColor: 'transparent',
              tooltip: {
                show: true,
                formatter: '{b}' 
              },
              categories: [
                { name: '核心', itemStyle: { color: '#d4af37', shadowBlur: 20, shadowColor: '#d4af37' } },
                { name: '脊兽', itemStyle: { color: '#ff6b6b', shadowBlur: 10, shadowColor: '#ff6b6b' } },
                { name: '建筑', itemStyle: { color: '#4facfe', shadowBlur: 10, shadowColor: '#4facfe' } }
              ],
              series: [{
                type: 'graph',
                layout: 'force',
                data: graphData.nodes,
                links: graphData.links,
                categories: [ {name:'核心'}, {name:'脊兽'}, {name:'建筑'} ],
                roam: true, 
                label: {
                  show: true,
                  position: 'right',
                  color: '#ffffff',
                  fontSize: 14,
                  textShadowBlur: 4,
                  textShadowColor: '#000'
                },
                force: {
                  repulsion: 300, 
                  edgeLength: [60, 120], 
                  gravity: 0.1 
                },
                lineStyle: {
                  color: 'source',
                  curveness: 0.3, 
                  width: 1.5,
                  opacity: 0.7
                }
              }]
            }
            chart.setOption(option)
          })
        })
      }
    },
    fail: (err) => {
      uni.hideLoading()
      uni.showToast({ title: '获取图谱失败', icon: 'error' })
    }
  })
}

onReady(() => {
  // 延迟 800ms，确保微信原生的 Canvas 组件彻底挂载完毕
  setTimeout(() => {
    renderGraph()
  }, 800)
})
</script>

<style>
page {
  background-color: #0f121a; 
  height: 100%;
}
.graph-container {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
}

.nav-bar {
  height: 180rpx;
  padding-top: 80rpx; 
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  background: linear-gradient(to bottom, rgba(15, 18, 26, 1), rgba(15, 18, 26, 0));
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
.back-icon {
  color: #fff;
  font-size: 60rpx;
  font-weight: 300;
}
.nav-title {
  color: #d4af37;
  font-size: 34rpx;
  font-weight: bold;
  letter-spacing: 4rpx;
}

/* 关键修复4：外层容器也必须强制高度 */
.chart-wrapper {
  flex: 1;
  width: 100%;
  height: 100%; 
  position: relative;
}

.tips {
  position: absolute;
  bottom: 80rpx;
  width: 100%;
  text-align: center;
  pointer-events: none; 
}
.tips text {
  color: rgba(255, 255, 255, 0.5);
  font-size: 24rpx;
  background: rgba(0, 0, 0, 0.4);
  padding: 10rpx 30rpx;
  border-radius: 30rpx;
  border: 1px solid rgba(255, 255, 255, 0.1);
}
</style>