<template>
  <div class="hot-search-section">
    <h2 class="section-title">其他用户在搜</h2>

    <div class="cloud-container">
      <!-- 加载状态 -->
      <div v-if="loading" class="loading-state">
        <div class="spinner"></div>
        <span>加载中...</span>
      </div>

      <!-- 智能标签云 -->
      <template v-else>
        <div class="tag-cloud" :class="{ 'has-data': searches.length > 0 }">
          <button
            v-for="item in searches"
            :key="item.term"
            class="tag-item"
            :style="getTagStyle(item.score)"
            @click="onSearchClick(item.term)"
          >
            {{ item.term }}
          </button>
        </div>
      </template>

      <!-- 空状态 -->
      <div v-if="!loading && searches.length === 0" class="empty-state">
        <div class="empty-icon">🔍</div>
        <p>暂无热搜数据</p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';

interface Props {
  onSearch: (term: string) => void;
}

interface HotSearchItem {
  term: string;
  score: number;
  lastSearched: number;
  createdAt: number;
}

const props = defineProps<Props>();

// 状态
const loading = ref(false);
const searches = ref<HotSearchItem[]>([]);

// 获取热搜数据
async function fetchHotSearches() {
  loading.value = true;
  try {
    const response = await fetch('/api/hot-searches?limit=30');
    const data = await response.json();

    if (data.code === 0 && data.data?.hotSearches) {
      // 按分数排序，高分在前
      searches.value = data.data.hotSearches
        .sort((a: HotSearchItem, b: HotSearchItem) => b.score - a.score)
        .slice(0, 30);
    }
  } catch (error) {
    console.error('获取热搜失败:', error);
    // 失败时使用假数据
    searches.value = generateFallbackData();
  } finally {
    loading.value = false;
  }
}

// 生成降级假数据
function generateFallbackData(): HotSearchItem[] {
  const now = Date.now();
  const terms = [
    { term: '黑神话悟空', score: 100 },
    { term: '流浪地球3', score: 95 },
    { term: 'Photoshop 2024', score: 90 },
    { term: 'Python教程', score: 85 },
    { term: '雅思真题', score: 80 },
    { term: '周杰伦', score: 75 },
    { term: '原神', score: 70 },
    { term: 'Office 2021', score: 65 },
    { term: 'VS Code', score: 60 },
    { term: '无损音乐', score: 55 },
    { term: 'Blender', score: 50 },
    { term: '考研资料', score: 45 },
    { term: '剪映专业版', score: 40 },
    { term: 'React教程', score: 35 },
    { term: 'Steam游戏', score: 30 },
    { term: 'Excel技巧', score: 25 },
    { term: 'PPT模板', score: 20 },
    { term: '电子书', score: 15 },
    { term: '车载HIFI', score: 10 },
    { term: '4K电影', score: 9 },
    { term: 'PS5游戏', score: 8 },
    { term: '雅思资料', score: 7 },
    { term: 'Adobe全家桶', score: 6 },
    { term: 'Switch游戏', score: 5 },
    { term: 'AI工具', score: 4 },
    { term: 'Blender教程', score: 3 },
    { term: '周杰伦专辑', score: 2 },
    { term: 'VSCode插件', score: 2 },
    { term: '雅思听力', score: 1 },
    { term: 'FLAC音乐', score: 1 }
  ];

  return terms.map(t => ({
    term: t.term,
    score: t.score,
    lastSearched: now,
    createdAt: now
  }));
}

// 根据分数计算标签样式
function getTagStyle(score: number) {
  // 分数映射到字体大小（12px - 24px）
  const minScore = Math.min(...searches.value.map(s => s.score));
  const maxScore = Math.max(...searches.value.map(s => s.score));
  const normalized = (score - minScore) / (maxScore - minScore || 1);
  const fontSize = 12 + normalized * 12; // 12px - 24px

  // 分数映射到颜色
  const colors = [
    { threshold: 80, color: '#ef4444' },  // 红色 - 最热
    { threshold: 60, color: '#f59e0b' },  // 橙色 - 热门
    { threshold: 40, color: '#eab308' },  // 黄色 - 较热
    { threshold: 20, color: '#22c55e' },  // 绿色 - 普通
    { threshold: 0, color: '#3b82f6' }    // 蓝色 - 一般
  ];

  const color = colors.find(c => score >= c.threshold)?.color || '#6b7280';

  // 分数映射到粗细和透明度
  const fontWeight = score >= 70 ? 800 : score >= 40 ? 700 : 600;
  const opacity = 0.7 + normalized * 0.3; // 0.7 - 1.0

  return {
    fontSize: `${fontSize}px`,
    color: color,
    fontWeight: fontWeight,
    opacity: opacity,
    padding: `${6 + normalized * 2}px ${10 + normalized * 4}px`,
    margin: `${4 + (1 - normalized) * 2}px`
  };
}

// 点击搜索词
function onSearchClick(term: string) {
  props.onSearch(term);
}

// 页面加载时获取数据
onMounted(() => {
  fetchHotSearches();
});
</script>

<style scoped>
.hot-search-section {
  width: 100%;
}

.section-title {
  font-size: 20px;
  font-weight: 700;
  color: var(--text-primary);
  margin: 0 0 16px 0;
  display: flex;
  align-items: center;
  gap: 8px;
}

.section-title::before {
  content: '👥';
  font-size: 24px;
}

.cloud-container {
  width: 100%;
}

/* 标签云容器 */
.tag-cloud {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
  gap: 4px;
  padding: 20px;
  background: var(--bg-glass);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: var(--radius-lg);
  min-height: 200px;
  transition: all 0.3s ease;
}

.tag-cloud.has-data {
  justify-content: center;
  align-items: center;
}

/* 标签样式 */
.tag-item {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  background: var(--bg-primary);
  border: 1px solid var(--border-light);
  border-radius: var(--radius-sm);
  cursor: pointer;
  transition: all var(--transition-fast);
  white-space: nowrap;
  text-align: center;
  line-height: 1.2;
  user-select: none;
}

.tag-item:hover {
  transform: translateY(-2px) scale(1.05);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  filter: brightness(1.1);
  z-index: 10;
}

/* 加载状态 */
.loading-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 12px;
  padding: 60px 20px;
  color: var(--text-secondary);
  background: var(--bg-glass);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: var(--radius-lg);
}

.spinner {
  width: 32px;
  height: 32px;
  border: 3px solid rgba(99, 102, 241, 0.2);
  border-top-color: var(--primary);
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

/* 空状态 */
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 60px 20px;
  text-align: center;
  color: var(--text-secondary);
  background: var(--bg-glass);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: var(--radius-lg);
}

.empty-icon {
  font-size: 48px;
  opacity: 0.5;
}

/* 动画 */
@keyframes spin {
  to { transform: rotate(360deg); }
}

/* 移动端优化 */
@media (max-width: 640px) {
  .section-title {
    font-size: 18px;
  }

  .tag-cloud {
    padding: 16px;
    gap: 3px;
    min-height: 160px;
  }

  .loading-state,
  .empty-state {
    padding: 40px 16px;
  }
}

/* 深色模式 */
@media (prefers-color-scheme: dark) {
  .tag-cloud {
    background: rgba(15, 23, 42, 0.6);
    border-color: rgba(255, 255, 255, 0.08);
  }

  .loading-state,
  .empty-state {
    background: rgba(15, 23, 42, 0.6);
    border-color: rgba(255, 255, 255, 0.08);
  }

  .tag-item {
    background: rgba(30, 41, 59, 0.5);
    border-color: rgba(100, 116, 139, 0.3);
  }

  .tag-item:hover {
    background: rgba(15, 23, 42, 0.7);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
  }
}

/* 减少动画模式 */
@media (prefers-reduced-motion: reduce) {
  .tag-item,
  .spinner {
    animation: none;
    transition: none;
  }

  .tag-item:hover {
    transform: none;
  }
}
</style>
