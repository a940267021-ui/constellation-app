
<script setup lang="ts">
import { computed, ref } from 'vue'

const zodiacs = [
  { name: '白羊座', symbol: '♈', element: '火象' },
  { name: '金牛座', symbol: '♉', element: '土象' },
  { name: '双子座', symbol: '♊', element: '风象' },
  { name: '巨蟹座', symbol: '♋', element: '水象' },
  { name: '狮子座', symbol: '♌', element: '火象' },
  { name: '处女座', symbol: '♍', element: '土象' },
  { name: '天秤座', symbol: '♎', element: '风象' },
  { name: '天蝎座', symbol: '♏', element: '水象' },
  { name: '射手座', symbol: '♐', element: '火象' },
  { name: '摩羯座', symbol: '♑', element: '土象' },
  { name: '水瓶座', symbol: '♒', element: '风象' },
  { name: '双鱼座', symbol: '♓', element: '水象' },
]

const first = ref('天蝎座')
const second = ref('双鱼座')
const showResult = ref(false)

const firstZodiac = computed(
  () => zodiacs.find(item => item.name === first.value) ?? zodiacs[7],
)

const secondZodiac = computed(
  () => zodiacs.find(item => item.name === second.value) ?? zodiacs[11],
)

const result = computed(() => {
  const a = firstZodiac.value.element
  const b = secondZodiac.value.element

  let score = 75
  let title = '需要理解与磨合'
  let description = '你们有各自的特点，需要通过沟通逐渐找到舒服的相处方式。'

  if (first.value === second.value) {
    score = 90
    title = '相似的灵魂'
    description = '你们容易理解彼此的想法，也可能因为性格相似而产生相同的坚持。'
  } else if (a === b) {
    score = 86
    title = '同频共振'
    description = '相同元素让你们更容易理解彼此的表达方式，但也要注意相似性带来的盲点。'
  } else if (
    (a === '火象' && b === '风象') ||
    (a === '风象' && b === '火象') ||
    (a === '土象' && b === '水象') ||
    (a === '水象' && b === '土象')
  ) {
    score = 92
    title = '彼此吸引，互相成就'
    description = '你们的元素特质具有一定互补性，容易在相处中发现对方身上的闪光点。'
  } else if (
    (a === '火象' && b === '水象') ||
    (a === '水象' && b === '火象')
  ) {
    score = 68
    title = '热情与细腻的碰撞'
    description = '一方可能更直接主动，另一方更在意情绪感受。理解彼此的表达习惯很重要。'
  } else if (
    (a === '土象' && b === '风象') ||
    (a === '风象' && b === '土象')
  ) {
    score = 70
    title = '现实与自由的平衡'
    description = '一方重视稳定与计划，另一方更喜欢变化和探索。相互尊重生活节奏是关键。'
  } else if (
    (a === '火象' && b === '土象') ||
    (a === '土象' && b === '火象')
  ) {
    score = 74
    title = '行动与稳健的互补'
    description = '一方倾向迅速行动，另一方习惯谨慎考虑。找到共同节奏，能够减少摩擦。'
  } else {
    score = 76
    title = '感性与理性的交流'
    description = '你们看待问题的角度可能不同，坦诚表达需求，有助于建立稳定的关系。'
  }

  return { score, title, description }
})

const loveAdvice = computed(() => {
  if (result.value.score >= 90) {
    return '珍惜彼此的默契，也要给对方保留独立的空间。不要因为熟悉，就忽略表达爱意。'
  }
  if (result.value.score >= 80) {
    return '多表达欣赏，少用自己的标准要求对方。理解差异，比追求完全一致更重要。'
  }
  return '遇到分歧时先倾听，再表达自己的感受。不要急着判断谁对谁错。'
})

function calculate() {
  showResult.value = true
}

function reset() {
  showResult.value = false
}
</script>

<template>
  <div class="compatibility-page">
    <section class="intro">
      <span class="eyebrow">COSMIC CONNECTION</span>
      <h1>两颗星星的相遇</h1>
      <p>选择你们的星座，探索彼此之间的默契与可能。</p>
    </section>

    <section class="match-card">
      <div class="match-heading">
        <span>LOVE COMPATIBILITY</span>
        <span class="heading-star">✦</span>
      </div>

      <div class="selectors">
        <div class="person">
          <div class="person-symbol">{{ firstZodiac.symbol }}</div>
          <label>选择你的星座</label>
          <select v-model="first" @change="reset">
            <option v-for="item in zodiacs" :key="item.name" :value="item.name">
              {{ item.name }}
            </option>
          </select>
          <small>{{ firstZodiac.element }}</small>
        </div>

        <div class="heart">♡</div>

        <div class="person">
          <div class="person-symbol">{{ secondZodiac.symbol }}</div>
          <label>选择对方的星座</label>
          <select v-model="second" @change="reset">
            <option v-for="item in zodiacs" :key="item.name" :value="item.name">
              {{ item.name }}
            </option>
          </select>
          <small>{{ secondZodiac.element }}</small>
        </div>
      </div>

      <button class="calculate-button" @click="calculate">
        ✧ 开始测算配对
      </button>

      <div v-if="showResult" class="result">
        <div class="result-divider"><span>YOUR COSMIC MATCH</span></div>

        <div class="score-ring">
          <strong>{{ result.score }}<small>%</small></strong>
          <span>契合指数</span>
        </div>

        <h2>{{ result.title }}</h2>
        <p class="result-description">{{ result.description }}</p>

        <div class="analysis-list">
          <article>
            <span class="analysis-icon">♡</span>
            <div>
              <strong>情感默契</strong>
              <p>关注彼此真正需要什么，不要只用自己的方式表达关心。</p>
            </div>
          </article>
          <article>
            <span class="analysis-icon">✧</span>
            <div>
              <strong>相处模式</strong>
              <p>尊重彼此的性格差异，在共同兴趣中建立连接。</p>
            </div>
          </article>
          <article>
            <span class="analysis-icon">☾</span>
            <div>
              <strong>相处建议</strong>
              <p>{{ loveAdvice }}</p>
            </div>
          </article>
        </div>

        <button class="reset-button" @click="reset">重新选择星座</button>
      </div>
    </section>

    <section class="notice">
      <span>✦ 关于配对结果</span>
      <p>本页面依据传统占星中的四元素关系设计，仅供兴趣探索与娱乐参考。实际关系取决于双方的性格、沟通、选择与共同经营，不能由星座分数决定。</p>
    </section>
  </div>
</template>

<style scoped>
.compatibility-page {
  width: min(100% - 36px, 760px);
  margin: 0 auto;
  padding: 48px 0 100px;
  color: #f5efff;
}
.intro { text-align: center; margin-bottom: 32px; }
.eyebrow {
  color: #d8bb8a;
  font-size: 11px;
  letter-spacing: 3px;
}
.intro h1 { font-size: clamp(26px, 5vw, 36px); margin: 15px 0 10px; }
.intro p { color: #a9a1bb; font-size: 14px; line-height: 1.8; }
.match-card {
  padding: 28px;
  border: 1px solid rgba(205, 180, 255, .2);
  border-radius: 22px;
  background: linear-gradient(145deg, rgba(48, 39, 82, .9), rgba(19, 23, 48, .95));
  box-shadow: 0 18px 60px rgba(0, 0, 0, .2);
}
.match-heading {
  display: flex;
  justify-content: space-between;
  color: #d8bb8a;
  font-size: 11px;
  letter-spacing: 2px;
}
.heading-star { font-size: 18px; }
.selectors {
  display: grid;
  grid-template-columns: 1fr 48px 1fr;
  align-items: center;
  gap: 12px;
  margin: 35px 0;
}
.person { text-align: center; min-width: 0; }
.person-symbol {
  width: 94px;
  height: 94px;
  margin: 0 auto 18px;
  display: grid;
  place-items: center;
  border-radius: 50%;
  border: 1px solid rgba(221, 195, 255, .35);
  background: radial-gradient(circle, rgba(139, 105, 190, .35), rgba(42, 37, 73, .2));
  color: #e5ca99;
  font-size: 42px;
}
.person label, .person small { display: block; color: #aaa2bd; font-size: 12px; }
.person select {
  width: 100%;
  max-width: 175px;
  margin: 10px auto;
  padding: 11px 8px;
  border: 1px solid rgba(205, 180, 255, .3);
  border-radius: 10px;
  background: #27213e;
  color: #fff;
  font-size: 14px;
}
.heart { text-align: center; color: #e9b7d1; font-size: 30px; }
.calculate-button {
  width: 100%;
  padding: 15px;
  border: 0;
  border-radius: 12px;
  background: linear-gradient(100deg, #b99bda, #e1c18f);
  color: #211a35;
  font-size: 15px;
  font-weight: 700;
  cursor: pointer;
}
.calculate-button:hover { filter: brightness(1.08); }
.result { text-align: center; padding-top: 28px; }
.result-divider {
  display: flex;
  align-items: center;
  gap: 12px;
  color: #c7b3e4;
  font-size: 10px;
  letter-spacing: 2px;
}
.result-divider::before, .result-divider::after {
  content: '';
  flex: 1;
  height: 1px;
  background: rgba(205, 180, 255, .2);
}
.score-ring {
  width: 148px;
  height: 148px;
  margin: 28px auto 18px;
  border-radius: 50%;
  border: 2px solid #d9bd91;
  box-shadow: 0 0 30px rgba(201, 169, 226, .15), inset 0 0 28px rgba(201, 169, 226, .1);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
.score-ring strong { font-size: 43px; color: #e7ca9c; }
.score-ring strong small { font-size: 20px; }
.score-ring > span { color: #b8adc9; font-size: 12px; }
.result h2 { font-size: 23px; margin: 12px 0; }
.result-description { color: #b8adc9; font-size: 14px; line-height: 1.9; }
.analysis-list { text-align: left; margin-top: 26px; }
.analysis-list article {
  display: flex;
  gap: 16px;
  padding: 18px 0;
  border-top: 1px solid rgba(205, 180, 255, .14);
}
.analysis-icon { color: #d9bd91; font-size: 22px; }
.analysis-list strong { font-size: 14px; }
.analysis-list p { margin: 7px 0 0; color: #b8adc9; font-size: 13px; line-height: 1.8; }
.reset-button {
  margin-top: 22px;
  padding: 11px 25px;
  border: 1px solid rgba(205, 180, 255, .35);
  border-radius: 30px;
  background: transparent;
  color: #e8dafa;
  cursor: pointer;
}
.notice {
  margin-top: 22px;
  padding: 20px;
  border-radius: 14px;
  background: rgba(255, 255, 255, .035);
}
.notice span { color: #d9bd91; font-size: 13px; }
.notice p { color: #9992aa; font-size: 12px; line-height: 1.9; margin-bottom: 0; }
@media (max-width: 480px) {
  .compatibility-page { width: min(100% - 24px, 760px); padding-top: 32px; }
  .match-card { padding: 20px 15px; }
  .selectors { grid-template-columns: 1fr 28px 1fr; gap: 5px; }
  .person-symbol { width: 76px; height: 76px; font-size: 34px; }
  .person select { font-size: 12px; }
}
</style>
