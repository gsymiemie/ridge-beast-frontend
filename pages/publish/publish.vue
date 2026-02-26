<template>
  <view class="publish-container">
    <view class="nav-bar">
      <view class="back-btn" @click="goBack">
        <text class="back-icon">‹</text>
      </view>
      <text class="nav-title">发布脊兽寻踪</text>
    </view>

    <view class="content-wrapper">
      <view class="upload-area" @click="chooseImage">
        <image v-if="tempImagePath" :src="tempImagePath" mode="aspectFill" class="preview-img"></image>
        <view v-else class="upload-placeholder">
          <text class="plus-icon">+</text>
          <text class="upload-text">点击拍照或上传相册图片</text>
        </view>
      </view>

      <view class="form-area">
        <view class="form-item">
          <text class="label">脊兽品种</text>
          <picker @change="onBeastChange" :value="beastIndex" :range="beastArray">
            <view class="picker-box">
              <text class="picker-text">{{ beastArray[beastIndex] || '请选择你拍到的脊兽种类' }}</text>
              <text class="arrow">›</text>
            </view>
          </picker>
        </view>

        <view class="form-item">
          <text class="label">发现地点</text>
          <input class="input-box" v-model="location" placeholder="例如：故宫太和殿、天坛..." placeholder-style="color:#666;" />
        </view>

        <view class="form-item">
          <text class="label">寻踪笔记</text>
          <textarea class="textarea-box" v-model="content" placeholder="分享一下你发现这只脊兽时的感悟吧，或是它的独特之处..." placeholder-style="color:#666;" :maxlength="200"></textarea>
          <text class="word-count">{{ content.length }}/200</text>
        </view>
      </view>

      <view class="publish-btn" :class="{ 'btn-active': isReadyToPublish }" @click="submitPost">
        <text>⚡ 立即发布</text>
      </view>
    </view>
  </view>
</template>

<script setup>
import { ref, computed } from 'vue'

// 表单数据状态
const tempImagePath = ref('')
const beastArray = ref(['螭吻', '嘲风', '仙人骑凤', '狻猊', '斗牛', '其他/不确定'])
const beastIndex = ref(-1)
const location = ref('')
const content = ref('')

// 计算属性：是否满足发布条件（必须有图和选择种类）
const isReadyToPublish = computed(() => {
  return tempImagePath.value !== '' && beastIndex.value !== -1
})

const goBack = () => {
  uni.navigateBack()
}

// 唤起相册或相机
const chooseImage = () => {
  uni.chooseImage({
    count: 1, // 暂定每次上传1张
    sizeType: ['compressed'], // 压缩图
    sourceType: ['album', 'camera'], // 支持相册和相机
    success: (res) => {
      // 获取到本地临时路径，渲染到界面上
      tempImagePath.value = res.tempFilePaths[0]
    }
  })
}

// 提交发布（真实对接后端）
const submitPost = () => {
  if (!isReadyToPublish.value) {
    uni.showToast({ title: '请先上传图片并选择脊兽种类', icon: 'none' })
    return
  }

  uni.showLoading({ title: '正在发布...' })

  // 调用小程序的原生上传 API
  uni.uploadFile({
    url: 'http://localhost:8080/api/posts/publish', // 你刚刚写好的 Spring Boot 接口
    filePath: tempImagePath.value,                  // 刚才选择的本地图片路径
    name: 'file',                                   // 必须和后端 @RequestParam("file") 名字一致
    formData: {
      'beastName': beastArray.value[beastIndex.value], // 获取选中的脊兽名字
      'location': location.value,
      'content': content.value
    },
    success: (uploadFileRes) => {
      uni.hideLoading()
      
      // 注意：uploadFile 返回的 data 默认是字符串格式，需要手动转成 JSON 对象
      const resData = JSON.parse(uploadFileRes.data)
      
      if (resData.code === 200) {
        uni.showToast({ title: '发布成功！', icon: 'success' })
        // 延迟 1.5 秒后自动返回上一页
        setTimeout(() => {
          goBack()
        }, 1500)
      } else {
        uni.showToast({ title: resData.message || '发布失败', icon: 'error' })
      }
    },
    fail: (err) => {
      uni.hideLoading()
      console.error("上传失败:", err)
      uni.showToast({ title: '网络通讯失败，请检查后端', icon: 'error' })
    }
  })
}

// 种类选择器变化
const onBeastChange = (e) => {
  // 关键修复：把微信返回的字符串索引强制转换为 10 进制的数字
  beastIndex.value = parseInt(e.detail.value, 10)
}
</script>

<style>
page {
  background-color: #0f121a;
  height: 100%;
}
.publish-container {
  width: 100vw;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

/* 顶部导航 */
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
.back-icon {
  color: #fff;
  font-size: 60rpx;
  font-weight: 300;
}
.nav-title {
  color: #d4af37;
  font-size: 34rpx;
  font-weight: bold;
}

/* 主体内容 */
.content-wrapper {
  padding: 40rpx 30rpx;
  flex: 1;
}

/* 上传区 */
.upload-area {
  width: 100%;
  height: 400rpx;
  background: rgba(255, 255, 255, 0.05);
  border: 2px dashed rgba(212, 175, 55, 0.3);
  border-radius: 30rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  margin-bottom: 50rpx;
}
.upload-placeholder {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.plus-icon {
  font-size: 80rpx;
  color: rgba(212, 175, 55, 0.6);
  font-weight: 300;
  margin-bottom: 10rpx;
}
.upload-text {
  font-size: 28rpx;
  color: #666;
}
.preview-img {
  width: 100%;
  height: 100%;
}

/* 表单区 */
.form-area {
  margin-bottom: 60rpx;
}
.form-item {
  margin-bottom: 40rpx;
  position: relative;
}
.label {
  display: block;
  font-size: 28rpx;
  color: #d4af37;
  margin-bottom: 16rpx;
  font-weight: bold;
}
.picker-box, .input-box {
  width: 100%;
  height: 100rpx;
  background: rgba(255, 255, 255, 0.08);
  border-radius: 20rpx;
  padding: 0 30rpx;
  box-sizing: border-box;
  display: flex;
  align-items: center;
  justify-content: space-between;
}
.picker-text, .input-box {
  color: #fff;
  font-size: 28rpx;
}
.arrow {
  color: #666;
  font-size: 40rpx;
}
.textarea-box {
  width: 100%;
  height: 240rpx;
  background: rgba(255, 255, 255, 0.08);
  border-radius: 20rpx;
  padding: 30rpx;
  box-sizing: border-box;
  color: #fff;
  font-size: 28rpx;
  line-height: 1.5;
}
.word-count {
  position: absolute;
  bottom: 20rpx;
  right: 30rpx;
  font-size: 24rpx;
  color: #666;
}

/* 发布按钮 */
.publish-btn {
  width: 100%;
  height: 100rpx;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 50rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  color: rgba(255, 255, 255, 0.4);
  font-size: 32rpx;
  font-weight: bold;
  transition: all 0.3s;
}
.btn-active {
  background: linear-gradient(135deg, #d4af37, #aa8529);
  color: #fff;
  box-shadow: 0 10rpx 30rpx rgba(212, 175, 55, 0.3);
}
</style>