<template>
  <section class="glass-card showcase-container">
    <div class="showcase-header">
      <div class="showcase-title-group">
        <h2 class="showcase-title">圣经箴言灵修演示案例库 (30 精选样例)</h2>
        <p class="showcase-subtitle">感悟圣言的恩典与智慧，点击“一键同款代祷”即可聆听箴言开示</p>
      </div>
      <div class="showcase-badge">圣言平安 · 免费体验</div>
    </div>

    <!-- 搜索与分类筛选 -->
    <div class="showcase-filter-bar">
      <div class="category-tabs">
        <button 
          v-for="cat in categories" 
          :key="cat"
          class="category-tab"
          :class="{ active: currentCategory === cat }"
          @click="currentCategory = cat"
        >
          {{ cat }}
        </button>
      </div>
      <div class="search-input-wrapper">
        <input 
          v-model="searchQuery"
          type="text"
          placeholder="搜索代祷事项、经卷章节或灵修流派..."
          class="search-input"
        />
      </div>
    </div>

    <!-- 样例列表格 Grid -->
    <div class="sample-grid">
      <div 
        v-for="sample in paginatedSamples" 
        :key="sample.id" 
        class="sample-card"
      >
        <div class="sample-card-header">
          <span class="topic-category-tag">{{ sample.category }}</span>
          <span class="verse-tag">{{ sample.verse }}</span>
        </div>
        <div class="sample-original">
          <span class="sample-label">代祷需求：</span>{{ sample.inquiry }}
        </div>
        <div class="sample-style-tag-line">
          <span class="sample-label">灵修流派：</span><span class="style-name">{{ sample.style }}</span>
        </div>
        <div class="sample-rewritten">
          <span class="sample-label">箴言默想：</span>{{ sample.summary }}
        </div>
        <div class="sample-card-footer">
          <button class="use-sample-btn" @click="$emit('use-sample', sample.inquiry)">
            一键同款代祷
          </button>
        </div>
      </div>
    </div>

    <!-- 空状态提示 -->
    <div v-if="filteredSamples.length === 0" class="empty-showcase">
      未找到匹配的灵修样例，请尝试切换分类或重置搜索关键词。
    </div>

    <!-- 分页组件 -->
    <div v-if="filteredSamples.length > pageSize" class="pagination-bar">
      <button 
        class="page-btn" 
        :disabled="currentPage === 1"
        @click="currentPage--"
      >
        上一页
      </button>
      <span class="page-info">第 {{ currentPage }} / {{ totalPages }} 页 (共 {{ filteredSamples.length }} 条)</span>
      <button 
        class="page-btn" 
        :disabled="currentPage === totalPages"
        @click="currentPage++"
      >
        下一页
      </button>
    </div>
  </section>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue';

defineEmits<{
  (e: 'use-sample', text: string): void;
}>();

const categories = ['全部', '职场迷茫', '平安祈祷', '信心试炼', '关系和好', '财务安息'];
const currentCategory = ref('全部');
const searchQuery = ref('');
const currentPage = ref(1);
const pageSize = 6;

interface BibleSample {
  id: number;
  category: string;
  verse: string;
  inquiry: string;
  style: string;
  summary: string;
}

// 精选 30 条圣经箴言与代祷案例
const raw30Samples: BibleSample[] = [
  {
    id: 1,
    category: '平安祈祷',
    verse: '诗篇 23:1-4',
    inquiry: '面对公司裁员风波，心里很没有安全感，求赐予我平静和前行的智慧。',
    style: '使徒箴言默想',
    summary: '耶和华是我的牧者，我必不致缺乏。他使我躺卧在青草地上，领我在可安歇的水边。行过死阴的幽谷也不怕遭害。'
  },
  {
    id: 2,
    category: '职场迷茫',
    verse: '腓立比书 4:6-7',
    inquiry: '面临创业与大厂高薪 Offer 的抉择，不知如何选择未来的道路。',
    style: '九巨擘神学论',
    summary: '马斯克与乔布斯视角：应当一无挂虑，把你的异象交托。第一性原理告诉我们，真正的呼召来自内心的真理。'
  },
  {
    id: 3,
    category: '信心试炼',
    verse: '哥林多后书 12:9',
    inquiry: '连续几次面试失败，陷入绝望与自我怀疑之中。',
    style: '先知当头警醒',
    summary: '我的恩典够你用的，因为我的能力是在人的软弱上显得完全。世俗的挫折不过是炼净你骄傲与虚妄的熔炉。'
  },
  {
    id: 4,
    category: '关系和好',
    verse: '哥林多前书 13:4-7',
    inquiry: '与家人因为观念差异发生严重争吵，如何修复彼此的关系？',
    style: '温情赞美与安息',
    summary: '爱是恒久忍耐，又有恩慈；爱是不嫉妒，爱是不自夸，不张狂。用温柔与倾听代替言语的锋芒。'
  },
  {
    id: 5,
    category: '财务安息',
    verse: '马太福音 6:26',
    inquiry: '房贷车贷压力巨大，每天都在为未来的生活费用焦灼。',
    style: '赛博代祷解压',
    summary: '系统格式化焦虑缓存：你们看那天上的飞鸟，也不种也不收，你们的天父尚且养活它，你们不比飞鸟贵重得多吗？'
  },
  {
    id: 6,
    category: '职场迷茫',
    verse: '箴言 3:5-6',
    inquiry: '刚步入管理岗位，感觉团队难以带好，缺乏威信。',
    style: '使徒箴言默想',
    summary: '你要专心仰赖耶和华，不可倚靠自己的聪明，在你一切所行的事上都要认定他，他必指引你的路。'
  },
  {
    id: 7,
    category: '平安祈祷',
    verse: '约翰福音 14:27',
    inquiry: '夜晚经常失眠焦虑，脑海里充斥着对未知的恐惧。',
    style: '温情赞美与安息',
    summary: '我留下平安给你们，我将我的平安赐给你们。你们心里不要忧愁，也不要胆怯。今夜且安心入睡。'
  },
  {
    id: 8,
    category: '信心试炼',
    verse: '罗马书 8:28',
    inquiry: '精心准备了一年的项目突然被取消，感觉付出付之东流。',
    style: '九巨擘神学论',
    summary: '柏拉图与秦始皇辩论：万事都互相效力，叫爱神的人得益处。基业的奠定非一日之功，暂时的曲折乃大局安排。'
  },
  {
    id: 9,
    category: '关系和好',
    verse: '以弗所书 4:32',
    inquiry: '被信任的好朋友误解甚至疏远，心里难过委屈。',
    style: '使徒箴言默想',
    summary: '只要以恩慈相待，存怜悯的心，彼此饶恕，正如神在基督里饶恕了你们一样。'
  },
  {
    id: 10,
    category: '财务安息',
    verse: '提摩太前书 6:6-8',
    inquiry: '看到同龄人买车买房投资赚钱，自己感到强烈的落差感。',
    style: '先知当头警醒',
    summary: '然而，敬虔加上知足的心便是大利了；因为我们没有带什么到世上来，也不能带什么去。'
  },
  {
    id: 11,
    category: '平安祈祷',
    verse: '诗篇 46:1',
    inquiry: '家人身体患病，心里十分揪心与担忧。',
    style: '温情赞美与安息',
    summary: '神是我们的避难所，是我们的力量，是我们在患难中随时的帮助。祈愿平安与医治临到。'
  },
  {
    id: 12,
    category: '职场迷茫',
    verse: '歌罗西书 3:23',
    inquiry: '工作枯燥重复，感觉找不到任何成就感与动力。',
    style: '使徒箴言默想',
    summary: '无论做什么，都要从心里做，像是给主做的，不是给人做的。在平凡的岗位上亦能展现不凡。'
  },
  {
    id: 13,
    category: '信心试炼',
    verse: '以赛亚书 40:31',
    inquiry: '长期处于高压状态，感到身心俱疲、灵力枯竭。',
    style: '赛博代祷解压',
    summary: '重启灵魂内核：但那等候耶和华的必重新得力。他们必如鹰展翅上腾；他们奔跑却不困倦，行走却不疲乏。'
  },
  {
    id: 14,
    category: '关系和好',
    verse: '雅各书 1:19',
    inquiry: '伴侣之间缺乏有效沟通，经常因为琐事冷战。',
    style: '使徒箴言默想',
    summary: '你们各人要快快的听，慢慢的说，慢慢的发怒。用耐心倾听打开彼此心门。'
  },
  {
    id: 15,
    category: '财务安息',
    verse: '希伯来书 13:5',
    inquiry: '创业资金吃紧，担心下个月发不出员工工资。',
    style: '九巨擘神学论',
    summary: '贝索斯视角：你们存心不可贪爱钱财，要以自己所有的为足。长期主义要求我们在危机中保持坚韧。'
  },
  {
    id: 16,
    category: '平安祈祷',
    verse: '诗篇 121:1-2',
    inquiry: '即将独自前往陌生的城市工作生活，内心忐忑。',
    style: '温情赞美与安息',
    summary: '我要向山举目；我的帮助从何而来？我的帮助从造天地的耶和华而来。他必保护你四围平安。'
  },
  {
    id: 17,
    category: '职场迷茫',
    verse: '箴言 16:3',
    inquiry: '想转行做 AI 行业，但不知从何入手搭建壁垒。',
    style: '赛博代祷解压',
    summary: '重构路线图：你所做的，要交托耶和华，你所筹算的，就必成立。专注核心能力升级。'
  },
  {
    id: 18,
    category: '信心试炼',
    verse: '马太福音 17:20',
    inquiry: '觉得自己的能力和资源极其微小，做不成大事。',
    style: '使徒箴言默想',
    summary: '你们若有信心像一粒芥菜种，就是对这座山说：‘你从这边挪到那边’，它也必挪去；并没有一件不能做的事了。'
  },
  {
    id: 19,
    category: '关系和好',
    verse: '箴言 15:1',
    inquiry: '职场中遭到同事的不善言语挑衅，如何应对？',
    style: '先知当头警醒',
    summary: '回答柔和，使怒消退；言语暴戾，触动怒气。不要落入世俗争竞的罗网。'
  },
  {
    id: 20,
    category: '财务安息',
    verse: '诗篇 37:5',
    inquiry: '理财投资出现亏损，心情沉重难以释怀。',
    style: '温情赞美与安息',
    summary: '当将你的道路交托耶和华，并倚靠他，他就必成全。物质得失皆有定数，保全内心的平安最贵。'
  },
  {
    id: 21,
    category: '平安祈祷',
    verse: '帖撒罗尼迦前书 5:16-18',
    inquiry: '生活遭遇诸多不顺，不知如何保持喜乐。',
    style: '使徒箴言默想',
    summary: '要常常喜乐，不住的祷告，凡事谢恩；因为这是神在基督耶稣里向你们所定的旨意。'
  },
  {
    id: 22,
    category: '职场迷茫',
    verse: '以弗所书 5:15-16',
    inquiry: '平时时间管理混乱，拖延症严重影响进度。',
    style: '先知当头警醒',
    summary: '你们要仔细行事，不要像愚昧人，当像智慧人。要爱惜光阴，因为现今的世代邪恶。'
  },
  {
    id: 23,
    category: '信心试炼',
    verse: '诗篇 34:18',
    inquiry: '经历了重大的失恋与情感创伤，感觉心碎。',
    style: '温情赞美与安息',
    summary: '耶和华靠近伤心的人，拯救灵性痛悔的人。愿恩典抚平每一道伤痕，赐你新生。'
  },
  {
    id: 24,
    category: '关系和好',
    verse: '马太福音 5:9',
    inquiry: '团队成员分歧巨大，如何做一名合格的调解者？',
    style: '九巨擘神学论',
    summary: '比尔盖茨与使徒视角：使人和睦的人有福了！因为他们必称为神的儿子。建立开放沟通与共识体系。'
  },
  {
    id: 25,
    category: '财务安息',
    verse: '路加福音 12:34',
    inquiry: '追求财富的同时，担心自己失去了初心与爱心。',
    style: '使徒箴言默想',
    summary: '因为，你们的财宝在哪里，你们的心也在哪里。保持清醒，让财物成为服侍社会的工具。'
  },
  {
    id: 26,
    category: '平安祈祷',
    verse: '以赛亚书 26:3',
    inquiry: '在充满不确定的时代，如何获得长久的内在定力？',
    style: '赛博代祷解压',
    summary: '坚心倚赖你的，你必保守他十分平安，因为他倚靠你。构建不可动摇的内在防火墙。'
  },
  {
    id: 27,
    category: '职场迷茫',
    verse: '箴言 22:29',
    inquiry: '想在专业领域做到顶尖，需要具备怎样的心志？',
    style: '九巨擘神学论',
    summary: '特斯拉与乔布斯视角：你看办事敏捷的人么？他必站在君王面前，不站在愚昧人面前。专注于极致的品质。'
  },
  {
    id: 28,
    category: '信心试炼',
    verse: '加拉太书 6:9',
    inquiry: '做公益与利他事情却遭到冷嘲热讽，要不要坚持？',
    style: '使徒箴言默想',
    summary: '我们行善，不可丧志；若不灰心，到了时候就要收成。坚持心中善念，必蒙应许。'
  },
  {
    id: 29,
    category: '关系和好',
    verse: '彼得前书 4:8',
    inquiry: '如何接纳伴侣身上无法改变的缺点？',
    style: '温情赞美与安息',
    summary: '最要紧的是彼此切实相爱，因为爱能遮掩许多的罪。用宽容与接纳成就彼此。'
  },
  {
    id: 30,
    category: '财务安息',
    verse: '申命记 8:18',
    inquiry: '事业取得一定成功后，如何防止自己走向骄傲？',
    style: '先知当头警醒',
    summary: '你要记念耶和华你的神，因为得货财的能力是他赐给你的。永怀谦卑感激之心。'
  }
];

const samples = ref<BibleSample[]>(raw30Samples);

const filteredSamples = computed(() => {
  return samples.value.filter(s => {
    const matchCat = currentCategory.value === '全部' || s.category === currentCategory.value;
    const matchQuery = !searchQuery.value.trim() || 
      s.inquiry.includes(searchQuery.value) || 
      s.verse.includes(searchQuery.value) || 
      s.style.includes(searchQuery.value) ||
      s.summary.includes(searchQuery.value);
    return matchCat && matchQuery;
  });
});

const totalPages = computed(() => Math.ceil(filteredSamples.value.length / pageSize) || 1);

const paginatedSamples = computed(() => {
  const start = (currentPage.value - 1) * pageSize;
  return filteredSamples.value.slice(start, start + pageSize);
});

watch([currentCategory, searchQuery], () => {
  currentPage.value = 1;
});
</script>
