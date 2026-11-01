<script setup lang="ts">
import { computed, ref } from 'vue'
import ZodiacPage from './components/ZodiacPage.vue'
import CompatibilityPage from './components/CompatibilityPage.vue'
import TestPage from './components/TestPage.vue'
import BirthChartPage from './components/BirthChartPage.vue'

type PanelType = 'zodiac' | 'compatibility' | 'test' | 'chart' | null

const selectedZodiac = ref('天蝎座')
const activePanel = ref<PanelType>(null)

const zodiacs = [
  { name: '白羊座', date: '3.21—4.19', symbol: '♈', element: '火象' },
  { name: '金牛座', date: '4.20—5.20', symbol: '♉', element: '土象' },
  { name: '双子座', date: '5.21—6.21', symbol: '♊', element: '风象' },
  { name: '巨蟹座', date: '6.22—7.22', symbol: '♋', element: '水象' },
  { name: '狮子座', date: '7.23—8.22', symbol: '♌', element: '火象' },
  { name: '处女座', date: '8.23—9.22', symbol: '♍', element: '土象' },
  { name: '天秤座', date: '9.23—10.23', symbol: '♎', element: '风象' },
  { name: '天蝎座', date: '10.24—11.22', symbol: '♏', element: '水象' },
  { name: '射手座', date: '11.23—12.21', symbol: '♐', element: '火象' },
  { name: '摩羯座', date: '12.22—1.19', symbol: '♑', element: '土象' },
  { name: '水瓶座', date: '1.20—2.18', symbol: '♒', element: '风象' },
  { name: '双鱼座', date: '2.19—3.20', symbol: '♓', element: '水象' },
]

const currentZodiac = computed(
  () => zodiacs.find(item => item.name === selectedZodiac.value) ?? zodiacs[7],
)

const fortuneScores = [88, 82, 91, 79, 94, 85, 89, 92, 86, 80, 90, 87]

const fortune = computed(() => {
  const index = zodiacs.findIndex(item => item.name === selectedZodiac.value)
  return fortuneScores[index] ?? 92
})

const panelTitle = computed(() => {
  const titles: Record<Exclude<PanelType, null>, string> = {
    zodiac: '十二星座图鉴',
    compatibility: '星座缘分配对',
    test: '探索内在自我',
    chart: '个人本命星盘',
  }
  return activePanel.value ? titles[activePanel.value] : ''
})

const today = computed(() => {
  return new Intl.DateTimeFormat('zh-CN', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
    weekday: 'long',
  }).format(new Date())
})

const features = [
  {
    id: 'zodiac' as const,
    symbol: '✧',
    name: '十二星座',
    en: 'ZODIAC SIGNS',
    desc: '探索十二种星座性格',
  },
  {
    id: 'compatibility' as const,
    symbol: '♡',
    name: '星座配对',
    en: 'LOVE COMPATIBILITY',
    desc: '探索两个人的相处密码',
  },
  {
    id: 'test' as const,
    symbol: '◇',
    name: '趣味测试',
    en: 'PERSONALITY TEST',
    desc: '发现内心深处的自己',
  },
  {
    id: 'chart' as const,
    symbol: '☾',
    name: '个人星盘',
    en: 'BIRTH CHART',
    desc: '绘制属于你的本命星图',
  },
]

/*
 * 十二星座星图。
 * 坐标为星盘内部的艺术化布局，不代表真实天体测量坐标。
 * 每个星座拥有独立星点和连接关系。
 */
type Star = { x: number; y: number; size?: number }
type StarLine = [number, number]

const constellationMaps: Record<string, { stars: Star[]; lines: StarLine[] }> = {
  白羊座: {
    stars: [
      { x: 22, y: 28, size: 1.3 },
      { x: 38, y: 20, size: 1.8 },
      { x: 55, y: 30, size: 1.5 },
      { x: 72, y: 42, size: 2.2 },
      { x: 57, y: 55, size: 1.2 },
      { x: 38, y: 51, size: 1.5 },
    ],
    lines: [[0, 1], [1, 2], [2, 3], [3, 4], [4, 5]],
  },
  金牛座: {
    stars: [
      { x: 20, y: 43, size: 1.4 },
      { x: 34, y: 28, size: 1.8 },
      { x: 50, y: 20, size: 2.3 },
      { x: 65, y: 32, size: 1.5 },
      { x: 78, y: 48, size: 1.8 },
      { x: 61, y: 58, size: 1.3 },
      { x: 43, y: 63, size: 1.5 },
      { x: 34, y: 45, size: 1.2 },
    ],
    lines: [[0, 1], [1, 2], [2, 3], [3, 4], [1, 7], [7, 6], [6, 5], [5, 4]],
  },
  双子座: {
    stars: [
      { x: 30, y: 18, size: 1.8 },
      { x: 28, y: 37, size: 1.4 },
      { x: 35, y: 58, size: 1.8 },
      { x: 48, y: 76, size: 1.3 },
      { x: 65, y: 20, size: 1.8 },
      { x: 64, y: 39, size: 1.4 },
      { x: 59, y: 59, size: 1.8 },
      { x: 48, y: 76, size: 1.3 },
      { x: 46, y: 43, size: 1.2 },
    ],
    lines: [[0, 1], [1, 2], [2, 3], [4, 5], [5, 6], [6, 7], [1, 8], [8, 5]],
  },
  巨蟹座: {
    stars: [
      { x: 20, y: 30, size: 1.5 },
      { x: 39, y: 22, size: 1.8 },
      { x: 57, y: 32, size: 1.4 },
      { x: 76, y: 25, size: 1.8 },
      { x: 67, y: 52, size: 1.5 },
      { x: 48, y: 67, size: 2 },
      { x: 29, y: 58, size: 1.4 },
    ],
    lines: [[0, 1], [1, 2], [2, 3], [2, 4], [4, 5], [5, 6], [6, 0]],
  },
  狮子座: {
    stars: [
      { x: 22, y: 27, size: 1.5 },
      { x: 39, y: 18, size: 2 },
      { x: 53, y: 30, size: 1.4 },
      { x: 45, y: 46, size: 1.8 },
      { x: 27, y: 51, size: 1.4 },
      { x: 63, y: 47, size: 1.5 },
      { x: 77, y: 60, size: 2.1 },
      { x: 62, y: 73, size: 1.4 },
      { x: 43, y: 68, size: 1.3 },
    ],
    lines: [[0, 1], [1, 2], [2, 3], [3, 4], [3, 5], [5, 6], [6, 7], [7, 8]],
  },
  处女座: {
    stars: [
      { x: 20, y: 20, size: 1.5 },
      { x: 34, y: 34, size: 1.8 },
      { x: 25, y: 51, size: 1.3 },
      { x: 43, y: 62, size: 1.7 },
      { x: 57, y: 47, size: 1.4 },
      { x: 72, y: 34, size: 2.1 },
      { x: 80, y: 53, size: 1.4 },
      { x: 66, y: 72, size: 1.7 },
      { x: 47, y: 80, size: 1.3 },
    ],
    lines: [[0, 1], [1, 2], [2, 3], [3, 4], [4, 5], [5, 6], [6, 7], [7, 8]],
  },
  天秤座: {
    stars: [
      { x: 25, y: 26, size: 1.5 },
      { x: 50, y: 20, size: 2 },
      { x: 75, y: 27, size: 1.5 },
      { x: 64, y: 49, size: 1.7 },
      { x: 36, y: 49, size: 1.7 },
      { x: 50, y: 72, size: 2 },
      { x: 28, y: 73, size: 1.3 },
      { x: 72, y: 73, size: 1.3 },
    ],
    lines: [[0, 1], [1, 2], [2, 3], [3, 5], [5, 4], [4, 0], [4, 6], [3, 7]],
  },
  天蝎座: {
    stars: [
      { x: 17, y: 30, size: 1.5 },
      { x: 29, y: 39, size: 1.4 },
      { x: 42, y: 34, size: 1.8 },
      { x: 53, y: 45, size: 1.4 },
      { x: 61, y: 59, size: 1.8 },
      { x: 72, y: 67, size: 1.5 },
      { x: 82, y: 58, size: 2.2 },
      { x: 76, y: 43, size: 1.4 },
      { x: 64, y: 37, size: 1.3 },
      { x: 88, y: 42, size: 1.5 },
    ],
    lines: [[0, 1], [1, 2], [2, 3], [3, 4], [4, 5], [5, 6], [6, 7], [7, 8], [6, 9]],
  },
  射手座: {
    stars: [
      { x: 20, y: 65, size: 1.5 },
      { x: 34, y: 48, size: 1.4 },
      { x: 49, y: 55, size: 1.8 },
      { x: 62, y: 37, size: 1.5 },
      { x: 76, y: 20, size: 2.2 },
      { x: 69, y: 54, size: 1.5 },
      { x: 82, y: 68, size: 1.7 },
      { x: 52, y: 77, size: 1.3 },
      { x: 37, y: 76, size: 1.4 },
    ],
    lines: [[0, 1], [1, 2], [2, 3], [3, 4], [3, 5], [5, 6], [2, 7], [7, 8]],
  },
  摩羯座: {
    stars: [
      { x: 18, y: 35, size: 1.5 },
      { x: 33, y: 22, size: 1.8 },
      { x: 49, y: 32, size: 1.4 },
      { x: 66, y: 26, size: 1.7 },
      { x: 80, y: 43, size: 2 },
      { x: 68, y: 60, size: 1.5 },
      { x: 51, y: 70, size: 1.8 },
      { x: 35, y: 62, size: 1.3 },
      { x: 25, y: 78, size: 1.4 },
    ],
    lines: [[0, 1], [1, 2], [2, 3], [3, 4], [4, 5], [5, 6], [6, 7], [7, 8]],
  },
  水瓶座: {
    stars: [
      { x: 18, y: 28, size: 1.5 },
      { x: 34, y: 39, size: 1.8 },
      { x: 48, y: 27, size: 1.4 },
      { x: 62, y: 42, size: 1.8 },
      { x: 80, y: 31, size: 1.5 },
      { x: 71, y: 61, size: 2 },
      { x: 52, y: 72, size: 1.5 },
      { x: 33, y: 64, size: 1.7 },
      { x: 19, y: 77, size: 1.3 },
    ],
    lines: [[0, 1], [1, 2], [2, 3], [3, 4], [3, 5], [5, 6], [6, 7], [7, 8]],
  },
  双鱼座: {
    stars: [
      { x: 22, y: 23, size: 1.5 },
      { x: 39, y: 33, size: 1.8 },
      { x: 53, y: 48, size: 1.4 },
      { x: 68, y: 62, size: 1.8 },
      { x: 82, y: 76, size: 2 },
      { x: 74, y: 25, size: 1.5 },
      { x: 60, y: 34, size: 1.4 },
      { x: 48, y: 53, size: 1.7 },
      { x: 33, y: 68, size: 1.3 },
    ],
    lines: [[0, 1], [1, 2], [2, 3], [3, 4], [5, 6], [6, 7], [7, 8], [2, 7]],
  },
}

const currentConstellation = computed(
  () => constellationMaps[selectedZodiac.value] ?? constellationMaps['天蝎座'],
)

function openPanel(panel: Exclude<PanelType, null>) {
  activePanel.value = panel
}

function closePanel() {
  activePanel.value = null
}
</script>

<template>
  <div class="astral-app">
    <div class="cosmos-bg"></div>
    <div class="nebula nebula-one"></div>
    <div class="nebula nebula-two"></div>
    <div class="star-dust"></div>

    <header class="topbar">
      <div class="brand">
        <div class="brand-symbol">✧</div>
        <div>
          <div class="brand-name">星阙</div>
          <div class="brand-en">ASTRAL PALACE</div>
        </div>
      </div>

      <div class="topbar-center">
        <span class="topbar-line"></span>
        <span>探索星辰 · 读懂自己</span>
        <span class="topbar-line"></span>
      </div>

      <div class="topbar-date">{{ today }}</div>
    </header>

    <main class="dashboard">
      <!-- 左侧：今日运势 -->
      <aside class="left-panel">
        <div class="section-eyebrow">
          <span class="tiny-star">✦</span>
          DAILY HOROSCOPE
        </div>

        <div class="side-heading">今日星象</div>
        <div class="side-subtitle">聆听属于你的宇宙讯息</div>

        <div class="zodiac-current">
          <div class="current-symbol">{{ currentZodiac.symbol }}</div>
          <div class="current-info">
            <div class="current-name">{{ currentZodiac.name }}</div>
            <div class="current-date">{{ currentZodiac.date }}</div>
          </div>
          <div class="current-element">{{ currentZodiac.element }}</div>
        </div>

        <div class="fortune-card">
          <div class="fortune-top">
            <span>今日综合指数</span>
            <span class="fortune-star">✦</span>
          </div>
          <div class="fortune-score">
            {{ fortune }}<small>/100</small>
          </div>
          <div class="fortune-track">
            <div
              class="fortune-progress"
              :style="{ width: `${fortune}%` }"
            ></div>
          </div>
          <div class="fortune-level">
            <span>星象能量</span>
            <span>{{ fortune >= 90 ? '能量充沛' : fortune >= 85 ? '状态良好' : '平稳前行' }}</span>
          </div>
        </div>

        <div class="fortune-message">
          <div class="message-title">
            <span>✧</span>
            今日寄语
          </div>
          <p>
            不必急于寻找所有问题的答案。
            保持内心的清醒，在自己的节奏里前行，
            你会发现新的可能。
          </p>
        </div>

        <button class="outline-button" @click="openPanel('zodiac')">
          探索我的星座
          <span>↗</span>
        </button>
      </aside>

      <!-- 中央：星阙星盘 -->
      <section class="universe">
        <div class="universe-heading">
          <div class="universe-kicker">THE CELESTIAL MAP</div>
          <div class="universe-title">十二星辰 · 命运之轮</div>
          <div class="universe-desc">在星辰的轨迹中，寻找属于自己的坐标</div>
        </div>

        <div class="zodiac-wheel">
          <div class="wheel-glow"></div>

          <div class="wheel-orbit orbit-outer"></div>
          <div class="wheel-orbit orbit-middle"></div>
          <div class="wheel-orbit orbit-inner"></div>
          <div class="wheel-ticks"></div>

          <!-- 十二宫几何结构与外圈星点 -->
          <svg
            class="zodiac-geometry"
            viewBox="0 0 600 600"
            aria-hidden="true"
          >
            <defs>
              <radialGradient id="starGlow">
                <stop offset="0%" stop-color="#f4dfae" />
                <stop offset="100%" stop-color="#d8bd83" stop-opacity="0.15" />
              </radialGradient>
            </defs>

            <g class="house-dividers">
              <line
                v-for="index in 12"
                :key="`divider-${index}`"
                :x1="300 + 178 * Math.cos(((index - 1) * 30 + 15) * Math.PI / 180)"
                :y1="300 + 178 * Math.sin(((index - 1) * 30 + 15) * Math.PI / 180)"
                :x2="300 + 284 * Math.cos(((index - 1) * 30 + 15) * Math.PI / 180)"
                :y2="300 + 284 * Math.sin(((index - 1) * 30 + 15) * Math.PI / 180)"
              />
            </g>

            <g class="outer-stars">
              <circle
                v-for="index in 36"
                :key="`star-${index}`"
                :cx="300 + 290 * Math.cos((index - 1) * 10 * Math.PI / 180)"
                :cy="300 + 290 * Math.sin((index - 1) * 10 * Math.PI / 180)"
                :r="index % 3 === 0 ? 2 : 1"
              />
            </g>
          </svg>

          <!-- 当前选中星座的星点与联线 -->
          <Transition name="constellation-fade" mode="out-in">
            <svg
              :key="selectedZodiac"
              class="active-constellation"
              viewBox="0 0 100 100"
              role="img"
              :aria-label="`${selectedZodiac}星座轨迹示意图`"
            >
              <defs>
                <filter id="constellationGlow" x="-100%" y="-100%" width="300%" height="300%">
                  <feGaussianBlur stdDeviation="1.8" result="blur" />
                  <feMerge>
                    <feMergeNode in="blur" />
                    <feMergeNode in="SourceGraphic" />
                  </feMerge>
                </filter>
              </defs>

              <g class="constellation-lines">
                <line
                  v-for="(line, index) in currentConstellation.lines"
                  :key="`line-${index}`"
                  :x1="currentConstellation.stars[line[0]].x"
                  :y1="currentConstellation.stars[line[0]].y"
                  :x2="currentConstellation.stars[line[1]].x"
                  :y2="currentConstellation.stars[line[1]].y"
                  :style="{ animationDelay: `${index * 0.09}s` }"
                />
              </g>

              <g class="constellation-stars" filter="url(#constellationGlow)">
                <circle
                  v-for="(star, index) in currentConstellation.stars"
                  :key="`point-${index}`"
                  :cx="star.x"
                  :cy="star.y"
                  :r="star.size ?? 1.5"
                  :style="{ animationDelay: `${index * 0.13}s` }"
                />
              </g>
            </svg>
          </Transition>

          <!-- 行星装饰标记：视觉元素，并非实时天体坐标 -->
          <div class="planet-marker planet-sun">
            <span>☉</span>
            <small>太阳</small>
          </div>
          <div class="planet-marker planet-moon">
            <span>☽</span>
            <small>月亮</small>
          </div>
          <div class="planet-marker planet-mercury">
            <span>☿</span>
            <small>水星</small>
          </div>
          <div class="planet-marker planet-saturn">
            <span>♄</span>
            <small>土星</small>
          </div>

          <div class="wheel-cross cross-horizontal"></div>
          <div class="wheel-cross cross-vertical"></div>

          <!-- 十二星座：按圆周坐标分布 -->
          <div
            v-for="(zodiac, index) in zodiacs"
            :key="zodiac.name"
            class="wheel-zodiac"
            :class="{ selected: selectedZodiac === zodiac.name }"
            :style="{
              left: `${50 + 42 * Math.cos(index * Math.PI / 6)}%`,
              top: `${50 + 42 * Math.sin(index * Math.PI / 6)}%`,
            }"
          >
            <button
              class="zodiac-node"
              :aria-label="`选择${zodiac.name}`"
              :aria-pressed="selectedZodiac === zodiac.name"
              @click="selectedZodiac = zodiac.name"
            >
              <span class="node-symbol">{{ zodiac.symbol }}</span>
              <span class="node-name">{{ zodiac.name }}</span>
            </button>
          </div>

          <div class="wheel-center">
            <div class="center-orbit"></div>
            <div class="center-star">✧</div>
            <div class="center-name">星阙</div>
            <div class="center-en">ASTRAL PALACE</div>
            <div class="center-divider"></div>
            <div class="center-caption">THE UNIVERSE WITHIN</div>
          </div>

          <div class="wheel-decoration decoration-top">✦</div>
          <div class="wheel-decoration decoration-bottom">✧</div>
          <div class="wheel-decoration decoration-left">✧</div>
          <div class="wheel-decoration decoration-right">✦</div>
        </div>

        <div class="wheel-instruction">
          <span class="instruction-dot"></span>
          点击星盘上的星座，探索不同的星辰能量
        </div>
      </section>

      <!-- 右侧：功能入口 -->
      <aside class="right-panel">
        <div class="section-eyebrow">
          <span class="tiny-star">✦</span>
          EXPLORE THE UNIVERSE
        </div>

        <div class="side-heading">星辰秘境</div>
        <div class="side-subtitle">开启你的探索之旅</div>

        <div class="feature-list">
          <button
            v-for="feature in features"
            :key="feature.id"
            class="feature-card"
            @click="openPanel(feature.id)"
          >
            <div class="feature-icon">{{ feature.symbol }}</div>
            <div class="feature-content">
              <div class="feature-name">{{ feature.name }}</div>
              <div class="feature-en">{{ feature.en }}</div>
              <div class="feature-desc">{{ feature.desc }}</div>
            </div>
            <div class="feature-arrow">↗</div>
          </button>
        </div>

        <div class="right-quote">
          <div class="quote-mark">“</div>
          <p>仰望星空，也别忘了探索自己的内心。</p>
          <div class="quote-line"></div>
        </div>
      </aside>
    </main>

    <footer class="bottom-bar">
      <div class="bottom-left">
        <span class="bottom-star">✦</span>
        星阙 · ASTRAL PALACE
      </div>
      <div class="bottom-center">星辰指引方向，答案始终在你心中</div>
      <div class="bottom-right">仅供兴趣探索与娱乐参考</div>
    </footer>

    <!-- 功能悬浮面板 -->
    <Teleport to="body">
      <Transition name="panel-fade">
        <div
          v-if="activePanel"
          class="panel-overlay"
          @click.self="closePanel"
        >
          <section
            class="feature-modal"
            role="dialog"
            aria-modal="true"
            :aria-label="panelTitle"
          >
            <header class="modal-header">
              <div class="modal-heading">
                <span class="modal-heading-star">✧</span>
                <span>{{ panelTitle }}</span>
              </div>
              <button
                class="modal-close"
                aria-label="关闭面板"
                @click="closePanel"
              >
                ×
              </button>
            </header>

            <div class="modal-content">
              <ZodiacPage v-if="activePanel === 'zodiac'" />
              <CompatibilityPage v-else-if="activePanel === 'compatibility'" />
              <TestPage v-else-if="activePanel === 'test'" />
              <BirthChartPage v-else-if="activePanel === 'chart'" />
            </div>
          </section>
        </div>
      </Transition>
    </Teleport>
  </div>
</template>

<style>
:root {
  color-scheme: dark;
}

html,
body,
#app {
  width: 100%;
  height: 100%;
  min-height: 100%;
  margin: 0;
  overflow: hidden;
}

body {
  font-family:
    "Microsoft YaHei",
    "PingFang SC",
    "Noto Sans SC",
    sans-serif;
  background: #070b1b;
}

* {
  box-sizing: border-box;
}

.astral-app {
  --gold: #d8bd83;
  --gold-light: #f0dfb3;
  --muted-gold: rgba(216, 189, 131, 0.55);
  position: relative;
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100dvh;
  min-height: 0;
  overflow: hidden;
  isolation: isolate;
  color: #eee9df;
  background:
    radial-gradient(ellipse at 50% 48%, rgba(50, 57, 111, 0.32), transparent 42%),
    linear-gradient(135deg, #080d20 0%, #10152d 48%, #080b1a 100%);
}

.cosmos-bg,
.star-dust,
.nebula {
  position: absolute;
  inset: 0;
  pointer-events: none;
}

.cosmos-bg {
  z-index: -4;
  opacity: 0.85;
  background-image:
    radial-gradient(1px 1px at 8% 17%, #fff 80%, transparent),
    radial-gradient(1px 1px at 22% 72%, #d8d9ff 80%, transparent),
    radial-gradient(2px 2px at 42% 18%, #fff 80%, transparent),
    radial-gradient(1px 1px at 67% 26%, #fff 80%, transparent),
    radial-gradient(1px 1px at 83% 66%, #d9d5ff 80%, transparent),
    radial-gradient(2px 2px at 92% 13%, #fff 80%, transparent),
    radial-gradient(1px 1px at 13% 89%, #fff 80%, transparent),
    radial-gradient(1px 1px at 57% 87%, #fff 80%, transparent);
  background-size: 360px 300px;
}

.star-dust {
  z-index: -3;
  opacity: 0.38;
  background-image:
    radial-gradient(1px 1px at 15% 30%, #e9d9b0 70%, transparent),
    radial-gradient(1px 1px at 75% 40%, #aab7ff 70%, transparent),
    radial-gradient(1px 1px at 45% 80%, #fff 70%, transparent);
  background-size: 190px 170px;
}

.nebula {
  z-index: -2;
  filter: blur(60px);
  opacity: 0.25;
}

.nebula-one {
  background: radial-gradient(ellipse at 28% 45%, rgba(70, 69, 153, 0.6), transparent 42%);
}

.nebula-two {
  background: radial-gradient(ellipse at 76% 60%, rgba(74, 49, 121, 0.42), transparent 38%);
}

/* 顶部 */
.topbar {
  position: relative;
  z-index: 2;
  flex: 0 0 76px;
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  align-items: center;
  padding: 0 4.2%;
  border-bottom: 1px solid rgba(216, 189, 131, 0.13);
  background: rgba(5, 9, 24, 0.28);
}

.brand {
  display: flex;
  align-items: center;
  gap: 12px;
}

.brand-symbol {
  color: var(--gold-light);
  font-size: 36px;
  line-height: 1;
  text-shadow: 0 0 20px rgba(226, 201, 144, 0.65);
}

.brand-name {
  color: #f0dfb3;
  font-family: "STSong", "Songti SC", serif;
  font-size: 25px;
  letter-spacing: 7px;
}

.brand-en {
  margin-top: 3px;
  color: rgba(216, 189, 131, 0.55);
  font-size: 8px;
  letter-spacing: 3px;
}

.topbar-center {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 17px;
  color: rgba(230, 224, 211, 0.6);
  font-size: 12px;
  letter-spacing: 3px;
}

.topbar-line {
  width: 35px;
  height: 1px;
  background: linear-gradient(90deg, transparent, var(--muted-gold));
}

.topbar-line:last-child {
  transform: rotate(180deg);
}

.topbar-date {
  justify-self: end;
  color: rgba(222, 211, 190, 0.55);
  font-size: 11px;
  letter-spacing: 1px;
}

/* 主界面 */
.dashboard {
  position: relative;
  z-index: 1;
  flex: 1;
  display: grid;
  grid-template-columns: minmax(220px, 0.82fr) minmax(420px, 1.8fr) minmax(220px, 0.82fr);
  align-items: center;
  gap: clamp(18px, 3vw, 54px);
  width: min(1440px, 94%);
  min-height: 0;
  margin: 0 auto;
  padding: 18px 0 12px;
}

.left-panel,
.right-panel {
  width: 100%;
  max-width: 300px;
  justify-self: center;
}

.section-eyebrow {
  color: rgba(216, 189, 131, 0.65);
  font-size: 9px;
  letter-spacing: 2.5px;
}

.tiny-star {
  margin-right: 6px;
  color: var(--gold-light);
}

.side-heading {
  margin-top: 15px;
  color: #f0e5cf;
  font-family: "STSong", "Songti SC", serif;
  font-size: 25px;
  letter-spacing: 5px;
}

.side-subtitle {
  margin-top: 8px;
  color: rgba(221, 216, 207, 0.48);
  font-size: 11px;
  letter-spacing: 1px;
}

.zodiac-current {
  display: flex;
  align-items: center;
  gap: 13px;
  margin-top: 32px;
  padding: 17px 15px;
  border: 1px solid rgba(216, 189, 131, 0.2);
  background: linear-gradient(115deg, rgba(50, 48, 83, 0.42), rgba(18, 23, 49, 0.4));
}

.current-symbol {
  color: var(--gold-light);
  font-size: 35px;
  text-shadow: 0 0 18px rgba(226, 201, 144, 0.35);
}

.current-info {
  flex: 1;
}

.current-name {
  color: #f0e5cf;
  font-family: "STSong", "Songti SC", serif;
  font-size: 18px;
  letter-spacing: 2px;
}

.current-date {
  margin-top: 5px;
  color: rgba(220, 212, 194, 0.48);
  font-size: 10px;
  letter-spacing: 1px;
}

.current-element {
  align-self: flex-start;
  padding: 4px 7px;
  border: 1px solid rgba(216, 189, 131, 0.2);
  color: rgba(225, 205, 161, 0.72);
  font-size: 9px;
}

.fortune-card {
  margin-top: 14px;
  padding: 18px 17px;
  border: 1px solid rgba(216, 189, 131, 0.16);
  background: rgba(12, 18, 42, 0.55);
}

.fortune-top,
.fortune-level {
  display: flex;
  align-items: center;
  justify-content: space-between;
  color: rgba(225, 218, 203, 0.63);
  font-size: 11px;
}

.fortune-star {
  color: var(--gold);
}

.fortune-score {
  margin-top: 10px;
  color: #f0dfb3;
  font-family: Georgia, serif;
  font-size: 45px;
  line-height: 1.1;
}

.fortune-score small {
  margin-left: 5px;
  color: rgba(225, 218, 203, 0.4);
  font-family: Arial, sans-serif;
  font-size: 11px;
}

.fortune-track {
  height: 3px;
  margin-top: 15px;
  overflow: hidden;
  background: rgba(255, 255, 255, 0.1);
}

.fortune-progress {
  height: 100%;
  background: linear-gradient(90deg, #9c83b8, #e0c78e);
  box-shadow: 0 0 12px rgba(224, 199, 142, 0.55);
}

.fortune-level {
  margin-top: 10px;
  font-size: 10px;
}

.fortune-level span:last-child {
  color: var(--gold-light);
}

.fortune-message {
  margin-top: 18px;
  padding-left: 13px;
  border-left: 1px solid rgba(216, 189, 131, 0.5);
}

.message-title {
  color: #e4d3ac;
  font-size: 12px;
  letter-spacing: 2px;
}

.message-title span {
  margin-right: 6px;
}

.fortune-message p {
  margin: 10px 0 0;
  color: rgba(225, 220, 209, 0.58);
  font-family: "STSong", "Songti SC", serif;
  font-size: 12px;
  line-height: 2;
  letter-spacing: 0.5px;
}

.outline-button {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
  margin-top: 22px;
  padding: 12px 14px;
  border: 1px solid rgba(216, 189, 131, 0.36);
  color: #e7d7b2;
  background: rgba(216, 189, 131, 0.04);
  font-size: 11px;
  letter-spacing: 2px;
  cursor: pointer;
  transition: 0.25s ease;
}

.outline-button:hover {
  border-color: var(--gold-light);
  background: rgba(216, 189, 131, 0.12);
}

.outline-button span {
  font-size: 16px;
}

/* 中央星盘 */
.universe {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-width: 0;
  height: 100%;
}

.universe-heading {
  margin-bottom: 8px;
  text-align: center;
}

.universe-kicker {
  color: rgba(216, 189, 131, 0.65);
  font-family: Georgia, serif;
  font-size: 9px;
  letter-spacing: 4px;
}

.universe-title {
  margin-top: 8px;
  color: #f0e4c9;
  font-family: "STSong", "Songti SC", serif;
  font-size: clamp(19px, 2vw, 25px);
  letter-spacing: 6px;
}

.universe-desc {
  margin-top: 8px;
  color: rgba(221, 216, 207, 0.45);
  font-size: 10px;
  letter-spacing: 2px;
}

.zodiac-wheel {
  position: relative;
  width: min(52vw, 620px, 65vh);
  aspect-ratio: 1;
  flex: 0 1 auto;
  max-height: 100%;
  margin: 0 auto;
  border-radius: 50%;
}

.wheel-glow {
  position: absolute;
  inset: 17%;
  border-radius: 50%;
  background: radial-gradient(circle, rgba(96, 88, 160, 0.22), rgba(37, 46, 93, 0.1) 48%, transparent 72%);
  filter: blur(20px);
  animation: glow-breathe 7s ease-in-out infinite;
}

.wheel-orbit {
  position: absolute;
  top: 50%;
  left: 50%;
  border: 1px solid rgba(216, 189, 131, 0.3);
  border-radius: 50%;
  transform: translate(-50%, -50%);
}

.orbit-outer {
  width: 96%;
  height: 96%;
  border-color: rgba(216, 189, 131, 0.32);
  box-shadow: 0 0 20px rgba(180, 156, 105, 0.08);
  animation: orbit-rotate 100s linear infinite;
}

.orbit-middle {
  width: 79%;
  height: 79%;
  border-color: rgba(174, 169, 221, 0.35);
  animation: orbit-rotate 75s linear infinite reverse;
}

.orbit-inner {
  width: 59%;
  height: 59%;
  border-color: rgba(216, 189, 131, 0.3);
  border-style: dashed;
  animation: orbit-rotate 55s linear infinite;
}

.wheel-ticks {
  position: absolute;
  inset: 4%;
  border: 1px dashed rgba(216, 189, 131, 0.22);
  border-radius: 50%;
  background: repeating-conic-gradient(
    from 0deg,
    rgba(216, 189, 131, 0.5) 0deg 0.35deg,
    transparent 0.35deg 5deg
  );
  mask: radial-gradient(transparent 68%, #000 68.5%, #000 69%, transparent 69.5%);
  animation: orbit-rotate 180s linear infinite reverse;
}

/* 十二宫几何结构 */
.zodiac-geometry {
  position: absolute;
  z-index: 1;
  inset: 0;
  width: 100%;
  height: 100%;
  overflow: visible;
  pointer-events: none;
}

.house-dividers line {
  stroke: rgba(216, 189, 131, 0.28);
  stroke-width: 0.8;
  stroke-dasharray: 3 5;
}

.outer-stars {
  fill: #e9d8b0;
  opacity: 0.7;
}

.outer-stars circle:nth-child(3n) {
  fill: #fff0c9;
  filter: drop-shadow(0 0 5px rgba(240, 223, 179, 0.9));
}

/* 当前星座星图 */
.active-constellation {
  position: absolute;
  z-index: 2;
  top: 50%;
  left: 50%;
  width: 72%;
  height: 72%;
  overflow: visible;
  transform: translate(-50%, -50%);
  pointer-events: none;
}

.constellation-lines line {
  stroke: rgba(222, 205, 169, 0.48);
  stroke-width: 0.42;
  stroke-linecap: round;
  stroke-dasharray: 100;
  stroke-dashoffset: 100;
  animation: constellation-draw 1.4s ease forwards;
}

.constellation-stars circle {
  fill: #f5e5bd;
  opacity: 0;
  animation: constellation-twinkle 1.2s ease forwards;
}

/* 切换星座时淡入淡出 */
.constellation-fade-enter-active,
.constellation-fade-leave-active {
  transition: opacity 0.45s ease, filter 0.45s ease;
}

.constellation-fade-enter-from,
.constellation-fade-leave-to {
  opacity: 0;
  filter: blur(4px);
}

/* 行星装饰标记 */
.planet-marker {
  position: absolute;
  z-index: 4;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
  width: 36px;
  color: #f0dfb3;
  text-shadow: 0 0 12px rgba(240, 223, 179, 0.7);
  pointer-events: none;
  animation: planet-breathe 4s ease-in-out infinite;
}

.planet-marker span {
  font-family: Georgia, serif;
  font-size: 25px;
  line-height: 1;
}

.planet-marker small {
  color: rgba(225, 211, 177, 0.65);
  font-size: 8px;
  letter-spacing: 1px;
}

.planet-sun {
  top: 27%;
  left: 68%;
}

.planet-moon {
  top: 34%;
  left: 25%;
  animation-delay: -1s;
}

.planet-mercury {
  top: 65%;
  left: 69%;
  animation-delay: -2s;
}

.planet-saturn {
  top: 68%;
  left: 28%;
  animation-delay: -3s;
}

.wheel-cross {
  position: absolute;
  top: 50%;
  left: 50%;
  background: rgba(216, 189, 131, 0.12);
  transform: translate(-50%, -50%);
}

.cross-horizontal {
  width: 100%;
  height: 1px;
}

.cross-vertical {
  width: 1px;
  height: 100%;
}

/* 十二星座按钮 */
.wheel-zodiac {
  position: absolute;
  z-index: 5;
  width: 58px;
  height: 58px;
  transform: translate(-50%, -50%);
  cursor: pointer;
}

.zodiac-node {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
  padding: 0;
  border: 1px solid rgba(216, 189, 131, 0.3);
  border-radius: 50%;
  color: rgba(225, 214, 188, 0.8);
  background: rgba(12, 17, 39, 0.96);
  box-shadow: 0 0 16px rgba(0, 0, 0, 0.3);
  cursor: pointer;
  transition:
    border-color 0.25s ease,
    color 0.25s ease,
    background 0.25s ease,
    box-shadow 0.25s ease,
    transform 0.25s ease;
}

.zodiac-node:hover,
.wheel-zodiac.selected .zodiac-node {
  border-color: var(--gold-light);
  color: #f4dfae;
  background: rgba(66, 58, 91, 0.98);
  box-shadow:
    0 0 10px rgba(216, 189, 131, 0.3),
    0 0 28px rgba(216, 189, 131, 0.2);
  transform: scale(1.1);
}

.node-symbol {
  font-size: 23px;
  line-height: 1.2;
}

.node-name {
  margin-top: 2px;
  font-size: 8px;
  letter-spacing: 1px;
}

/* 中央品牌 */
.wheel-center {
  position: absolute;
  z-index: 3;
  top: 50%;
  left: 50%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 38%;
  height: 38%;
  border: 1px solid rgba(216, 189, 131, 0.38);
  border-radius: 50%;
  background:
    radial-gradient(circle at 50% 40%, rgba(74, 71, 123, 0.5), rgba(13, 19, 43, 0.95) 68%);
  box-shadow:
    0 0 28px rgba(97, 89, 164, 0.18),
    inset 0 0 28px rgba(216, 189, 131, 0.06);
  transform: translate(-50%, -50%);
  pointer-events: none;
}

.center-orbit {
  position: absolute;
  inset: 8%;
  border: 1px dashed rgba(216, 189, 131, 0.24);
  border-radius: 50%;
  animation: orbit-rotate 35s linear infinite;
}

.center-star {
  color: #f0dfb3;
  font-size: 37px;
  line-height: 1;
  text-shadow: 0 0 20px rgba(240, 223, 179, 0.7);
}

.center-name {
  margin-top: 8px;
  color: #f0dfb3;
  font-family: "STSong", "Songti SC", serif;
  font-size: 24px;
  letter-spacing: 8px;
  text-indent: 8px;
}

.center-en {
  margin-top: 5px;
  color: rgba(216, 189, 131, 0.58);
  font-family: Georgia, serif;
  font-size: 7px;
  letter-spacing: 2px;
}

.center-divider {
  width: 45px;
  height: 1px;
  margin-top: 13px;
  background: linear-gradient(90deg, transparent, var(--gold), transparent);
}

.center-caption {
  margin-top: 9px;
  color: rgba(225, 218, 203, 0.4);
  font-size: 7px;
  letter-spacing: 1.5px;
}

.wheel-decoration {
  position: absolute;
  z-index: 1;
  color: rgba(240, 223, 179, 0.8);
  text-shadow: 0 0 15px rgba(240, 223, 179, 0.6);
  pointer-events: none;
}

.decoration-top {
  top: 1%;
  left: 49%;
}

.decoration-bottom {
  bottom: 1%;
  left: 49%;
}

.decoration-left {
  top: 49%;
  left: 1%;
}

.decoration-right {
  top: 49%;
  right: 1%;
}

.wheel-instruction {
  margin-top: 5px;
  color: rgba(221, 216, 207, 0.45);
  font-size: 10px;
  letter-spacing: 1px;
}

.instruction-dot {
  display: inline-block;
  width: 5px;
  height: 5px;
  margin-right: 7px;
  border-radius: 50%;
  background: var(--gold);
  box-shadow: 0 0 8px var(--gold);
  vertical-align: middle;
}

/* 右侧功能 */
.feature-list {
  display: flex;
  flex-direction: column;
  gap: 11px;
  margin-top: 29px;
}

.feature-card {
  display: flex;
  align-items: center;
  width: 100%;
  min-height: 75px;
  padding: 12px 11px;
  border: 1px solid rgba(216, 189, 131, 0.17);
  color: #eee5d3;
  text-align: left;
  background: linear-gradient(110deg, rgba(41, 42, 75, 0.48), rgba(14, 20, 44, 0.48));
  cursor: pointer;
  transition: 0.25s ease;
}

.feature-card:hover {
  border-color: rgba(240, 223, 179, 0.6);
  background: linear-gradient(110deg, rgba(72, 62, 102, 0.58), rgba(23, 30, 61, 0.7));
  transform: translateX(-4px);
}

.feature-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 43px;
  height: 43px;
  flex: 0 0 43px;
  border: 1px solid rgba(216, 189, 131, 0.3);
  border-radius: 50%;
  color: var(--gold-light);
  font-size: 22px;
}

.feature-content {
  flex: 1;
  min-width: 0;
  padding-left: 12px;
}

.feature-name {
  color: #eee3cb;
  font-family: "STSong", "Songti SC", serif;
  font-size: 15px;
  letter-spacing: 2px;
}

.feature-en {
  margin-top: 4px;
  color: rgba(216, 189, 131, 0.5);
  font-family: Georgia, serif;
  font-size: 7px;
  letter-spacing: 1.3px;
}

.feature-desc {
  margin-top: 5px;
  color: rgba(225, 220, 209, 0.48);
  font-size: 9px;
}

.feature-arrow {
  padding: 0 3px;
  color: rgba(216, 189, 131, 0.65);
  font-size: 17px;
}

.right-quote {
  position: relative;
  margin-top: 24px;
  padding-left: 12px;
  border-left: 1px solid rgba(216, 189, 131, 0.35);
}

.quote-mark {
  height: 20px;
  color: rgba(216, 189, 131, 0.55);
  font-family: Georgia, serif;
  font-size: 30px;
  line-height: 1;
}

.right-quote p {
  margin: 5px 0 0;
  color: rgba(225, 220, 209, 0.48);
  font-family: "STSong", "Songti SC", serif;
  font-size: 11px;
  line-height: 1.8;
  letter-spacing: 1px;
}

.quote-line {
  width: 28px;
  height: 1px;
  margin-top: 10px;
  background: rgba(216, 189, 131, 0.45);
}

/* 底部 */
.bottom-bar {
  position: relative;
  z-index: 2;
  flex: 0 0 43px;
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  align-items: center;
  padding: 0 4.2%;
  border-top: 1px solid rgba(216, 189, 131, 0.12);
  color: rgba(221, 216, 207, 0.36);
  font-size: 9px;
  letter-spacing: 1px;
}

.bottom-left {
  color: rgba(216, 189, 131, 0.6);
  font-family: Georgia, serif;
  letter-spacing: 2px;
}

.bottom-star {
  margin-right: 7px;
  color: var(--gold);
}

.bottom-center {
  text-align: center;
}

.bottom-right {
  text-align: right;
}

/* 悬浮功能面板 */
.panel-overlay {
  position: fixed;
  z-index: 1000;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 28px;
  background: rgba(3, 6, 18, 0.78);
  backdrop-filter: blur(13px);
}

.feature-modal {
  display: flex;
  flex-direction: column;
  width: min(1050px, 94vw);
  height: min(820px, 90dvh);
  min-height: 0;
  overflow: hidden;
  border: 1px solid rgba(216, 189, 131, 0.38);
  background:
    radial-gradient(ellipse at 50% 0%, rgba(66, 65, 119, 0.25), transparent 55%),
    #0c1228;
  box-shadow:
    0 0 70px rgba(0, 0, 0, 0.65),
    0 0 35px rgba(120, 103, 174, 0.12);
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex: 0 0 58px;
  padding: 0 22px;
  border-bottom: 1px solid rgba(216, 189, 131, 0.18);
  background: rgba(5, 10, 25, 0.5);
}

.modal-heading {
  color: #edddb8;
  font-family: "STSong", "Songti SC", serif;
  font-size: 16px;
  letter-spacing: 3px;
}

.modal-heading-star {
  margin-right: 10px;
  color: var(--gold);
}

.modal-close {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  border: 1px solid rgba(216, 189, 131, 0.25);
  border-radius: 50%;
  color: #e7d8b6;
  background: transparent;
  font-size: 23px;
  line-height: 1;
  cursor: pointer;
  transition: 0.2s ease;
}

.modal-close:hover {
  border-color: var(--gold-light);
  background: rgba(216, 189, 131, 0.12);
}

.modal-content {
  flex: 1;
  min-height: 0;
  overflow: auto;
  overscroll-behavior: contain;
  padding: 18px;
}

.panel-fade-enter-active,
.panel-fade-leave-active {
  transition: opacity 0.25s ease;
}

.panel-fade-enter-from,
.panel-fade-leave-to {
  opacity: 0;
}

/* 中等屏幕 */
@media (max-width: 1100px) {
  .dashboard {
    grid-template-columns: minmax(190px, 0.8fr) minmax(360px, 1.5fr) minmax(190px, 0.8fr);
    gap: 15px;
    width: 96%;
  }

  .wheel-zodiac {
    width: 50px;
    height: 50px;
  }

  .node-symbol {
    font-size: 20px;
  }

  .node-name {
    font-size: 7px;
  }

  .feature-card {
    min-height: 68px;
  }

  .right-quote {
    margin-top: 15px;
  }
}

/* 平板与手机 */
@media (max-width: 820px) {
  .topbar {
    flex-basis: 58px;
    padding: 0 18px;
    grid-template-columns: 1fr auto;
  }

  .brand-name {
    font-size: 21px;
  }

  .brand-symbol {
    font-size: 29px;
  }

  .topbar-center {
    display: none;
  }

  .topbar-date {
    font-size: 9px;
  }

  .dashboard {
    display: grid;
    grid-template-columns: 1fr;
    grid-template-rows: auto minmax(0, 1fr) auto;
    gap: 5px;
    width: 100%;
    padding: 8px 14px 5px;
  }

  .left-panel {
    display: grid;
    grid-template-columns: 1fr auto;
    align-items: center;
    gap: 4px 10px;
    width: 100%;
    max-width: none;
  }

  .left-panel .section-eyebrow,
  .left-panel .side-heading,
  .left-panel .side-subtitle,
  .fortune-message,
  .outline-button {
    display: none;
  }

  .zodiac-current {
    grid-column: 1;
    grid-row: 1;
    gap: 8px;
    margin: 0;
    padding: 5px 9px;
    border: none;
    background: transparent;
  }

  .current-symbol {
    font-size: 26px;
  }

  .current-name {
    font-size: 14px;
  }

  .current-date {
    font-size: 8px;
  }

  .current-element {
    font-size: 8px;
  }

  .fortune-card {
    grid-column: 2;
    grid-row: 1;
    width: 135px;
    margin: 0;
    padding: 7px 9px;
  }

  .fortune-top {
    font-size: 8px;
  }

  .fortune-score {
    margin-top: 2px;
    font-size: 25px;
  }

  .fortune-track {
    margin-top: 5px;
  }

  .fortune-level {
    margin-top: 4px;
    font-size: 8px;
  }

  .universe {
    min-height: 0;
  }

  .universe-heading {
    margin-bottom: 0;
  }

  .universe-kicker {
    font-size: 7px;
  }

  .universe-title {
    margin-top: 4px;
    font-size: 17px;
    letter-spacing: 4px;
  }

  .universe-desc {
    margin-top: 4px;
    font-size: 8px;
  }

  .zodiac-wheel {
    width: min(75vw, 48vh, 470px);
    max-height: 100%;
  }

  .wheel-zodiac {
    width: 45px;
    height: 45px;
  }

  .node-symbol {
    font-size: 18px;
  }

  .node-name {
    font-size: 7px;
  }

  .center-star {
    font-size: 27px;
  }

  .center-name {
    margin-top: 4px;
    font-size: 18px;
    letter-spacing: 5px;
  }

  .center-en {
    font-size: 6px;
  }

  .center-divider {
    margin-top: 7px;
  }

  .center-caption {
    margin-top: 5px;
    font-size: 6px;
  }

  .wheel-instruction {
    margin-top: 0;
    font-size: 8px;
  }

  .planet-marker span {
    font-size: 19px;
  }

  .planet-marker small {
    font-size: 7px;
  }

  .right-panel {
    display: grid;
    grid-template-columns: repeat(4, minmax(0, 1fr));
    gap: 7px;
    width: 100%;
    max-width: none;
    padding-bottom: 3px;
  }

  .right-panel .section-eyebrow,
  .right-panel .side-heading,
  .right-panel .side-subtitle,
  .right-quote {
    display: none;
  }

  .feature-list {
    display: contents;
  }

  .feature-card {
    flex-direction: column;
    justify-content: center;
    min-width: 0;
    min-height: 64px;
    padding: 5px 2px;
    text-align: center;
  }

  .feature-card:hover {
    transform: translateY(-2px);
  }

  .feature-icon {
    width: 27px;
    height: 27px;
    flex-basis: 27px;
    font-size: 15px;
  }

  .feature-content {
    padding: 4px 0 0;
  }

  .feature-name {
    font-size: 10px;
    letter-spacing: 1px;
  }

  .feature-en,
  .feature-desc,
  .feature-arrow {
    display: none;
  }

  .bottom-bar {
    flex-basis: 27px;
    grid-template-columns: 1fr 1fr;
    padding: 0 14px;
    font-size: 8px;
  }

  .bottom-center {
    display: none;
  }

  .panel-overlay {
    padding: 10px;
  }

  .feature-modal {
    width: 100%;
    height: 96dvh;
  }

  .modal-content {
    padding: 10px;
  }
}

@media (max-width: 430px) {
  .topbar {
    flex-basis: 51px;
  }

  .brand-name {
    font-size: 18px;
  }

  .brand-en {
    font-size: 7px;
  }

  .topbar-date {
    font-size: 8px;
  }

  .dashboard {
    padding-right: 9px;
    padding-left: 9px;
  }

  .zodiac-wheel {
    width: min(82vw, 43vh, 390px);
  }

  .wheel-zodiac {
    width: 39px;
    height: 39px;
  }

  .node-symbol {
    font-size: 16px;
  }

  .node-name {
    font-size: 6px;
  }

  .fortune-card {
    width: 118px;
  }

  .fortune-score {
    font-size: 22px;
  }

  .feature-card {
    min-height: 57px;
  }

  .bottom-right {
    font-size: 7px;
  }
}

/* 动态效果 */
@keyframes orbit-rotate {
  from {
    transform: translate(-50%, -50%) rotate(0deg);
  }
  to {
    transform: translate(-50%, -50%) rotate(360deg);
  }
}

@keyframes glow-breathe {
  0%, 100% {
    opacity: 0.65;
    transform: scale(0.96);
  }
  50% {
    opacity: 1;
    transform: scale(1.05);
  }
}

@keyframes planet-breathe {
  0%, 100% {
    opacity: 0.65;
    transform: scale(0.96);
  }
  50% {
    opacity: 1;
    transform: scale(1.08);
  }
}

@keyframes constellation-draw {
  to {
    stroke-dashoffset: 0;
  }
}

@keyframes constellation-twinkle {
  0% {
    opacity: 0;
    transform: scale(0.4);
  }
  70% {
    opacity: 0.95;
    transform: scale(1.25);
  }
  100% {
    opacity: 0.72;
    transform: scale(1);
  }
}

/* 尊重系统减少动态效果设置 */
@media (prefers-reduced-motion: reduce) {
  .orbit-outer,
  .orbit-middle,
  .orbit-inner,
  .wheel-ticks,
  .wheel-glow,
  .center-orbit,
  .planet-marker {
    animation: none !important;
  }

  .constellation-fade-enter-active,
  .constellation-fade-leave-active {
    transition: none;
  }

  .constellation-lines line,
  .constellation-stars circle {
    animation: none;
    opacity: 0.75;
    stroke-dashoffset: 0;
  }
}
</style>