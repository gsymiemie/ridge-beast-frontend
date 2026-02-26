<template>
  <view class="chat-container">
    <view class="nav-bar">
      <view class="back-btn" @click="goBack">
        <text class="back-icon">‹</text>
      </view>
      <view class="title-area">
        <text class="nav-title">脊兽智能体</text>
        <text class="nav-subtitle">Ta 正在屋脊上看着你...</text>
      </view>
    </view>

    <scroll-view class="chat-list" scroll-y :scroll-into-view="scrollToView">
      <view style="height: 180rpx;"></view> 

      <block v-for="(msg, index) in messageList" :key="index">
        <view v-if="msg.role === 'ai'" class="message-row ai-row" :id="'msg-' + index">
          <view class="avatar ai-avatar">🐲</view>
          <view class="bubble ai-bubble">
            <rich-text :nodes="formatMessage(msg.content)"></rich-text>
          </view>
        </view>

        <view v-else class="message-row user-row" :id="'msg-' + index">
          <view class="bubble user-bubble">
            <text class="msg-text">{{ msg.content }}</text>
          </view>
          <view class="avatar user-avatar">👤</view>
        </view>
      </block>
      
      <view v-if="isThinking" class="message-row ai-row" id="thinking">
        <view class="avatar ai-avatar">🐲</view>
        <view class="bubble ai-bubble thinking-bubble">
          <text class="typing-dot">.</text><text class="typing-dot">.</text><text class="typing-dot">.</text>
        </view>
      </view>
      
      <view style="height: 160rpx;"></view>
    </scroll-view>

    <view class="input-area">
      <scroll-view class="quick-prompts" scroll-x :show-scrollbar="false">
        <view class="prompt-tag" @click="sendQuickMsg('介绍一下太和殿的十脊兽')">📜 介绍太和殿十脊兽</view>
        <view class="prompt-tag" @click="sendQuickMsg('你和普通石狮子有什么区别？')">🤔 你和石狮子的区别？</view>
        <view class="prompt-tag" @click="sendQuickMsg('帮我生成一张手机壁纸')">✨ 生成脊兽壁纸</view>
      </scroll-view>
      
      <view class="input-box">
        <input 
          class="chat-input" 
          v-model="inputText" 
          placeholder="问问脊兽千年的故事..." 
          placeholder-style="color: rgba(255,255,255,0.3);"
          @confirm="sendMessage"
        />
        <view class="send-btn" :class="{ 'btn-active': inputText.length > 0 }" @click="sendMessage">
          <text>发送</text>
        </view>
      </view>
    </view>
  </view>
</template>

<script setup>
import { ref, nextTick } from 'vue'

// 解析大模型返回的文本，将换行符和所有 Markdown 链接强制转换为图片
const formatMessage = (text) => {
  if (!text) return ''
  
  // 匹配标准图片 ![alt](url) 以及普通链接 [alt](url)
  let html = text.replace(/!?\[([^\]]*)\]\((https?:\/\/[^\)]+)\)/g, (match, alt, url) => {
    // 强制将提取出的 URL 塞进 img 标签里渲染
    return `<img src="${url}" alt="${alt}" style="max-width: 100%; border-radius: 16rpx; margin-top: 20rpx; margin-bottom: 20rpx; display: block; box-shadow: 0 4rpx 15rpx rgba(0,0,0,0.2);" />`
  })
  
  // 将换行符 \n 转换为 <br/>
  html = html.replace(/\n/g, '<br/>')
  return html
}

// 聊天记录状态
const messageList = ref([
  { role: 'ai', content: '吾乃太和殿正脊之上的嘲风。在这紫禁城之巅吹了六百年的风，你有什么想问我的吗？' }
])
const inputText = ref('')
const isThinking = ref(false)
const scrollToView = ref('')

const goBack = () => {
  uni.navigateBack()
}

// 发送快捷提示词
const sendQuickMsg = (text) => {
  inputText.value = text
  sendMessage()
}

// 核心发送逻辑（真实对接 Spring Boot 与大模型）
const sendMessage = () => {
  if (!inputText.value.trim() || isThinking.value) return

  const userText = inputText.value
  
  // 1. 把用户的话加到界面上
  messageList.value.push({ role: 'user', content: userText })
  inputText.value = ''
  scrollToBottom()

  // 2. 显示 AI 正在思考（加载动画）
  isThinking.value = true
  scrollToBottom()

  // 3. 真实调用后端 API，跨越次元壁！
  uni.request({
    url: 'http://192.168.68.178:8080/api/ai/chat',
    method: 'POST',
    data: { 
      message: userText 
    },
    success: (res) => {
      isThinking.value = false
      if (res.data && res.data.code === 200) {
        // 成功获取到 Coze 大模型的真实回复
        messageList.value.push({ role: 'ai', content: res.data.data })
      } else {
        // 后端报错处理
        messageList.value.push({ role: 'ai', content: '（服务器开小差了：' + (res.data.message || '未知错误') + '）' })
      }
      scrollToBottom()
    },
    fail: (err) => {
      isThinking.value = false
      messageList.value.push({ role: 'ai', content: '（信号丢失，请检查本地后端是否启动）' })
      scrollToBottom()
      console.error("AI接口报错:", err)
    }
  })
}


// 永远滚动到最新消息
const scrollToBottom = () => {
  nextTick(() => {
    if (isThinking.value) {
      scrollToView.value = 'thinking'
    } else {
      scrollToView.value = 'msg-' + (messageList.value.length - 1)
    }
  })
}
</script>

<style>
page {
  background-color: #0f121a;
  height: 100%;
}
.chat-container {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  position: relative;
}

/* 顶部导航 */
.nav-bar {
  position: absolute;
  top: 0;
  width: 100%;
  height: 160rpx;
  padding-top: 60rpx;
  display: flex;
  align-items: center;
  background: rgba(15, 18, 26, 0.9);
  backdrop-filter: blur(10px);
  z-index: 10;
  box-shadow: 0 4rpx 20rpx rgba(0,0,0,0.5);
}
.back-btn {
  width: 80rpx;
  height: 80rpx;
  display: flex;
  align-items: center;
  justify-content: center;
}
.back-icon {
  color: #fff;
  font-size: 60rpx;
  font-weight: 300;
}
.title-area {
  display: flex;
  flex-direction: column;
}
.nav-title {
  color: #d4af37;
  font-size: 32rpx;
  font-weight: bold;
}
.nav-subtitle {
  color: rgba(255,255,255,0.5);
  font-size: 20rpx;
  margin-top: 4rpx;
}

/* 聊天列表 */
.chat-list {
  flex: 1;
  width: 100%;
  height: 100%;
}
.message-row {
  display: flex;
  padding: 20rpx 30rpx;
  margin-bottom: 20rpx;
}
.ai-row {
  justify-content: flex-start;
}
.user-row {
  justify-content: flex-end;
}
.avatar {
  width: 80rpx;
  height: 80rpx;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 40rpx;
  flex-shrink: 0;
}
.ai-avatar {
  background: linear-gradient(135deg, #2c3e50, #1a252f);
  border: 1px solid #d4af37;
  margin-right: 20rpx;
}
.user-avatar {
  background: linear-gradient(135deg, #d4af37, #aa8529);
  margin-left: 20rpx;
}
.bubble {
  max-width: 65%;
  padding: 24rpx 30rpx;
  border-radius: 30rpx;
  font-size: 28rpx;
  line-height: 1.6;
}
.ai-bubble {
  background: rgba(255, 255, 255, 0.1);
  color: #fff;
  border-radius: 0 30rpx 30rpx 30rpx;
  border: 1px solid rgba(255,255,255,0.1);
}
.user-bubble {
  background: linear-gradient(135deg, #d4af37, #aa8529);
  color: #fff;
  border-radius: 30rpx 0 30rpx 30rpx;
  box-shadow: 0 4rpx 15rpx rgba(212, 175, 55, 0.3);
}

/* 底部输入区 */
.input-area {
  position: absolute;
  bottom: 0;
  width: 100%;
  background: rgba(15, 18, 26, 0.95);
  backdrop-filter: blur(10px);
  padding-bottom: constant(safe-area-inset-bottom);
  padding-bottom: env(safe-area-inset-bottom);
  border-top: 1px solid rgba(255,255,255,0.05);
}
.quick-prompts {
  white-space: nowrap;
  padding: 20rpx 30rpx;
}
.prompt-tag {
  display: inline-block;
  background: rgba(212, 175, 55, 0.15);
  color: #d4af37;
  font-size: 24rpx;
  padding: 12rpx 24rpx;
  border-radius: 30rpx;
  margin-right: 20rpx;
  border: 1px solid rgba(212, 175, 55, 0.3);
}
.input-box {
  display: flex;
  align-items: center;
  padding: 10rpx 30rpx 30rpx 30rpx;
}
.chat-input {
  flex: 1;
  height: 80rpx;
  background: rgba(255,255,255,0.1);
  border-radius: 40rpx;
  padding: 0 30rpx;
  color: #fff;
  font-size: 28rpx;
}
.send-btn {
  margin-left: 20rpx;
  width: 100rpx;
  height: 80rpx;
  border-radius: 40rpx;
  background: rgba(255,255,255,0.1);
  color: rgba(255,255,255,0.3);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 28rpx;
  transition: all 0.3s;
}
.btn-active {
  background: #d4af37;
  color: #000;
  font-weight: bold;
}

/* 思考动画 */
.thinking-bubble {
  display: flex;
  align-items: center;
}
.typing-dot {
  animation: typing 1.4s infinite ease-in-out both;
  font-size: 40rpx;
  line-height: 20rpx;
  margin: 0 4rpx;
}
.typing-dot:nth-child(1) { animation-delay: -0.32s; }
.typing-dot:nth-child(2) { animation-delay: -0.16s; }
@keyframes typing {
  0%, 80%, 100% { opacity: 0; }
  40% { opacity: 1; }
}
</style>