<template>
  <div class="app-container">
    <!-- 顶部生成成功浮动 Toast -->
    <transition name="fade">
      <div v-if="showSuccessToast" class="top-success-toast">
        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
          <polyline points="20 6 9 17 4 12"></polyline>
        </svg>
        <span>圣经箴言代祷默想生成成功！</span>
      </div>
    </transition>

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

    <!-- 顶部 App Header (已移除未登录额度提示栏) -->
    <header>
      <h1>{{ appTitle }}</h1>
      <p>智能 AI 体验引擎 · 每日圣经箴言与灵修代祷</p>
    </header>

    <!-- 活跃动态与使用人数轮播 -->
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

      <!-- 本地历史记录视图 -->
      <div v-if="showHistory" class="history-view">
        <div class="history-header">
          <span>本地圣经箴言历史记录</span>
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
                <span class="h-city-tag">处境: {{ item.category }}</span>
                <span class="h-score-badge">灵性评分: {{ getAverageScore(item) }}</span>
              </div>

              <!-- 迷你指标条 (已移除 Emoji) -->
              <div class="h-mini-metrics">
                <div class="h-mini-item">信心: {{ item.userScores.faith }}/5</div>
                <div class="h-mini-item">平安: {{ item.userScores.peace }}/5</div>
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
            <label class="selector-label">请滑动评估您的灵性维度状态 (1-5分)</label>
            <div class="score-sliders">
              <div class="slider-group-item">
                <div class="slider-header">
                  <span class="slider-title">信心指数 (Faith)</span>
                  <span class="slider-value">{{ userScores.faith }} / 5</span>
                </div>
                <input type="range" min="1" max="5" step="1" v-model.number="userScores.faith" class="range-slider" />
              </div>

              <div class="slider-group-item">
                <div class="slider-header">
                  <span class="slider-title">平安指数 (Peace)</span>
                  <span class="slider-value">{{ userScores.peace }} / 5</span>
                </div>
                <input type="range" min="1" max="5" step="1" v-model.number="userScores.peace" class="range-slider" />
              </div>

              <div class="slider-group-item">
                <div class="slider-header">
                  <span class="slider-title">爱心慈悲 (Love)</span>
                  <span class="slider-value">{{ userScores.love }} / 5</span>
                </div>
                <input type="range" min="1" max="5" step="1" v-model.number="userScores.love" class="range-slider" />
              </div>

              <div class="slider-group-item">
                <div class="slider-header">
                  <span class="slider-title">智慧感悟 (Wisdom)</span>
                  <span class="slider-value">{{ userScores.wisdom }} / 5</span>
                </div>
                <input type="range" min="1" max="5" step="1" v-model.number="userScores.wisdom" class="range-slider" />
              </div>

              <div class="slider-group-item">
                <div class="slider-header">
                  <span class="slider-title">试炼困惑 (Trial)</span>
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
              rows="4"
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

          <!-- 异常提示 -->
          <div v-if="errorMsg" class="error-banner">
            {{ errorMsg }}
          </div>
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
                <path class="flame-inner" d="M80,35 C70,60 75,75 80,75 C85,75 90,60 80,35 Z" fill="#fbbf24" />
                <path class="flame-outer" d="M80,20 C60,55 70,80 80,80 C90,80 100,55 80,20 Z" fill="rgba(245, 158, 11, 0.6)" />
                <line x1="80" y1="80" x2="80" y2="90" stroke="#78350f" stroke-width="3" />
                <rect x="55" y="90" width="50" height="90" rx="6" fill="rgba(255, 255, 255, 0.08)" stroke="rgba(255, 255, 255, 0.15)" stroke-width="2" />
                <rect x="50" y="180" width="60" height="10" rx="3" fill="rgba(255, 255, 255, 0.12)" />
              </svg>
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
              <span class="merit-sub-tip">（点击圣烛鸣响圣钟加持）</span>
            </div>
          </div>

          <!-- 双轨灵性数据对比看板 -->
          <div v-if="aiScores" class="nomad-dashboard">
            <div class="dashboard-header">
              <span class="d-title">灵性数据对比看板</span>
              <span class="d-sub">自我认知 vs AI 判定</span>
            </div>

            <div class="nomad-metrics-list">
              <div class="nomad-metric-item">
                <div class="m-label-line">
                  <span class="m-name">信心指数 (Faith)</span>
                  <span class="m-comp">{{ userScores.faith }}/5 <span class="vs-divider">|</span> AI: {{ aiScores.faith }}/5</span>
                </div>
                <div class="m-bar-track">
                  <div class="m-bar-user" :style="{ width: (userScores.faith * 20) + '%' }"></div>
                  <div class="m-bar-ai" :style="{ width: (aiScores.faith * 20) + '%' }"></div>
                </div>
              </div>

              <div class="nomad-metric-item">
                <div class="m-label-line">
                  <span class="m-name">平安指数 (Peace)</span>
                  <span class="m-comp">{{ userScores.peace }}/5 <span class="vs-divider">|</span> AI: {{ aiScores.peace }}/5</span>
                </div>
                <div class="m-bar-track">
                  <div class="m-bar-user" :style="{ width: (userScores.peace * 20) + '%' }"></div>
                  <div class="m-bar-ai" :style="{ width: (aiScores.peace * 20) + '%' }"></div>
                </div>
              </div>

              <div class="nomad-metric-item">
                <div class="m-label-line">
                  <span class="m-name">爱心慈悲 (Love)</span>
                  <span class="m-comp">{{ userScores.love }}/5 <span class="vs-divider">|</span> AI: {{ aiScores.love }}/5</span>
                </div>
                <div class="m-bar-track">
                  <div class="m-bar-user" :style="{ width: (userScores.love * 20) + '%' }"></div>
                  <div class="m-bar-ai" :style="{ width: (aiScores.love * 20) + '%' }"></div>
                </div>
              </div>

              <div class="nomad-metric-item">
                <div class="m-label-line">
                  <span class="m-name">智慧感悟 (Wisdom)</span>
                  <span class="m-comp">{{ userScores.wisdom }}/5 <span class="vs-divider">|</span> AI: {{ aiScores.wisdom }}/5</span>
                </div>
                <div class="m-bar-track">
                  <div class="m-bar-user" :style="{ width: (userScores.wisdom * 20) + '%' }"></div>
                  <div class="m-bar-ai" :style="{ width: (aiScores.aiScores ? aiScores.wisdom : userScores.wisdom) * 20 + '%' }"></div>
                </div>
              </div>
            </div>
          </div>

          <div class="result-header">
            <div class="result-title-group">
              <span class="result-title">圣言与灵修代祷开示</span>
              <span class="success-badge">生成成功</span>
            </div>
            <div class="button-actions">
              <button class="icon-btn" @click="copyText">
                {{ copied ? '已复制' : '复制箴言' }}
              </button>
              <button class="icon-btn highlight" @click="showShareCard = true">
                生成分享卡片
              </button>
              <button class="icon-btn restart-btn" @click="resetForm">
                重新代祷
              </button>
            </div>
          </div>

          <!-- 加载中骨架屏 -->
          <div v-if="loading" class="skeleton">
            <div class="skeleton-line" style="width: 80%"></div>
            <div class="skeleton-line" style="width: 95%"></div>
            <div class="skeleton-line" style="width: 60%"></div>
            <div class="skeleton-line" style="width: 75%"></div>
          </div>

          <!-- 渲染回复 -->
          <div v-else-if="result" class="output-content">
            {{ displayResultText }}
          </div>
        </div>
      </div>
    </main>

    <!-- 演示案例区组件 (模块三：30 条圣经箴言精选案例展示) -->
    <DemoShowcase @use-sample="handleUseSample" />

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
          <p>我们非常重视您的隐私。您在本应用中输入的代祷需求与评分仅用于实时大模型灵修默想，我们不会在服务器端进行永久存储或记录。</p>
          <p>为了记录您的代祷历史与恩典计数，本应用会在您的浏览器本地（localStorage）保存相关数据。</p>
        </div>
        <button class="modal-btn" @click="showPrivacy = false">关闭</button>
      </div>
    </div>

    <!-- 服务条款弹窗 -->
    <div v-if="showTerms" class="modal-overlay" @click.self="showTerms = false">
      <div class="modal-content">
        <h3>Terms of Service</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>欢迎使用我们的 AI 圣经箴言微应用。本应用仅提供灵修默想与心理舒缓代祷，解析内容不作为任何医疗诊断或法律投资的替代依据。</p>
        </div>
        <button class="modal-btn" @click="showTerms = false">关闭</button>
      </div>
    </div>

    <!-- 联系我们弹窗 (自适应高度 + weixin.png & dingtalk.png 展示) -->
    <div v-if="showContact" class="modal-overlay" @click.self="showContact = false">
      <div class="modal-content contact-modal-content">
        <h3>Contact Us</h3>
        <div class="modal-text-content contact-card-body">
          <p>如果您在使用过程中遇到任何问题，或有合作意向，可以通过以下方式联系我们：</p>
          <div class="contact-qr-container">
            <div class="contact-qr-card">
              <img :src="weixinImg" alt="微信客服" class="contact-qr-img" />
              <span class="contact-qr-label">微信客服</span>
            </div>
            <div class="contact-qr-card">
              <img :src="dingtalkImg" alt="钉钉交流群" class="contact-qr-img" />
              <span class="contact-qr-label">钉钉交流群</span>
            </div>
          </div>
          <p class="contact-email">支持反馈邮箱: <span style="color: var(--primary-color);">us@wuxian.xyz</span></p>
        </div>
        <button class="modal-btn" @click="showContact = false">关闭</button>
      </div>
    </div>

    <!-- 裂变拦截弹窗 (模块四：裂变机制) -->
    <FissionModal 
      :visible="showFission" 
      :wechat-id="wechatId"
      @unlocked="handleUnlocked"
    />

    <!-- 分享卡片弹窗 (模块二扩展) -->
    <ShareCardModal
      :visible="showShareCard"
      :content="displayResultText"
      :wechat-id="wechatId"
      @close="showShareCard = false"
    />

    <!-- 微信 H5 分享引导浮层 -->
    <div v-if="showShareGuide" class="share-guide-overlay" @click="handleShareClose">
      <div class="share-guide-arrow">↗</div>
      <div class="share-guide-content">
        <p>点击右上角菜单 <strong>•••</strong></p>
        <p>选择 <strong>「分享到朋友圈」</strong></p>
        <p class="share-guide-sub">分享这款高效率的 AI 智能微应用</p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import UserTicker from './components/UserTicker.vue';
import FissionModal from './components/FissionModal.vue';
import DemoShowcase from './components/DemoShowcase.vue';
import ShareCardModal from './components/ShareCardModal.vue';
import appConfig from './config.json';
import weixinImg from '../asset/weixin.png';
import dingtalkImg from '../asset/dingtalk.png';

const appTitle = ref(appConfig.title || '网腾无限AI 圣经箴言');
const wechatId = ref(appConfig.wechatId || 'ai_wuxian_xyz');
const promptTopic = ref(appConfig.promptTopic || '');

const inquiryCategory = ref('平安祈祷');
const userInput = ref('');
const loading = ref(false);
const errorMsg = ref('');
const result = ref('');
const copied = ref(false);
const showSuccessToast = ref(false);
const showFission = ref(false);
const showPrivacy = ref(false);
const showTerms = ref(false);
const showContact = ref(false);
const showShareGuide = ref(false);
const showShareCard = ref(false);

const userScores = ref({
  faith: 3,
  peace: 3,
  love: 4,
  wisdom: 3,
  trial: 2
});

const aiScores = ref<{ faith: number; peace: number; love: number; wisdom: number; trial: number; } | null>(null);

const styleOptions = [
  { label: '使徒箴言默想 (经典经卷与虔诚应用)', value: '使徒箴言默想' },
  { label: '九巨擘神学论 (9大巨擘跨时空神学对话)', value: '九巨擘神学论' },
  { label: '赛博代祷解压 (科幻机能与重构灵魂防火墙)', value: '赛博代祷解压' },
  { label: '先知当头警醒 (严厉直白痛击世俗贪恋)', value: '先知当头警醒' },
  { label: '温情赞美与安息 (诗篇般舒缓抚平内心的伤痛)', value: '温情赞美与安息' }
];

const activeStyle = ref(styleOptions[0].value);

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

const playSanctuaryBellSound = () => {
  try {
    const AudioContext = window.AudioContext || (window as any).webkitAudioContext;
    if (!AudioContext) return;
    const ctx = new AudioContext();
    const now = ctx.currentTime;
    
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
  
  playSanctuaryBellSound();
  
  totalGrace.value += 1;
  localStorage.setItem('bible_total_grace', totalGrace.value.toString());
  
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
  return text.replace(/\[BIBLE_SCORES\][\s\S]*?\[\/BIBLE_SCORES\]/gi, '').trim();
};

const displayResultText = computed(() => {
  return cleanResponseText(result.value);
});

const cleanExcerpt = (text: string) => {
  const cleaned = cleanResponseText(text);
  return cleaned.length > 80 ? cleaned.slice(0, 80) + '...' : cleaned;
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

const triggerSuccessToast = () => {
  showSuccessToast.value = true;
  setTimeout(() => {
    showSuccessToast.value = false;
  }, 3000);
};

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
    const fullPrompt = `【处境代祷需求】：${inquiryCategory.value}。${userInput.value.trim() ? '细节困惑：' + userInput.value : ''}\n【自我灵性评分】：信心:${userScores.value.faith}, 平安:${userScores.value.peace}, 爱心:${userScores.value.love}, 智慧:${userScores.value.wisdom}, 试炼:${userScores.value.trial}\n【选定流派】：${activeStyle.value}\n${promptTopic.value}`;

    const response = await fetch(apiEndpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      credentials: 'include',
      body: JSON.stringify({
        taskType: 'text',
        prompt: fullPrompt,
        style: activeStyle.value
      })
    });

    const data = await response.json();
    if (data.error) {
      errorMsg.value = data.error;
    } else {
      result.value = data.result;
      aiScores.value = parseAIScores(data.result);
      addHistoryRecord();
      triggerSuccessToast();
      
      const currentUses = parseInt(localStorage.getItem('free_uses') || '0', 10);
      localStorage.setItem('free_uses', (currentUses + 1).toString());
    }
  } catch (err: any) {
    errorMsg.value = '请求接口失败，请检查网络或本地代理服务。';
  } finally {
    loading.value = false;
  }
};

const resetForm = () => {
  result.value = '';
  aiScores.value = null;
};

const handleUseSample = (sampleInquiry: string) => {
  userInput.value = sampleInquiry;
  showHistory.value = false;
  window.scrollTo({ top: 0, behavior: 'smooth' });
};

const handleUnlocked = () => {
  showFission.value = false;
  handleGenerate();
};

const copyText = async () => {
  try {
    await navigator.clipboard.writeText(displayResultText.value);
    copied.value = true;
    setTimeout(() => {
      copied.value = false;
    }, 2000);
  } catch (err) {
    errorMsg.value = '复制失败，请手动选择复制。';
  }
};
</script>
