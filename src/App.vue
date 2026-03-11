<template>
  <div class="app-container" :class="{ 'dark-mode': isDarkMode }">
    <!-- 顶部导航 -->
    <van-nav-bar 
      :title="currentPageTitle" 
      fixed 
      placeholder
      :border="false"
      class="nav-bar"
    >
      <template #left>
        <van-icon 
          v-if="currentView !== 'home'" 
          name="arrow-left" 
          size="20" 
          @click="goBack" 
        />
        <div v-else class="logo">
          <span class="logo-icon">🤖</span>
        </div>
      </template>
      <template #right>
        <van-icon name="wap-nav" size="20" @click="showMenu = true" />
      </template>
    </van-nav-bar>

    <!-- 主页面 - 首页 -->
    <div v-if="currentView === 'home'" class="page home-page">
      <!-- 用户欢迎区 -->
      <div class="welcome-section">
        <div class="user-info">
          <div class="user-avatar">👤</div>
          <div class="user-details">
            <div class="greeting">{{ getGreeting() }}，{{ userInfo.name }}</div>
            <div class="user-phone">{{ userInfo.phone }}</div>
          </div>
        </div>
        <div class="user-stats">
          <div class="stat-item">
            <div class="stat-value">{{ userInfo.balance }}</div>
            <div class="stat-label">话费余额</div>
          </div>
          <div class="stat-item">
            <div class="stat-value">{{ userInfo.dataLeft }}GB</div>
            <div class="stat-label">流量剩余</div>
          </div>
          <div class="stat-item">
            <div class="stat-value">{{ userInfo.voiceLeft }}分钟</div>
            <div class="stat-label">语音剩余</div>
          </div>
        </div>
      </div>

      <!-- 快捷服务 -->
      <div class="section">
        <div class="section-header">
          <span class="section-title">⚡ 快捷服务</span>
        </div>
        <div class="service-grid">
          <div 
            v-for="service in services" 
            :key="service.id"
            class="service-item"
            @click="handleService(service)"
          >
            <div class="service-icon" :style="{ background: service.color }">
              {{ service.icon }}
            </div>
            <div class="service-name">{{ service.name }}</div>
          </div>
        </div>
      </div>

      <!-- 热门问题 -->
      <div class="section">
        <div class="section-header">
          <span class="section-title">🔥 热门问题</span>
          <span class="section-more" @click="showAllQuestions">查看全部</span>
        </div>
        <div class="hot-questions">
          <div 
            v-for="(question, index) in hotQuestions" 
            :key="index"
            class="question-item"
            @click="askQuestion(question.text)"
          >
            <span class="question-icon">{{ question.icon }}</span>
            <span class="question-text">{{ question.text }}</span>
            <van-icon name="arrow" size="14" color="#999" />
          </div>
        </div>
      </div>

      <!-- 安全提醒 -->
      <div class="section">
        <div class="section-header">
          <span class="section-title">🛡️ 安全中心</span>
        </div>
        <div class="security-cards">
          <div class="security-card" @click="navigateTo('security')">
            <div class="security-icon">🚫</div>
            <div class="security-info">
              <div class="security-title">骚扰拦截</div>
              <div class="security-desc">已拦截 {{ securityStats.blockedCalls }} 个骚扰电话</div>
            </div>
            <van-icon name="arrow" size="16" color="#999" />
          </div>
          <div class="security-card" @click="navigateTo('antifraud')">
            <div class="security-icon">⚠️</div>
            <div class="security-info">
              <div class="security-title">诈骗防护</div>
              <div class="security-desc">识别 {{ securityStats.fraudDetected }} 次诈骗风险</div>
            </div>
            <van-icon name="arrow" size="16" color="#999" />
          </div>
        </div>
      </div>

      <!-- 开始对话按钮 -->
      <div class="start-chat-btn" @click="navigateTo('chat')">
        <span class="btn-icon">💬</span>
        <span class="btn-text">开始智能对话</span>
      </div>
    </div>

    <!-- 对话页面 -->
    <div v-if="currentView === 'chat'" class="page chat-page">
      <!-- 对话区域 -->
      <div class="chat-container" ref="chatContainer">
        <div class="message-list">
          <!-- 欢迎消息 -->
          <div class="welcome-card" v-if="messages.length === 1">
            <div class="welcome-avatar">🤖</div>
            <div class="welcome-content">
              <div class="welcome-title">您好！我是联通AI智能客服</div>
              <div class="welcome-desc">
                我可以帮您查询话费、办理业务、故障报修、防诈骗咨询等。
              </div>
            </div>
          </div>

          <!-- 消息列表 -->
          <div
            v-for="(msg, index) in messages"
            :key="index"
            :class="['message-item', msg.role]"
          >
            <div class="message-avatar" :class="msg.role">
              {{ msg.role === 'bot' ? '🤖' : '👤' }}
            </div>
            <div class="message-content">
              <!-- 意图识别流程 -->
              <div v-if="msg.intentFlow" class="intent-flow">
                <div class="flow-steps">
                  <div 
                    v-for="(step, sIndex) in msg.intentFlow" 
                    :key="sIndex"
                    :class="['flow-step', { active: step.active, done: step.done }]"
                  >
                    <div class="step-icon">{{ step.icon }}</div>
                    <div class="step-info">
                      <div class="step-label">{{ step.label }}</div>
                      <div class="step-detail">{{ step.detail }}</div>
                    </div>
                    <div v-if="step.time" class="step-time">{{ step.time }}</div>
                  </div>
                </div>
              </div>
              
              <!-- 普通文本消息 -->
              <div v-if="msg.content" class="message-bubble" :class="msg.role">
                <div v-html="formatMessage(msg.content)"></div>
              </div>

              <!-- 卡片消息 -->
              <div v-if="msg.card" class="message-card">
                <div class="card-header">
                  <span class="card-icon">{{ msg.card.icon }}</span>
                  <span class="card-title">{{ msg.card.title }}</span>
                </div>
                <div class="card-body">
                  <div v-for="(item, i) in msg.card.items" :key="i" class="card-item">
                    <span class="item-label">{{ item.label }}</span>
                    <span class="item-value" :class="item.highlight ? 'highlight' : ''">{{ item.value }}</span>
                  </div>
                </div>
                <div v-if="msg.card.actions" class="card-actions">
                  <van-button 
                    v-for="(action, i) in msg.card.actions" 
                    :key="i"
                    size="small"
                    :type="action.type || 'default'"
                    @click="handleCardAction(action)"
                  >
                    {{ action.text }}
                  </van-button>
                </div>
              </div>

              <!-- 列表消息 -->
              <div v-if="msg.list" class="message-list-card">
                <div v-for="(item, i) in msg.list" :key="i" class="list-item" @click="handleListClick(item)">
                  <div class="list-icon">{{ item.icon }}</div>
                  <div class="list-content">
                    <div class="list-title">{{ item.title }}</div>
                    <div class="list-desc">{{ item.desc }}</div>
                  </div>
                  <van-icon name="arrow" size="14" color="#999" />
                </div>
              </div>
              
              <div class="message-time">{{ msg.time }}</div>
            </div>
          </div>

          <!-- 加载中 -->
          <div v-if="loading" class="message-item bot">
            <div class="message-avatar bot">🤖</div>
            <div class="message-bubble bot typing">
              <span class="typing-dot"></span>
              <span class="typing-dot"></span>
              <span class="typing-dot"></span>
            </div>
          </div>
        </div>
      </div>

      <!-- 快捷入口 -->
      <div class="quick-actions" v-if="messages.length <= 2">
        <div 
          v-for="action in quickActions" 
          :key="action.text"
          class="quick-btn"
          @click="handleQuickAction(action)"
        >
          <span class="quick-icon">{{ action.icon }}</span>
          <span class="quick-text">{{ action.text }}</span>
        </div>
      </div>

      <!-- 输入区域 -->
      <div class="input-area">
        <div class="input-wrapper">
          <van-field
            v-model="inputText"
            placeholder="请输入您的问题..."
            type="textarea"
            rows="1"
            autosize
            :border="false"
            @keyup.enter="sendMessage"
          />
          <van-button 
            type="primary" 
            size="small" 
            @click="sendMessage"
            :disabled="!inputText.trim() || loading"
            class="send-btn"
          >
            发送
          </van-button>
        </div>
      </div>
    </div>

    <!-- 意图识别演示页面 -->
    <div v-if="currentView === 'intent-demo'" class="page intent-demo-page">
      <div class="demo-header">
        <div class="demo-title">🧠 意图识别流程演示</div>
        <div class="demo-desc">展示AI如何理解用户输入并做出响应</div>
      </div>

      <div class="demo-input-section">
        <van-field
          v-model="demoInput"
          placeholder="输入示例：查话费、网络坏了、这个号码是诈骗吗"
          :border="false"
          class="demo-input"
        />
        <van-button type="primary" @click="runIntentDemo" :loading="demoLoading">
          开始演示
        </van-button>
      </div>

      <!-- 流程可视化 -->
      <div class="flow-visualization">
        <div 
          v-for="(step, index) in intentSteps" 
          :key="index"
          :class="['flow-step-card', { active: step.active, done: step.done }]"
        >
          <div class="step-header">
            <div class="step-number">{{ index + 1 }}</div>
            <div class="step-title">{{ step.title }}</div>
          </div>
          <div class="step-content">
            <div class="step-desc">{{ step.desc }}</div>
            <div v-if="step.result" class="step-result">{{ step.result }}</div>
          </div>
          <div v-if="step.time" class="step-metrics">
            <van-tag type="primary">{{ step.time }}</van-tag>
          </div>
        </div>
      </div>

      <!-- 预设示例 -->
      <div class="demo-examples">
        <div class="examples-title">📝 示例输入</div>
        <div class="examples-list">
          <div 
            v-for="(example, i) in intentExamples" 
            :key="i"
            class="example-item"
            @click="demoInput = example.text"
          >
            <span class="example-icon">{{ example.icon }}</span>
            <span class="example-text">{{ example.text }}</span>
          </div>
        </div>
      </div>
    </div>

    <!-- 安全中心页面 -->
    <div v-if="currentView === 'security'" class="page security-page">
      <div class="security-header">
        <div class="security-badge">
          <span class="badge-icon">🛡️</span>
          <span class="badge-text">安全防护已开启</span>
        </div>
      </div>

      <div class="security-stats">
        <div class="stat-card">
          <div class="stat-icon">🚫</div>
          <div class="stat-info">
            <div class="stat-number">{{ securityStats.blockedCalls }}</div>
            <div class="stat-label">已拦截骚扰电话</div>
          </div>
        </div>
        <div class="stat-card">
          <div class="stat-icon">⚠️</div>
          <div class="stat-info">
            <div class="stat-number">{{ securityStats.fraudDetected }}</div>
            <div class="stat-label">识别诈骗风险</div>
          </div>
        </div>
        <div class="stat-card">
          <div class="stat-icon">📱</div>
          <div class="stat-info">
            <div class="stat-number">{{ securityStats.smsBlocked }}</div>
            <div class="stat-label">拦截垃圾短信</div>
          </div>
        </div>
      </div>

      <div class="security-features">
        <div class="feature-item" @click="navigateTo('antifraud')">
          <div class="feature-icon">🔍</div>
          <div class="feature-info">
            <div class="feature-title">号码识别</div>
            <div class="feature-desc">查询可疑号码，识别诈骗电话</div>
          </div>
          <van-switch v-model="securitySettings.callBlock" size="20" />
        </div>
        <div class="feature-item">
          <div class="feature-icon">📨</div>
          <div class="feature-info">
            <div class="feature-title">短信过滤</div>
            <div class="feature-desc">自动过滤垃圾短信和诈骗短信</div>
          </div>
          <van-switch v-model="securitySettings.smsBlock" size="20" />
        </div>
        <div class="feature-item">
          <div class="feature-icon">👨‍👩‍👧‍👦</div>
          <div class="feature-info">
            <div class="feature-title">亲情守护</div>
            <div class="feature-desc">保护家人通信安全</div>
          </div>
          <van-icon name="arrow" size="16" color="#999" />
        </div>
      </div>

      <!-- 号码查询 -->
      <div class="number-query">
        <div class="query-title">🔍 号码安全查询</div>
        <van-search
          v-model="queryNumber"
          placeholder="输入可疑号码进行查询"
          show-action
          @search="queryPhoneNumber"
        >
          <template #action>
            <van-button size="small" type="primary" @click="queryPhoneNumber">查询</van-button>
          </template>
        </van-search>
        
        <div v-if="queryResult" class="query-result" :class="queryResult.level">
          <div class="result-header">
            <span class="result-icon">{{ queryResult.icon }}</span>
            <span class="result-status">{{ queryResult.status }}</span>
          </div>
          <div class="result-details">
            <div class="detail-item">
              <span class="detail-label">号码</span>
              <span class="detail-value">{{ queryResult.number }}</span>
            </div>
            <div class="detail-item">
              <span class="detail-label">标记</span>
              <span class="detail-value">{{ queryResult.mark }}</span>
            </div>
            <div class="detail-item">
              <span class="detail-label">举报次数</span>
              <span class="detail-value">{{ queryResult.reports }}次</span>
            </div>
          </div>
          <div class="result-actions">
            <van-button size="small" type="danger" @click="blockNumber">一键拉黑</van-button>
            <van-button size="small" @click="reportNumber">举报号码</van-button>
          </div>
        </div>
      </div>
    </div>

    <!-- 侧边菜单 -->
    <van-popup 
      v-model:show="showMenu" 
      position="right" 
      :style="{ width: '70%', height: '100%' }"
      class="menu-popup"
    >
      <div class="menu-header">
        <div class="menu-avatar">👤</div>
        <div class="menu-user-info">
          <div class="menu-user-name">{{ userInfo.name }}</div>
          <div class="menu-user-phone">{{ userInfo.phone }}</div>
        </div>
      </div>
      <van-cell-group :border="false">
        <van-cell title="首页" icon="home-o" @click="navigateTo('home')" />
        <van-cell title="智能对话" icon="chat-o" @click="navigateTo('chat')" />
        <van-cell title="意图识别演示" icon="eye-o" @click="navigateTo('intent-demo')" />
        <van-cell title="安全中心" icon="shield-o" @click="navigateTo('security')" />
        <van-cell title="通话记录" icon="phone-o" is-link />
        <van-cell title="我的套餐" icon="card" is-link />
        <van-cell title="账单查询" icon="balance-list-o" is-link />
      </van-cell-group>
      <div class="menu-footer">
        <van-switch v-model="isDarkMode" size="22" />
        <span class="dark-mode-label">深色模式</span>
      </div>
    </van-popup>
  </div>
</template>

<script setup>
import { ref, computed, nextTick, onMounted } from 'vue'
import { showToast, showSuccessToast, showLoadingToast } from 'vant'

// ==================== 状态管理 ====================
const currentView = ref('home')
const showMenu = ref(false)
const isDarkMode = ref(false)
const loading = ref(false)
const demoLoading = ref(false)

// 用户信息
const userInfo = ref({
  name: '用户先生',
  phone: '138****8888',
  balance: '¥68.50',
  dataLeft: 12.5,
  voiceLeft: 450
})

// 安全统计
const securityStats = ref({
  blockedCalls: 156,
  fraudDetected: 23,
  smsBlocked: 89
})

// 安全设置
const securitySettings = ref({
  callBlock: true,
  smsBlock: true
})

// ==================== 页面导航 ====================
const currentPageTitle = computed(() => {
  const titles = {
    'home': '联通AI智能客服',
    'chat': '智能对话',
    'intent-demo': '意图识别演示',
    'security': '安全中心',
    'antifraud': '诈骗防护'
  }
  return titles[currentView.value] || '联通AI智能客服'
})

const navigateTo = (view) => {
  currentView.value = view
  showMenu.value = false
}

const goBack = () => {
  if (currentView.value === 'chat' && messages.value.length > 1) {
    currentView.value = 'home'
  } else {
    currentView.value = 'home'
  }
}

// ==================== 首页功能 ====================
const getGreeting = () => {
  const hour = new Date().getHours()
  if (hour < 6) return '夜深了'
  if (hour < 9) return '早上好'
  if (hour < 12) return '上午好'
  if (hour < 14) return '中午好'
  if (hour < 18) return '下午好'
  if (hour < 22) return '晚上好'
  return '夜深了'
}

// 快捷服务
const services = [
  { id: 1, name: '话费查询', icon: '💰', color: 'linear-gradient(135deg, #667eea, #764ba2)', action: 'balance' },
  { id: 2, name: '流量查询', icon: '📶', color: 'linear-gradient(135deg, #f093fb, #f5576c)', action: 'data' },
  { id: 3, name: '套餐办理', icon: '📱', color: 'linear-gradient(135deg, #4facfe, #00f2fe)', action: 'package' },
  { id: 4, name: '话费充值', icon: '💳', color: 'linear-gradient(135deg, #43e97b, #38f9d7)', action: 'recharge' },
  { id: 5, name: '故障报修', icon: '🔧', color: 'linear-gradient(135deg, #fa709a, #fee140)', action: 'repair' },
  { id: 6, name: '骚扰拦截', icon: '🚫', color: 'linear-gradient(135deg, #a8edea, #fed6e3)', action: 'block' },
  { id: 7, name: '诈骗防护', icon: '🛡️', color: 'linear-gradient(135deg, #ff9a9e, #fecfef)', action: 'antifraud' },
  { id: 8, name: '人工客服', icon: '👥', color: 'linear-gradient(135deg, #ffecd2, #fcb69f)', action: 'human' }
]

const handleService = (service) => {
  navigateTo('chat')
  const prompts = {
    'balance': '查询我的话费余额',
    'data': '查询我的流量使用情况',
    'package': '我想办理新套餐',
    'recharge': '如何进行话费充值',
    'repair': '我遇到了网络故障',
    'block': '查看骚扰拦截记录',
    'antifraud': '诈骗防护功能介绍',
    'human': '转接人工客服'
  }
  if (prompts[service.action]) {
    setTimeout(() => {
      inputText.value = prompts[service.action]
      sendMessage()
    }, 300)
  }
}

// 热门问题
const hotQuestions = [
  { icon: '💰', text: '如何查询话费余额？' },
  { icon: '📱', text: '如何办理5G套餐？' },
  { icon: '🔧', text: '网络故障如何报修？' },
  { icon: '🚫', text: '如何屏蔽骚扰电话？' }
]

const showAllQuestions = () => {
  showToast('查看全部问题')
}

const askQuestion = (text) => {
  navigateTo('chat')
  setTimeout(() => {
    inputText.value = text
    sendMessage()
  }, 300)
}

// ==================== 对话功能 ====================
const messages = ref([
  { role: 'bot', content: '您好！我是联通AI智能客服小通，很高兴为您服务！', time: '刚刚' }
])

const inputText = ref('')
const chatContainer = ref(null)

const quickActions = [
  { text: '话费查询', icon: '💰', prompt: '查询我的话费余额' },
  { text: '流量查询', icon: '📶', prompt: '查询我的流量使用情况' },
  { text: '套餐办理', icon: '📱', prompt: '我想办理新套餐' },
  { text: '故障报修', icon: '🔧', prompt: '我遇到了网络故障' }
]

// 模拟响应数据
const mockResponses = {
  '话费': {
    card: {
      icon: '💰',
      title: '话费信息',
      items: [
        { label: '当前余额', value: '¥68.50', highlight: true },
        { label: '本月已用', value: '¥31.50' },
        { label: '账户状态', value: '正常' }
      ],
      actions: [
        { text: '立即充值', type: 'primary', action: 'recharge' },
        { text: '查看账单', action: 'bill' }
      ]
    }
  },
  '流量': {
    card: {
      icon: '📶',
      title: '流量信息',
      items: [
        { label: '流量剩余', value: '12.5GB', highlight: true },
        { label: '本月已用', value: '7.5GB' },
        { label: '套餐总量', value: '20GB' }
      ],
      actions: [
        { text: '流量加油包', type: 'primary', action: 'data-pack' }
      ]
    }
  },
  '套餐': {
    list: [
      { icon: '📱', title: '5G畅享套餐 99元', desc: '30GB流量+500分钟通话', action: 'package-99' },
      { icon: '📱', title: '5G畅享套餐 129元', desc: '50GB流量+800分钟通话', action: 'package-129' },
      { icon: '📱', title: '5G畅享套餐 159元', desc: '80GB流量+1000分钟通话', action: 'package-159' }
    ]
  },
  '故障': {
    content: '很抱歉给您带来不便。请告诉我具体情况：\n\n1️⃣ 是完全无法连接，还是速度慢？\n2️⃣ 是WiFi问题还是有线网络？\n3️⃣ 大概从什么时候开始的？'
  },
  '骚扰': {
    card: {
      icon: '🚫',
      title: '骚扰拦截统计',
      items: [
        { label: '今日拦截', value: '12个' },
        { label: '本月拦截', value: '156个' },
        { label: '拦截状态', value: '已开启' }
      ],
      actions: [
        { text: '查看详情', action: 'block-detail' }
      ]
    }
  },
  '诈骗': {
    content: '🛡️ 诈骗防护功能介绍：\n\n• 智能识别诈骗电话，来电预警提醒\n• 风险号码自动拦截\n• 诈骗短信智能过滤\n• 一键举报可疑号码\n\n已为您识别23次诈骗风险，有效保护您的通信安全。'
  },
  '人工': {
    content: '正在为您转接人工客服...\n\n预计等待时间：2分钟\n\n您也可以继续与我对话，我会尽力为您解决问题。'
  }
}

const sendMessage = async () => {
  const text = inputText.value.trim()
  if (!text || loading.value) return

  // 添加用户消息
  const userTime = new Date().toLocaleTimeString('zh-CN', { hour: '2-digit', minute: '2-digit' })
  messages.value.push({ role: 'user', content: text, time: userTime })
  inputText.value = ''

  await nextTick()
  scrollToBottom()

  // 显示意图识别流程
  loading.value = true
  
  // 模拟意图识别过程
  const intentFlow = await simulateIntentFlow(text)
  messages.value.push({ 
    role: 'bot', 
    intentFlow,
    time: new Date().toLocaleTimeString('zh-CN', { hour: '2-digit', minute: '2-digit' })
  })
  
  await nextTick()
  scrollToBottom()
  
  // 生成响应
  await new Promise(r => setTimeout(r, 1500))
  loading.value = false

  // 根据关键词匹配响应
  let response = { role: 'bot', content: '我理解您的问题，让我为您查询...', time: new Date().toLocaleTimeString('zh-CN', { hour: '2-digit', minute: '2-digit' }) }
  
  for (const [key, value] of Object.entries(mockResponses)) {
    if (text.includes(key)) {
      response = { ...response, ...value }
      break
    }
  }

  messages.value.push(response)
  await nextTick()
  scrollToBottom()
}

// 模拟意图识别流程
const simulateIntentFlow = async (text) => {
  const steps = [
    { icon: '👤', label: '用户输入', detail: text.slice(0, 20) + (text.length > 20 ? '...' : ''), active: false, done: false },
    { icon: '🎯', label: '意图识别', detail: '分析中...', active: false, done: false, time: '' },
    { icon: '🤖', label: '智能体调度', detail: '等待调度', active: false, done: false, time: '' },
    { icon: '💬', label: '响应生成', detail: '生成中...', active: false, done: false, time: '' }
  ]

  // 模拟处理过程
  for (let i = 0; i < steps.length; i++) {
    steps[i].active = true
    await new Promise(r => setTimeout(r, 300))
    
    if (i === 1) {
      // 意图识别结果
      const intents = ['话费查询', '流量查询', '套餐办理', '故障报修', '骚扰拦截', '诈骗防护']
      steps[i].detail = intents[Math.floor(Math.random() * intents.length)]
      steps[i].time = '45ms'
    }
    if (i === 2) {
      steps[i].detail = '联通客服智能体'
      steps[i].time = '12ms'
    }
    if (i === 3) {
      steps[i].detail = '完成'
      steps[i].time = '156ms'
    }
    
    steps[i].done = true
    steps[i].active = false
  }

  return steps
}

const handleQuickAction = (action) => {
  inputText.value = action.prompt
  sendMessage()
}

const handleCardAction = (action) => {
  if (action.action === 'recharge') {
    showToast('跳转充值页面')
  } else if (action.action === 'bill') {
    showToast('跳转账单页面')
  }
}

const handleListClick = (item) => {
  showToast(`办理${item.title}`)
}

const formatMessage = (content) => {
  return content.replace(/\n/g, '<br>')
}

const scrollToBottom = () => {
  if (chatContainer.value) {
    chatContainer.value.scrollTop = chatContainer.value.scrollHeight
  }
}

// ==================== 意图识别演示 ====================
const demoInput = ref('')
const intentSteps = ref([
  { title: '输入预处理', desc: '分词、纠错、标准化', active: false, done: false },
  { title: '意图识别', desc: 'NLU模型分类', active: false, done: false },
  { title: '槽位填充', desc: '提取关键信息', active: false, done: false },
  { title: '智能体匹配', desc: '路由到对应服务', active: false, done: false },
  { title: '响应生成', desc: '生成回复内容', active: false, done: false }
])

const intentExamples = [
  { icon: '💰', text: '查一下我的话费' },
  { icon: '📶', text: '流量不够用了怎么办' },
  { icon: '🔧', text: '我家网络坏了' },
  { icon: '🚫', text: '这个号码是诈骗吗 13812345678' }
]

const runIntentDemo = async () => {
  if (!demoInput.value.trim()) {
    showToast('请输入演示内容')
    return
  }

  demoLoading.value = true
  
  // 重置步骤
  intentSteps.value.forEach(s => {
    s.active = false
    s.done = false
    s.result = null
    s.time = null
  })

  // 模拟处理
  for (let i = 0; i < intentSteps.value.length; i++) {
    intentSteps.value[i].active = true
    await new Promise(r => setTimeout(r, 500 + Math.random() * 500))
    
    // 添加结果
    const results = [
      `原文: "${demoInput.value}"`,
      '意图: 话费查询 (置信度: 0.95)',
      '槽位: [查询对象: 话费余额]',
      '智能体: 话费查询服务',
      '响应: 您的话费余额为68.50元'
    ]
    intentSteps.value[i].result = results[i]
    intentSteps.value[i].time = `${Math.floor(30 + Math.random() * 50)}ms`
    
    intentSteps.value[i].done = true
    intentSteps.value[i].active = false
  }

  demoLoading.value = false
}

// ==================== 安全中心 ====================
const queryNumber = ref('')
const queryResult = ref(null)

const queryPhoneNumber = () => {
  if (!queryNumber.value.trim()) {
    showToast('请输入号码')
    return
  }

  // 模拟查询结果
  const results = [
    { level: 'safe', icon: '✅', status: '正常号码', mark: '无标记', reports: 0 },
    { level: 'warning', icon: '⚠️', status: '疑似推销', mark: '推销电话', reports: 45 },
    { level: 'danger', icon: '🚫', status: '高风险号码', mark: '诈骗电话', reports: 234 }
  ]
  
  queryResult.value = {
    ...results[Math.floor(Math.random() * results.length)],
    number: queryNumber.value
  }
}

const blockNumber = () => {
  showSuccessToast('已加入黑名单')
}

const reportNumber = () => {
  showSuccessToast('举报成功')
}

// 初始化
onMounted(() => {
  // 加载主题设置
  const savedTheme = localStorage.getItem('darkMode')
  if (savedTheme) {
    isDarkMode.value = savedTheme === 'true'
  }
})
</script>

<style>
/* 全局样式 */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: #f5f7fa;
}

#app {
  max-width: 100vw;
  min-height: 100vh;
}

.app-container {
  min-height: 100vh;
  background: #f5f7fa;
}

.app-container.dark-mode {
  background: #1a1a2e;
  color: #eee;
}

/* 导航栏 */
.nav-bar {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%) !important;
}

.dark-mode .nav-bar {
  background: #16213e !important;
}

.logo-icon {
  font-size: 24px;
}

/* 页面基础 */
.page {
  padding-bottom: 20px;
}

/* ==================== 首页样式 ==================== */
.welcome-section {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 20px;
  color: white;
}

.user-info {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 20px;
}

.user-avatar {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  background: rgba(255,255,255,0.2);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
}

.greeting {
  font-size: 18px;
  font-weight: 600;
}

.user-phone {
  font-size: 13px;
  opacity: 0.8;
  margin-top: 2px;
}

.user-stats {
  display: flex;
  justify-content: space-around;
  background: rgba(255,255,255,0.1);
  border-radius: 12px;
  padding: 16px;
}

.stat-item {
  text-align: center;
}

.stat-value {
  font-size: 20px;
  font-weight: 700;
}

.stat-label {
  font-size: 12px;
  opacity: 0.8;
  margin-top: 4px;
}

/* 区块 */
.section {
  background: white;
  margin: 12px;
  border-radius: 12px;
  padding: 16px;
}

.dark-mode .section {
  background: #16213e;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.section-title {
  font-size: 15px;
  font-weight: 600;
}

.section-more {
  font-size: 12px;
  color: #667eea;
}

/* 服务网格 */
.service-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 12px;
}

.service-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  cursor: pointer;
}

.service-icon {
  width: 48px;
  height: 48px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
  transition: transform 0.2s;
}

.service-item:active .service-icon {
  transform: scale(0.9);
}

.service-name {
  font-size: 12px;
  color: #666;
}

.dark-mode .service-name {
  color: #aaa;
}

/* 热门问题 */
.hot-questions {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.question-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px;
  background: #f8f9fa;
  border-radius: 8px;
  cursor: pointer;
}

.dark-mode .question-item {
  background: #1a1a2e;
}

.question-item:active {
  background: #e9ecef;
}

.question-icon {
  font-size: 18px;
}

.question-text {
  flex: 1;
  font-size: 14px;
}

/* 安全卡片 */
.security-cards {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.security-card {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px;
  background: #f8f9fa;
  border-radius: 8px;
  cursor: pointer;
}

.dark-mode .security-card {
  background: #1a1a2e;
}

.security-icon {
  font-size: 24px;
}

.security-info {
  flex: 1;
}

.security-title {
  font-size: 14px;
  font-weight: 500;
}

.security-desc {
  font-size: 12px;
  color: #999;
  margin-top: 2px;
}

/* 开始对话按钮 */
.start-chat-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  margin: 16px;
  padding: 14px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border-radius: 24px;
  font-size: 16px;
  font-weight: 500;
  cursor: pointer;
  transition: transform 0.2s;
}

.start-chat-btn:active {
  transform: scale(0.98);
}

/* ==================== 对话页面样式 ==================== */
.chat-page {
  height: calc(100vh - 46px);
  display: flex;
  flex-direction: column;
  background: linear-gradient(180deg, #f0f4ff 0%, #f5f7fa 100%);
}

.dark-mode .chat-page {
  background: #1a1a2e;
}

.chat-container {
  flex: 1;
  overflow-y: auto;
  padding: 16px;
}

.message-list {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

/* 欢迎卡片 */
.welcome-card {
  display: flex;
  gap: 12px;
  padding: 16px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.05);
}

.dark-mode .welcome-card {
  background: #16213e;
}

.welcome-avatar {
  font-size: 32px;
}

.welcome-title {
  font-size: 16px;
  font-weight: 600;
  margin-bottom: 4px;
}

.welcome-desc {
  font-size: 13px;
  color: #666;
}

.dark-mode .welcome-desc {
  color: #aaa;
}

/* 消息项 */
.message-item {
  display: flex;
  gap: 10px;
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

.message-item.user {
  flex-direction: row-reverse;
}

.message-avatar {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 18px;
  flex-shrink: 0;
}

.message-avatar.bot {
  background: linear-gradient(135deg, #667eea, #764ba2);
}

.message-avatar.user {
  background: linear-gradient(135deg, #f093fb, #f5576c);
}

.message-content {
  max-width: 80%;
}

.message-bubble {
  padding: 12px 16px;
  border-radius: 16px;
  font-size: 14px;
  line-height: 1.6;
  word-break: break-word;
}

.message-bubble.bot {
  background: white;
  color: #333;
  border-bottom-left-radius: 4px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.05);
}

.dark-mode .message-bubble.bot {
  background: #16213e;
  color: #eee;
}

.message-bubble.user {
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
  border-bottom-right-radius: 4px;
}

.message-time {
  font-size: 11px;
  color: #999;
  margin-top: 4px;
  text-align: right;
}

.message-item.bot .message-time {
  text-align: left;
}

/* 打字动画 */
.message-bubble.typing {
  display: flex;
  gap: 4px;
  padding: 16px;
}

.typing-dot {
  width: 8px;
  height: 8px;
  background: #999;
  border-radius: 50%;
  animation: typing 1.4s infinite ease-in-out;
}

.typing-dot:nth-child(2) { animation-delay: 0.2s; }
.typing-dot:nth-child(3) { animation-delay: 0.4s; }

@keyframes typing {
  0%, 60%, 100% { transform: translateY(0); }
  30% { transform: translateY(-6px); }
}

/* 意图流程 */
.intent-flow {
  margin-bottom: 8px;
}

.flow-steps {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.flow-step {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 12px;
  background: #f8f9fa;
  border-radius: 8px;
  font-size: 12px;
  transition: all 0.3s;
}

.dark-mode .flow-step {
  background: #1a1a2e;
}

.flow-step.active {
  background: linear-gradient(135deg, rgba(102,126,234,0.2), rgba(118,75,162,0.2));
  border: 1px solid #667eea;
}

.flow-step.done {
  background: rgba(16,185,129,0.1);
}

.step-icon {
  font-size: 16px;
}

.step-info {
  flex: 1;
}

.step-label {
  font-weight: 500;
}

.step-detail {
  color: #666;
  font-size: 11px;
}

.dark-mode .step-detail {
  color: #aaa;
}

.step-time {
  color: #667eea;
  font-weight: 500;
}

/* 消息卡片 */
.message-card {
  background: white;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0,0,0,0.05);
}

.dark-mode .message-card {
  background: #16213e;
}

.card-header {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 12px 16px;
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
}

.card-icon {
  font-size: 20px;
}

.card-title {
  font-weight: 500;
}

.card-body {
  padding: 12px 16px;
}

.card-item {
  display: flex;
  justify-content: space-between;
  padding: 8px 0;
  border-bottom: 1px solid #f0f0f0;
}

.dark-mode .card-item {
  border-bottom-color: #2a2a4e;
}

.card-item:last-child {
  border-bottom: none;
}

.item-label {
  color: #666;
}

.dark-mode .item-label {
  color: #aaa;
}

.item-value {
  font-weight: 500;
}

.item-value.highlight {
  color: #667eea;
  font-size: 18px;
}

.card-actions {
  display: flex;
  gap: 8px;
  padding: 12px 16px;
  border-top: 1px solid #f0f0f0;
}

.dark-mode .card-actions {
  border-top-color: #2a2a4e;
}

/* 列表消息 */
.message-list-card {
  background: white;
  border-radius: 12px;
  overflow: hidden;
}

.dark-mode .message-list-card {
  background: #16213e;
}

.list-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 16px;
  border-bottom: 1px solid #f0f0f0;
  cursor: pointer;
}

.dark-mode .list-item {
  border-bottom-color: #2a2a4e;
}

.list-item:last-child {
  border-bottom: none;
}

.list-item:active {
  background: #f8f9fa;
}

.list-icon {
  font-size: 24px;
}

.list-content {
  flex: 1;
}

.list-title {
  font-weight: 500;
  margin-bottom: 2px;
}

.list-desc {
  font-size: 12px;
  color: #999;
}

/* 快捷入口 */
.quick-actions {
  display: flex;
  gap: 8px;
  padding: 12px 16px;
  overflow-x: auto;
  background: white;
}

.dark-mode .quick-actions {
  background: #16213e;
}

.quick-btn {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  padding: 10px 16px;
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
  border-radius: 12px;
  font-size: 12px;
  white-space: nowrap;
  cursor: pointer;
  transition: transform 0.2s;
}

.quick-btn:active {
  transform: scale(0.95);
}

.quick-icon {
  font-size: 18px;
}

/* 输入区域 */
.input-area {
  background: white;
  padding: 12px 16px;
  border-top: 1px solid #eee;
}

.dark-mode .input-area {
  background: #16213e;
  border-top-color: #2a2a4e;
}

.input-wrapper {
  display: flex;
  align-items: flex-end;
  gap: 8px;
}

.send-btn {
  flex-shrink: 0;
}

/* ==================== 意图演示页面 ==================== */
.intent-demo-page {
  padding: 16px;
}

.demo-header {
  text-align: center;
  margin-bottom: 24px;
}

.demo-title {
  font-size: 20px;
  font-weight: 700;
  margin-bottom: 8px;
}

.demo-desc {
  font-size: 14px;
  color: #666;
}

.dark-mode .demo-desc {
  color: #aaa;
}

.demo-input-section {
  display: flex;
  gap: 8px;
  margin-bottom: 24px;
}

.demo-input {
  flex: 1;
  background: white;
  border-radius: 8px;
}

.dark-mode .demo-input {
  background: #16213e;
}

/* 流程可视化 */
.flow-visualization {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-bottom: 24px;
}

.flow-step-card {
  background: white;
  border-radius: 12px;
  padding: 16px;
  transition: all 0.3s;
  border: 2px solid transparent;
}

.dark-mode .flow-step-card {
  background: #16213e;
}

.flow-step-card.active {
  border-color: #667eea;
  box-shadow: 0 4px 12px rgba(102,126,234,0.3);
}

.flow-step-card.done {
  border-color: #10b981;
}

.step-header {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 8px;
}

.step-number {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  font-weight: 600;
}

.step-title {
  font-weight: 600;
}

.step-content {
  padding-left: 36px;
}

.step-desc {
  font-size: 13px;
  color: #666;
}

.dark-mode .step-desc {
  color: #aaa;
}

.step-result {
  margin-top: 8px;
  padding: 8px 12px;
  background: #f0f4ff;
  border-radius: 6px;
  font-size: 13px;
  color: #667eea;
}

.dark-mode .step-result {
  background: #1a1a2e;
}

.step-metrics {
  margin-top: 8px;
  padding-left: 36px;
}

/* 示例 */
.demo-examples {
  background: white;
  border-radius: 12px;
  padding: 16px;
}

.dark-mode .demo-examples {
  background: #16213e;
}

.examples-title {
  font-weight: 600;
  margin-bottom: 12px;
}

.examples-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.example-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 12px;
  background: #f8f9fa;
  border-radius: 8px;
  cursor: pointer;
}

.dark-mode .example-item {
  background: #1a1a2e;
}

.example-item:active {
  background: #e9ecef;
}

.example-icon {
  font-size: 18px;
}

.example-text {
  font-size: 14px;
}

/* ==================== 安全中心页面 ==================== */
.security-page {
  padding: 16px;
}

.security-header {
  text-align: center;
  margin-bottom: 20px;
}

.security-badge {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 8px 16px;
  background: linear-gradient(135deg, #10b981, #34d399);
  color: white;
  border-radius: 20px;
  font-size: 14px;
  font-weight: 500;
}

.badge-icon {
  font-size: 18px;
}

.security-stats {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
  margin-bottom: 20px;
}

.stat-card {
  background: white;
  border-radius: 12px;
  padding: 16px;
  text-align: center;
}

.dark-mode .stat-card {
  background: #16213e;
}

.stat-icon {
  font-size: 28px;
  margin-bottom: 8px;
}

.stat-number {
  font-size: 24px;
  font-weight: 700;
  color: #667eea;
}

.stat-label {
  font-size: 12px;
  color: #666;
  margin-top: 4px;
}

.dark-mode .stat-label {
  color: #aaa;
}

/* 功能列表 */
.security-features {
  background: white;
  border-radius: 12px;
  margin-bottom: 20px;
}

.dark-mode .security-features {
  background: #16213e;
}

.feature-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 16px;
  border-bottom: 1px solid #f0f0f0;
  cursor: pointer;
}

.dark-mode .feature-item {
  border-bottom-color: #2a2a4e;
}

.feature-item:last-child {
  border-bottom: none;
}

.feature-icon {
  font-size: 24px;
}

.feature-info {
  flex: 1;
}

.feature-title {
  font-weight: 500;
  margin-bottom: 2px;
}

.feature-desc {
  font-size: 12px;
  color: #999;
}

/* 号码查询 */
.number-query {
  background: white;
  border-radius: 12px;
  padding: 16px;
}

.dark-mode .number-query {
  background: #16213e;
}

.query-title {
  font-weight: 600;
  margin-bottom: 12px;
}

.query-result {
  margin-top: 16px;
  padding: 16px;
  border-radius: 12px;
}

.query-result.safe {
  background: rgba(16,185,129,0.1);
  border: 1px solid #10b981;
}

.query-result.warning {
  background: rgba(245,158,11,0.1);
  border: 1px solid #f59e0b;
}

.query-result.danger {
  background: rgba(239,68,68,0.1);
  border: 1px solid #ef4444;
}

.result-header {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 12px;
}

.result-icon {
  font-size: 24px;
}

.result-status {
  font-weight: 600;
  font-size: 16px;
}

.query-result.safe .result-status { color: #10b981; }
.query-result.warning .result-status { color: #f59e0b; }
.query-result.danger .result-status { color: #ef4444; }

.result-details {
  margin-bottom: 12px;
}

.detail-item {
  display: flex;
  justify-content: space-between;
  padding: 8px 0;
  border-bottom: 1px dashed rgba(0,0,0,0.1);
}

.dark-mode .detail-item {
  border-bottom-color: rgba(255,255,255,0.1);
}

.detail-label {
  color: #666;
}

.dark-mode .detail-label {
  color: #aaa;
}

.result-actions {
  display: flex;
  gap: 8px;
  margin-top: 12px;
}

/* 菜单 */
.menu-popup {
  background: #f5f7fa;
}

.dark-mode .menu-popup {
  background: #1a1a2e;
}

.menu-header {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 24px 16px;
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
}

.menu-avatar {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  background: rgba(255,255,255,0.2);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
}

.menu-user-name {
  font-size: 16px;
  font-weight: 600;
}

.menu-user-phone {
  font-size: 13px;
  opacity: 0.8;
  margin-top: 2px;
}

.menu-footer {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 16px;
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  background: white;
  border-top: 1px solid #eee;
}

.dark-mode .menu-footer {
  background: #16213e;
  border-top-color: #2a2a4e;
}

.dark-mode-label {
  font-size: 14px;
}
</style>