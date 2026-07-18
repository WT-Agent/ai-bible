<template>
  <div class="app-container">
    <!-- 右上角常驻分享按钮 -->
    <button class="floating-share-btn" @click="showShareGuide = true">
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="share-icon">
        <circle cx="18" cy="5" r="3"></circle>
        <circle cx="6" cy="12" r="3"></circle>
        <circle cx="18" cy="19" r="3"></circle>
        <line x1="8.59" y1="13.51" x2="15.42" y2="17.49"></line>
        <line x1="15.41" y1="6.51" x2="8.59" y2="10.49"></line>
      </svg>
      <span>分享</span>
    </button>

    <header>
      <h1>{{ appTitle }}</h1>
      <p>智能 AI 体验引擎 · 每日 AI 圣经箴言与代祷</p>
    </header>

    <!-- 活跃动态 -->
    <UserTicker />

    <!-- 核心卡片 / 结果与历史记录展示 -->
    <main class="glass-card input-group">
      <div class="card-tabs">
        <button class="tab-btn" :class="{ active: !showHistory }" @click="showHistory = false">
          数字灵修
        </button>
        <button class="tab-btn" :class="{ active: showHistory }" @click="showHistory = true">
          历史记录 ({{ historyList.length }})
        </button>
      </div>

      <div v-if="showHistory" class="history-view">
        <div class="history-header">
          <span>本地圣经箴言历史</span>
          <button v-if="historyList.length > 0" class="clear-all-btn" @click="clearAllHistory">清空全部</button>
        </div>

        <div v-if="historyList.length === 0" class="empty-state">
          <p>暂无历史代祷记录</p>
        </div>

        <div v-else class="history-grid">
          <div v-for="item in historyList" :key="item.id" class="history-card">
            <div class="h-card-header">
              <span class="h-card-style">{{ item.styleLabel }}</span>
              <span class="h-card-time">{{ item.timestamp }}</span>
            </div>
            
            <div class="h-card-body">
              <div class="h-card-nomad-title">
                <span class="h-city-tag">✝️ {{ item.category }}</span>
                <span class="h-score-badge">🚀 灵性度: {{ getAverageScore(item) }}</span>
              </div>

              <!-- 迷你指标条 -->
              <div class="h-mini-metrics">
                <div class="h-mini-item">✝️ 信心: {{ item.userScores.faith }}/5</div>
                <div class="h-mini-item">🕊️ 平安: {{ item.userScores.peace }}/5</div>
              </div>

              <p class="h-card-excerpt"><strong>箴言开示：</strong>{{ cleanExcerpt(item.output) }}</p>
            </div>

            <div class="h-card-actions">
              <button class="h-action-btn load-btn" @click="selectHistoryItem(item)">
                加载详情
              </button>
              <button class="h-action-btn delete-btn" @click="deleteHistoryRecord(item.id)">
                删除
              </button>
            </div>
          </div>
        </div>
      </div>

      <div v-else>
        <!-- 主测评输入区域 -->
        <div v-if="!result" class="divination-setup">
          <div class="selector-group">
            <label class="selector-label">选择当前所面临的处境与代祷需求</label>
            <select v-model="inquiryCategory" class="style-select">
              <option value="职场迷茫">职场迷茫 (工作选择与事业瓶颈)</option>
              <option value="平安祈祷">平安祈祷 (消除内心恐惧与焦虑)</option>
              <option value="信心试炼">信心试炼 (面对困境时的绝望与软弱)</option>
              <option value="关系和好">关系和好 (家庭、婚姻与人际爱心)</option>
              <option value="财务安息">财务安息 (解除对未来的物质担忧)</option>
            </select>
          </div>

          <div class="selector-group">
            <label class="selector-label">请滑动评估您的灵性状态</label>
            <div class="score-sliders">
              <div class="slider-group-item">
                <div class="slider-header">
                  <span class="slider-title">✝️ 信心指数 (Faith)</span>
                  <span class="slider-value">{{ userScores.faith }} / 5</span>
                </div>
                <input type="range" min="1" max="5" step="1" v-model.number="userScores.faith" class="range-slider" />
              </div>

              <div class="slider-group-item">
                <div class="slider-header">
                  <span class="slider-title">🕊️ 平安指数 (Peace)</span>
                  <span class="slider-value">{{ userScores.peace }} / 5</span>
                </div>
                <input type="range" min="1" max="5" step="1" v-model.number="userScores.peace" class="range-slider" />
              </div>

              <div class="slider-group-item">
                <div class="slider-header">
                  <span class="slider-title">❤️ 爱心慈悲 (Love)</span>
                  <span class="slider-value">{{ userScores.love }} / 5</span>
                </div>
                <input type="range" min="1" max="5" step="1" v-model.number="userScores.love" class="range-slider" />
              </div>

              <div class="slider-group-item">
                <div class="slider-header">
                  <span class="slider-title">📖 智慧感悟 (Wisdom)</span>
                  <span class="slider-value">{{ userScores.wisdom }} / 5</span>
                </div>
                <input type="range" min="1" max="5" step="1" v-model.number="userScores.wisdom" class="range-slider" />
              </div>

              <div class="slider-group-item">
                <div class="slider-header">
                  <span class="slider-title">🌪️ 试炼困惑 (Trial)</span>
                  <span class="slider-value">{{ userScores.trial }} / 5</span>
                </div>
                <input type="range" min="1" max="5" step="1" v-model.number="userScores.trial" class="range-slider" />
              </div>
            </div>
          </div>

          <div class="selector-group">
            <label class="selector-label">倾诉您的代祷需求或困境细节 (选填)</label>
            <textarea 
              v-model="userInput" 
              placeholder="请输入您的具体祷告或难题，例如：面对公司裁员风波，心里很没有安全感，求赐予我平静和前行的智慧..."
            ></textarea>
          </div>

          <div class="selector-group">
            <label class="selector-label">选择灵修默想流派与视角</label>
            <select v-model="activeStyle" class="style-select">
              <option 
                v-for="style in styleOptions" 
                :key="style.value" 
                :value="style.value"
              >
                {{ style.label }}
              </option>
            </select>
          </div>

          <button 
            class="action-btn" 
            :disabled="loading" 
            @click="handleGenerate"
          >
            {{ loading ? '灵修默想中...' : '提交代祷，聆听箴言' }}
          </button>
        </div>

        <!-- 测评结果展现 -->
        <div v-else class="divination-result">
          <!-- 虚拟祈祷圣烛解压交互板块 -->
          <div class="candle-section">
            <div class="candle-canvas">
              <svg 
                class="candle-svg" 
                :class="{ flickering: isFlickering }" 
                @click="lightPrayerCandle" 
                viewBox="0 0 160 200"
              >
                <!-- 烛火焰 -->
                <path class="flame-inner" d="M80,35 C70,60 75,75 80,75 C85,75 90,60 80,35 Z" fill="#fbbf24" />
                <path class="flame-outer" d="M80,20 C60,55 70,80 80,80 C90,80 100,55 80,20 Z" fill="rgba(245, 158, 11, 0.6)" />
                <!-- 烛心 -->
                <line x1="80" y1="80" x2="80" y2="90" stroke="#78350f" stroke-width="3" />
                <!-- 烛身 -->
                <rect x="55" y="90" width="50" height="90" rx="6" fill="rgba(255, 255, 255, 0.08)" stroke="rgba(255, 255, 255, 0.15)" stroke-width="2" />
                <rect x="50" y="180" width="60" height="10" rx="3" fill="rgba(255, 255, 255, 0.12)" />
              </svg>
              <!-- 浮空恩典动效 -->
              <transition-group name="float-up">
                <span 
                  v-for="item in floatingItems" 
                  :key="item.id" 
                  class="floating-merit"
                  :style="{ transform: `translate(${item.x}px, ${item.y}px)` }"
                >
                  {{ item.text }}
                </span>
              </transition-group>
            </div>
            <div class="merit-counter-display">
              累计祷告恩典：<strong style="color: #fbbf24;">{{ totalGrace }}</strong>
              <p class="wood-fish-tip">点击上方祈祷圣烛敲响圣钟，恩典 +1</p>
            </div>
          </div>

          <!-- 数据对比看板 -->
          <div v-if="aiScores" class="comparison-dashboard">
            <h3 class="dashboard-title">📊 灵性数据对比 (您的打分 vs AI共识)</h3>
            <div class="comparison-grid">
              <div v-for="metric in metricsList" :key="metric.key" class="comparison-row">
                <div class="metric-info">
                  <span class="metric-label">{{ metric.icon }} {{ metric.label }}</span>
                  <span class="metric-scores-text">
                    您的打分: <strong style="color: var(--primary-color)">{{ userScores[metric.key] }}</strong> | 
                    AI共识: <strong style="color: var(--accent-color)">{{ aiScores[metric.key] }}</strong>
                  </span>
                </div>
                <div class="comparison-bars">
                  <!-- 用户评分条 -->
                  <div class="bar-container">
                    <span class="bar-label">自己</span>
                    <div class="bar-bg">
                      <div class="bar-fill user-fill" :style="{ width: userScores[metric.key] * 20 + '%' }"></div>
                    </div>
                  </div>
                  <!-- AI评分条 -->
                  <div class="bar-container">
                    <span class="bar-label">AI</span>
                    <div class="bar-bg">
                      <div class="bar-fill ai-fill" :style="{ width: aiScores[metric.key] * 20 + '%' }"></div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="result-header">
            <span class="result-title">📖 圣经箴言开示：{{ inquiryCategory }}</span>
            <div class="button-actions">
              <button class="icon-btn" @click="copyText">
                {{ copied ? '已复制经文' : '复制箴言' }}
              </button>
              <button class="icon-btn" @click="showShareGuide = true">
                分享朋友圈
              </button>
              <button class="icon-btn" @click="resetReview">
                重新代祷
              </button>
            </div>
          </div>

          <div class="ai-response-wrapper">
            <div class="output-content scroll-box" style="text-align: left;">{{ cleanResponseText(result) }}</div>
          </div>
        </div>

        <!-- 加载状态 -->
        <div v-if="loading" class="ai-loading">
          <div class="spinner"></div>
          <p>正在为您查考经卷，检索最相贴切的圣言，撰写祷告默想...</p>
        </div>

        <!-- 异常提示 -->
        <div v-if="errorMsg" style="color: var(--accent-color); font-size: 0.85rem; text-align: center; margin-top: 0.5rem;">
          {{ errorMsg }}
        </div>
      </div>
    </main>

    <!-- 底部隐私与服务条款链接 -->
    <footer class="footer-links">
      <button class="footer-link-btn" @click="showPrivacy = true">Privacy Policy</button>
      <button class="footer-link-btn" @click="showTerms = true">Terms of Service</button>
      <button class="footer-link-btn" @click="showContact = true">Contact Us</button>
    </footer>

    <!-- 隐私政策弹窗 -->
    <div v-if="showPrivacy" class="modal-overlay" @click.self="showPrivacy = false">
      <div class="modal-content">
        <h3>Privacy Policy</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>我们非常重视您的隐私。您在本应用中选择的代祷需求、滑块评分以及补充的困扰文字等数据，均仅用于实时大模型求问，我们不会在服务器端持久记录您的个人敏感信息。</p>
          <p>为了在您的浏览器本地保留“恩典计数”和您的“圣经箴言卡”历史，应用会使用浏览器的本地存储（localStorage）保存相应状态。</p>
        </div>
        <button class="modal-btn" @click="showPrivacy = false">关闭</button>
      </div>
    </div>

    <!-- 服务条款弹窗 -->
    <div v-if="showTerms" class="modal-overlay" @click.self="showTerms = false">
      <div class="modal-content">
        <h3>Terms of Service</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>欢迎使用我们的 AI 圣经箴言与代祷灵修服务。使用本应用代表您同意遵守当地有关人工智能生成内容的各项管理条例。</p>
          <p>本微应用给出的圣言默想、祷告词与恩典计数均为趣味与心灵安慰性互动设计，不构成任何硬性的宗教教义宣告、心理治疗或医疗法律指导。</p>
        </div>
        <button class="modal-btn" @click="showTerms = false">关闭</button>
      </div>
    </div>

    <!-- 联系我们弹窗 -->
    <div v-if="showContact" class="modal-overlay" @click.self="showContact = false">
      <div class="modal-content" style="max-width: 420px;">
        <h3>Contact Us</h3>
        <div class="modal-text-content">
          <p>如果您在使用过程中遇到任何问题，或有合作意向，可以通过微信或钉钉联系我们：</p>
          <div style="display: flex; gap: 1rem; justify-content: center; margin-top: 0.5rem; margin-bottom: 0.5rem; flex-wrap: wrap;">
            <div style="text-align: center;">
              <img :src="weixinImg" alt="微信二维码" style="width: 130px; height: 130px; border-radius: 8px; border: 1px solid rgba(255,255,255,0.1);" />
              <div style="font-size: 0.75rem; margin-top: 0.25rem; color: var(--text-secondary);">微信</div>
            </div>
            <div style="text-align: center;">
              <img :src="dingtalkImg" alt="钉钉二维码" style="width: 130px; height: 130px; border-radius: 8px; border: 1px solid rgba(255,255,255,0.1);" />
              <div style="font-size: 0.75rem; margin-top: 0.25rem; color: var(--text-secondary);">钉钉</div>
            </div>
          </div>
        </div>
        <button class="modal-btn" @click="showContact = false">关闭</button>
      </div>
    </div>

    <!-- 裂变拦截弹窗 -->
    <FissionModal 
      :visible="showFission" 
      :wechat-id="wechatId"
      @unlocked="handleUnlocked"
    />

    <!-- 分享引导浮层 -->
    <div v-if="showShareGuide" class="share-guide-overlay" @click="handleShareClose">
      <div class="share-guide-arrow">↗</div>
      <div class="share-guide-content">
        <p>点击右上角菜单 <strong>•••</strong></p>
        <p>选择 <strong>「分享到朋友圈」</strong></p>
        <p class="share-guide-sub">分享这份来自圣言的平安与亮光</p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import UserTicker from './components/UserTicker.vue';
import FissionModal from './components/FissionModal.vue';
import appConfig from './config.json';
import weixinImg from '../asset/weixin.png';
import dingtalkImg from '../asset/dingtalk.png';

// 读取动态配置
const appTitle = ref(appConfig.title || '网腾无限AI 圣经箴言');
const wechatId = ref(appConfig.wechatId || 'ai_wuxian_xyz');
const promptTopic = ref(appConfig.promptTopic || '');

const inquiryCategory = ref('职场迷茫');
const userInput = ref('');
const loading = ref(false);
const errorMsg = ref('');
const result = ref('');
const copied = ref(false);
const showFission = ref(false);
const showPrivacy = ref(false);
const showTerms = ref(false);
const showContact = ref(false);
const showShareGuide = ref(false);

const userScores = ref({
  faith: 3,
  peace: 3,
  love: 3,
  wisdom: 3,
  trial: 3
});

const aiScores = ref<{ faith: number; peace: number; love: number; wisdom: number; trial: number; } | null>(null);

const styleOptions = [
  { label: '使徒箴言默想', value: '使徒箴言流派：引用经典圣经原文经卷章节，进行极具灵性深度与虔诚真理的生活默想。' },
  { label: '九巨擘神学论', value: '九巨擘流派：模拟马斯克（热力学与第一性原理）、乔布斯（呼召与极简）、秦始皇（主权与基业）等的科学与哲学神学激辩。' },
  { label: '赛博代祷解压', value: '赛博代祷流派：用系统升级恩典 2.0、格式化罪咎与焦虑缓存、重构灵魂防火墙等科幻机能代祷。' },
  { label: '先知当头警醒', value: '先知警醒流派：严厉直白，如先知般一针见血戳穿世俗贪恋与骄傲，唤醒沉睡软弱的灵魂。' },
  { label: '温情赞美与安息', value: '温情安息流派：如诗篇般优雅舒缓，提供极具安慰感的祷告词与陪伴，抚平内心恐惧带来安息。' }
];

const activeStyle = ref(styleOptions[0].value);

const metricsList = [
  { key: 'faith', label: '信心指数 (Faith)', icon: '✝️' },
  { key: 'peace', label: '平安指数 (Peace)', icon: '🕊️' },
  { key: 'love', label: '爱心慈悲 (Love)', icon: '❤️' },
  { key: 'wisdom', label: '智慧感悟 (Wisdom)', icon: '📖' },
  { key: 'trial', label: '试炼困惑 (Trial)', icon: '🌪️' }
] as const;

interface FloatingItem {
  id: number;
  x: number;
  y: number;
  text: string;
}

const floatingItems = ref<FloatingItem[]>([]);
const isFlickering = ref(false);
const totalGrace = ref(0);
let floatId = 0;

interface HistoryItem {
  id: string;
  timestamp: string;
  category: string;
  input: string;
  styleLabel: string;
  userScores: { faith: number; peace: number; love: number; wisdom: number; trial: number; };
  aiScores: { faith: number; peace: number; love: number; wisdom: number; trial: number; } | null;
  output: string;
}

const historyList = ref<HistoryItem[]>([]);
const showHistory = ref(false);

// 纯前端利用 Web Audio API 动态合成 528Hz 空灵悠扬的圣钟声
const playSanctuaryBellSound = () => {
  try {
    const AudioContext = window.AudioContext || (window as any).webkitAudioContext;
    if (!AudioContext) return;
    const ctx = new AudioContext();
    const now = ctx.currentTime;
    
    // 528Hz 索尔法吉奥频率及三倍频泛音，构建教堂圣钟的空灵余音
    const freqs = [528, 1056, 1584];
    const gains = [0.5, 0.25, 0.1];
    
    freqs.forEach((freq, idx) => {
      const osc = ctx.createOscillator();
      const gainNode = ctx.createGain();
      
      osc.type = 'sine';
      osc.frequency.setValueAtTime(freq, now);
      
      gainNode.gain.setValueAtTime(gains[idx], now);
      gainNode.gain.exponentialRampToValueAtTime(0.0001, now + 1.8);
      
      osc.connect(gainNode);
      gainNode.connect(ctx.destination);
      
      osc.start(now);
      osc.stop(now + 2.0);
    });
  } catch (e) {
    console.error('AudioContext error:', e);
  }
};

const lightPrayerCandle = () => {
  if (isFlickering.value) return;
  isFlickering.value = true;
  
  // 触发圣钟发声
  playSanctuaryBellSound();
  
  // 累加恩典
  totalGrace.value += 1;
  localStorage.setItem('bible_total_grace', totalGrace.value.toString());
  
  // 生成漂浮文字
  const id = floatId++;
  const x = Math.floor(Math.random() * 40) - 20;
  const y = -45;
  
  floatingItems.value.push({ id, x, y, text: '恩典 +1' });
  
  setTimeout(() => {
    floatingItems.value = floatingItems.value.filter(item => item.id !== id);
  }, 1000);

  setTimeout(() => {
    isFlickering.value = false;
  }, 120);
};

const loadHistory = () => {
  try {
    const raw = localStorage.getItem('bible_history_records');
    historyList.value = raw ? JSON.parse(raw) : [];
    
    const rawGrace = localStorage.getItem('bible_total_grace');
    totalGrace.value = rawGrace ? parseInt(rawGrace, 10) : 0;
  } catch (e) {
    historyList.value = [];
  }
};

const saveHistory = () => {
  localStorage.setItem('bible_history_records', JSON.stringify(historyList.value));
};

const addHistoryRecord = () => {
  const matched = styleOptions.find(o => o.value === activeStyle.value);
  const styleLabel = matched ? matched.label : '灵修流派';

  const newItem: HistoryItem = {
    id: Date.now().toString(),
    timestamp: new Date().toLocaleString(),
    category: inquiryCategory.value,
    input: userInput.value,
    styleLabel,
    userScores: { ...userScores.value },
    aiScores: aiScores.value ? { ...aiScores.value } : null,
    output: result.value
  };
  historyList.value.unshift(newItem);
  saveHistory();
};

const deleteHistoryRecord = (id: string) => {
  historyList.value = historyList.value.filter(item => item.id !== id);
  saveHistory();
};

const clearAllHistory = () => {
  if (confirm('确认清空所有历史灵修代祷记录吗？此操作不可恢复。')) {
    historyList.value = [];
    saveHistory();
  }
};

const selectHistoryItem = (item: HistoryItem) => {
  inquiryCategory.value = item.category;
  userInput.value = item.input;
  userScores.value = { ...item.userScores };
  aiScores.value = item.aiScores ? { ...item.aiScores } : null;
  result.value = item.output;
  showHistory.value = false;
};

const getAverageScore = (item: HistoryItem) => {
  const s = item.aiScores || item.userScores;
  const avg = (s.faith + s.peace + s.love + s.wisdom + s.trial) / 5;
  return avg.toFixed(1);
};

const cleanExcerpt = (text: string) => {
  const cleaned = cleanResponseText(text);
  return cleaned.length > 80 ? cleaned.slice(0, 80) + '...' : cleaned;
};

const parseAIScores = (text: string) => {
  const match = text.match(/\[BIBLE_SCORES\](.*?)\[\/BIBLE_SCORES\]/);
  if (match) {
    const scoreStr = match[1].trim();
    const scores: Record<string, number> = {};
    scoreStr.split(',').forEach(item => {
      const [key, val] = item.split(':');
      if (key && val) {
        scores[key.trim()] = Math.min(5, Math.max(1, parseInt(val.trim(), 10) || 3));
      }
    });
    return {
      faith: scores.faith || 3,
      peace: scores.peace || 3,
      love: scores.love || 3,
      wisdom: scores.wisdom || 3,
      trial: scores.trial || 3
    };
  }
  return null;
};

const cleanResponseText = (text: string) => {
  return text.replace(/\[BIBLE_SCORES\].*?\[\/BIBLE_SCORES\]/g, '').trim();
};

onMounted(() => {
  loadHistory();
});

const handleShareClose = () => {
  showShareGuide.value = false;
  localStorage.setItem('shared_fission', 'true');
};

const isLimitReached = computed(() => {
  const uses = parseInt(localStorage.getItem('free_uses') || '0', 10);
  const shared = localStorage.getItem('shared_fission') === 'true';
  return uses >= 3 && !shared;
});

const apiEndpoint = import.meta.env.DEV
  ? '/api/local/generate'
  : (import.meta.env.VITE_API_ENDPOINT || 'https://api.wuxian.xyz/api/v1/generate');

const handleGenerate = async () => {
  if (isLimitReached.value) {
    showFission.value = true;
    return;
  }

  loading.value = true;
  errorMsg.value = '';
  result.value = '';
  aiScores.value = null;

  try {
    const response = await fetch(apiEndpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        taskType: 'text',
        prompt: `起卦主题：${promptTopic.value}。所面困境代祷需求：${inquiryCategory.value}。用户自我评分：信心 ${userScores.value.faith}分，平安 ${userScores.value.peace}分，爱心 ${userScores.value.love}分，智慧 ${userScores.value.wisdom}分，试炼 ${userScores.value.trial}分。用户补充经历困扰：${userInput.value}。流派倾向：${activeStyle.value}`,
        style: activeStyle.value
      })
    });

    const data = await response.json();
    if (data.error) {
      errorMsg.value = data.error;
    } else {
      result.value = data.result;
      
      const parsed = parseAIScores(data.result);
      if (parsed) {
        aiScores.value = parsed;
      } else {
        aiScores.value = { ...userScores.value };
      }

      // 自动存储历史
      addHistoryRecord();
      
      // 累加免费次数
      const currentUses = parseInt(localStorage.getItem('free_uses') || '0', 10);
      localStorage.setItem('free_uses', (currentUses + 1).toString());
    }
  } catch (err: any) {
    errorMsg.value = '请求接口失败，请检查网络或本地代理服务。';
  } finally {
    loading.value = false;
  }
};

const handleUnlocked = () => {
  showFission.value = false;
  handleGenerate();
};

const resetReview = () => {
  userInput.value = '';
  result.value = '';
  aiScores.value = null;
  errorMsg.value = '';
  userScores.value = { faith: 3, peace: 3, love: 3, wisdom: 3, trial: 3 };
};

const copyText = async () => {
  try {
    const cleanedText = cleanResponseText(result.value);
    await navigator.clipboard.writeText(cleanedText);
    copied.value = true;
    setTimeout(() => {
      copied.value = false;
    }, 2000);
  } catch (err) {
    errorMsg.value = '复制失败，请手动选择复制。';
  }
};
</script>

<style scoped>
/* 评分滑块 */
.score-sliders {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1.25rem;
  margin-bottom: 0.5rem;
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid rgba(255, 255, 255, 0.04);
  padding: 1.25rem;
  border-radius: 12px;
}

.slider-group-item {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  text-align: left;
}

.slider-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 0.8rem;
  color: var(--text-secondary);
}

.slider-title {
  font-weight: 500;
}

.slider-value {
  color: #fbbf24;
  font-weight: bold;
}

.range-slider {
  -webkit-appearance: none;
  width: 100%;
  height: 6px;
  border-radius: 3px;
  background: rgba(255, 255, 255, 0.1);
  outline: none;
  cursor: pointer;
  transition: background 0.3s;
}

.range-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 16px;
  height: 16px;
  border-radius: 50%;
  background: #fbbf24;
  cursor: pointer;
  box-shadow: 0 0 10px rgba(251, 191, 36, 0.5);
  transition: transform 0.1s ease;
}

.range-slider::-webkit-slider-thumb:hover {
  transform: scale(1.2);
}

/* 祈祷圣烛解压区域 */
.candle-section {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
  padding: 1.5rem;
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid rgba(255, 255, 255, 0.05);
  border-radius: 16px;
  margin-bottom: 1.5rem;
}

.candle-canvas {
  position: relative;
  width: 120px;
  height: 150px;
}

.candle-svg {
  width: 100%;
  height: 100%;
  cursor: pointer;
  transition: transform 0.1s ease-in-out;
}

.candle-svg:hover {
  filter: drop-shadow(0 0 12px rgba(251, 191, 36, 0.4));
}

.candle-svg.flickering {
  transform: scale(0.94);
}

.flame-inner, .flame-outer {
  animation: candlePulse 1.5s ease-in-out infinite alternate;
}

@keyframes candlePulse {
  0% { transform: scale(1); opacity: 0.9; }
  100% { transform: scale(1.08); opacity: 1; filter: drop-shadow(0 0 6px #fbbf24); }
}

.floating-merit {
  position: absolute;
  left: 50%;
  top: 40%;
  font-size: 0.95rem;
  font-weight: bold;
  color: #fbbf24;
  text-shadow: 0 0 10px rgba(251, 191, 36, 0.6);
  pointer-events: none;
  white-space: nowrap;
}

/* 浮空渐隐动画 */
.float-up-enter-active {
  animation: floatUpAnim 1s ease-out forwards;
}

@keyframes floatUpAnim {
  0% {
    transform: translate(-50%, -40px);
    opacity: 1;
  }
  100% {
    transform: translate(-50%, -100px);
    opacity: 0;
  }
}

.merit-counter-display {
  font-size: 0.95rem;
  color: var(--text-primary);
  text-align: center;
}

.merit-counter-display strong {
  font-size: 1.3rem;
  text-shadow: 0 0 15px rgba(251, 191, 36, 0.4);
}

.wood-fish-tip {
  font-size: 0.75rem;
  color: var(--text-secondary);
  margin: 0.25rem 0 0 0;
}

/* 对比看板 */
.comparison-dashboard {
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(255, 255, 255, 0.06);
  border-radius: 12px;
  padding: 1.25rem;
  margin-bottom: 1.5rem;
  text-align: left;
}

.dashboard-title {
  font-size: 0.95rem;
  margin-top: 0;
  margin-bottom: 1.25rem;
  color: var(--text-primary);
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
  padding-bottom: 0.5rem;
}

.comparison-grid {
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
}

.comparison-row {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.metric-info {
  display: flex;
  justify-content: space-between;
  font-size: 0.85rem;
  font-weight: 500;
}

.metric-label {
  color: var(--text-primary);
}

.metric-scores-text {
  color: var(--text-secondary);
}

.comparison-bars {
  display: flex;
  flex-direction: column;
  gap: 0.35rem;
}

.bar-container {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.bar-label {
  font-size: 0.7rem;
  color: var(--text-secondary);
  width: 25px;
}

.bar-bg {
  flex: 1;
  height: 8px;
  background: rgba(255, 255, 255, 0.08);
  border-radius: 4px;
  overflow: hidden;
}

.bar-fill {
  height: 100%;
  border-radius: 4px;
  transition: width 0.8s cubic-bezier(0.4, 0, 0.2, 1);
}

.user-fill {
  background: linear-gradient(90deg, #f59e0b 0%, #fbbf24 100%);
  box-shadow: 0 0 8px rgba(245, 158, 11, 0.4);
}

.ai-fill {
  background: linear-gradient(90deg, #06b6d4 0%, #3b82f6 100%);
  box-shadow: 0 0 8px rgba(6, 182, 212, 0.4);
}

/* 历史卡片定制：Nomad Style */
.h-card-nomad-title {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.5rem;
}

.h-city-tag {
  font-size: 1.05rem;
  font-weight: bold;
  color: var(--text-primary);
}

.h-score-badge {
  background: rgba(245, 158, 11, 0.15);
  color: #f59e0b;
  padding: 0.2rem 0.5rem;
  border-radius: 6px;
  font-size: 0.75rem;
  font-weight: bold;
}

.h-mini-metrics {
  display: flex;
  gap: 0.75rem;
  margin-bottom: 0.5rem;
}

.h-mini-item {
  font-size: 0.75rem;
  color: var(--text-secondary);
  background: rgba(255, 255, 255, 0.04);
  padding: 0.15rem 0.4rem;
  border-radius: 4px;
  border: 1px solid rgba(255, 255, 255, 0.04);
}

/* 分享样式与浮层 */
.share-guide-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(17, 14, 36, 0.9);
  z-index: 1000;
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  padding: 2rem;
  box-sizing: border-box;
  color: #fff;
  cursor: pointer;
}

.share-guide-arrow {
  font-size: 3rem;
  color: var(--primary-color);
  animation: bounce 1s infinite alternate;
  margin-right: 1.5rem;
}

.share-guide-content {
  text-align: center;
  width: 100%;
  margin-top: 2rem;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.share-guide-content p {
  font-size: 1.2rem;
  margin: 0;
}

.share-guide-sub {
  font-size: 0.9rem;
  color: var(--text-secondary);
  margin-top: 1rem !important;
}

/* 右上角常驻分享按钮 */
.floating-share-btn {
  position: fixed;
  top: 1rem;
  right: 1rem;
  z-index: 99;
  display: flex;
  align-items: center;
  gap: 0.35rem;
  padding: 0.5rem 0.8rem;
  background: rgba(255, 255, 255, 0.08);
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 20px;
  color: var(--text-primary);
  font-size: 0.8rem;
  font-weight: 500;
  cursor: pointer;
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  transition: all 0.2s ease;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.floating-share-btn:hover {
  background: rgba(255, 255, 255, 0.15);
  border-color: var(--primary-color);
  box-shadow: 0 4px 16px rgba(168, 85, 247, 0.2);
}

.share-icon {
  width: 14px;
  height: 14px;
}

/* 历史记录选项卡 */
.card-tabs {
  display: flex;
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
  margin-bottom: 1.25rem;
  gap: 0.5rem;
}

.tab-btn {
  background: none;
  border: none;
  padding: 0.5rem 1rem;
  color: var(--text-secondary);
  font-size: 0.9rem;
  font-weight: bold;
  cursor: pointer;
  border-bottom: 2px solid transparent;
  transition: all 0.2s ease;
}

.tab-btn:hover {
  color: var(--text-primary);
}

.tab-btn.active {
  color: var(--primary-color);
  border-bottom-color: var(--primary-color);
}

/* 历史卡片网格 */
.history-view {
  text-align: left;
}

.history-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  font-size: 0.85rem;
  color: var(--text-secondary);
}

.clear-all-btn {
  background: none;
  border: none;
  color: var(--accent-color);
  font-size: 0.8rem;
  cursor: pointer;
  transition: opacity 0.2s ease;
}

.clear-all-btn:hover {
  opacity: 0.8;
}

.empty-state {
  text-align: center;
  padding: 3rem 1rem;
  color: var(--text-secondary);
  font-size: 0.9rem;
}

.history-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
  gap: 1rem;
}

.history-card {
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid rgba(255, 255, 255, 0.05);
  border-radius: 10px;
  padding: 1rem;
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
  transition: all 0.3s ease;
}

.history-card:hover {
  background: rgba(255, 255, 255, 0.04);
  border-color: rgba(168, 85, 247, 0.2);
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
}

.h-card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 0.75rem;
}

.h-card-style {
  background: rgba(168, 85, 247, 0.15);
  color: var(--primary-color);
  padding: 0.2rem 0.5rem;
  border-radius: 4px;
  font-weight: bold;
}

.h-card-time {
  color: var(--text-secondary);
}

.h-card-body {
  flex: 1;
  font-size: 0.85rem;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.h-card-excerpt {
  color: var(--text-primary);
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  margin: 0;
  white-space: pre-wrap;
}

.h-card-actions {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-top: 1px solid rgba(255, 255, 255, 0.05);
  padding-top: 0.75rem;
}

.h-action-btn {
  background: none;
  border: none;
  font-size: 0.8rem;
  font-weight: bold;
  cursor: pointer;
  transition: color 0.2s ease;
}

.load-btn {
  color: var(--primary-color);
}

.load-btn:hover {
  color: var(--primary-hover);
}

.delete-btn {
  color: var(--accent-color);
}

.delete-btn:hover {
  color: #ef4444;
}

.scroll-box {
  max-height: 320px;
  overflow-y: auto;
  padding-right: 0.5rem;
}
</style>
