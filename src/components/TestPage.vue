
<script setup lang="ts">
import { computed, ref } from 'vue'

type AnswerType = 'A' | 'B' | 'C' | 'D'

const tests = [
  {
    id: 'personality',
    title: '性格探索',
    subtitle: '发现你不曾留意的自己',
    icon: '✧',
    description: '从日常选择出发，看看你更接近哪一种性格。',
    questions: [
      {
        title: '结束了一周忙碌的工作，你最想怎样度过周末？',
        options: [
          { text: '约上朋友，去热闹的地方放松', type: 'A' },
          { text: '待在家里，享受独处的时间', type: 'B' },
          { text: '去一个陌生的地方走走', type: 'C' },
          { text: '安排好计划，把想做的事逐一完成', type: 'D' },
        ],
      },
      {
        title: '面对一件重要的决定，你通常会？',
        options: [
          { text: '相信直觉，先行动再调整', type: 'A' },
          { text: '仔细思考自己的真实感受', type: 'B' },
          { text: '搜集信息，比较不同可能', type: 'C' },
          { text: '制定计划，评估风险和结果', type: 'D' },
        ],
      },
      {
        title: '朋友遇到困难向你倾诉，你更可能？',
        options: [
          { text: '主动陪伴，带对方走出低落', type: 'A' },
          { text: '认真倾听，理解对方的情绪', type: 'B' },
          { text: '帮忙分析问题，寻找解决办法', type: 'C' },
          { text: '给出实际建议，帮助制定行动方案', type: 'D' },
        ],
      },
      {
        title: '来到一个陌生的社交场合，你会？',
        options: [
          { text: '主动认识别人，很快融入气氛', type: 'A' },
          { text: '先观察环境，再决定是否交流', type: 'B' },
          { text: '寻找感兴趣的话题，和人深入交流', type: 'C' },
          { text: '保持礼貌，按照自己的节奏参与', type: 'D' },
        ],
      },
      {
        title: '你最希望别人如何评价你？',
        options: [
          { text: '热情、有感染力', type: 'A' },
          { text: '真诚、细腻、有深度', type: 'B' },
          { text: '聪明、独立、有想法', type: 'C' },
          { text: '可靠、稳重、有担当', type: 'D' },
        ],
      },
    ],
    profiles: {
      A: { title: '热情行动型', slogan: '你像一束光，愿意主动拥抱生活。', detail: '你通常有较强的行动意愿，喜欢与外界建立联系，也容易为周围的人带来活力。面对机会，你更愿意先迈出一步。', strength: '积极主动，感染力强，适应新环境较快。', growth: '有时容易因为热情而忽略细节，也需要给自己留出冷静思考的时间。' },
      B: { title: '敏感感知型', slogan: '你拥有细腻的内心，也懂得倾听自己。', detail: '你重视内心感受，善于察觉情绪变化。相比表面的热闹，你更看重真实的连接和有质量的独处。', strength: '共情能力较强，观察细致，重视情感深度。', growth: '不要总把别人的感受放在自己前面，适当表达需求也是一种力量。' },
      C: { title: '独立探索型', slogan: '你喜欢思考，也愿意寻找自己的答案。', detail: '你对新鲜事物保持好奇，习惯从不同角度理解问题。你需要一定的自由空间，也不喜欢被固定模式限制。', strength: '好奇心强，思维灵活，愿意尝试不同可能。', growth: '想法太多时容易分散精力，选定方向并持续行动，会让你的优势更容易发挥。' },
      D: { title: '沉稳规划型', slogan: '你相信积累，也重视长期的价值。', detail: '你倾向于先观察、再判断、后行动。面对复杂问题，你更愿意建立秩序，一步一步把事情做好。', strength: '责任感强，做事有条理，稳定性较好。', growth: '不必要求每件事都有完美计划，适当接受变化，也会带来新的机会。' },
    },
  },
  {
    id: 'love',
    title: '爱情模式',
    subtitle: '了解你在亲密关系中的表现',
    icon: '♡',
    description: '从情感与相处方式出发，探索你的爱情倾向。',
    questions: [
      {
        title: '喜欢一个人时，你通常会？',
        options: [
          { text: '主动表达好感，希望关系有所进展', type: 'A' },
          { text: '默默关注，确认感觉后再靠近', type: 'B' },
          { text: '先成为朋友，从交流中了解对方', type: 'C' },
          { text: '观察对方是否认真，再决定投入多少', type: 'D' },
        ],
      },
      {
        title: '恋人情绪低落时，你最希望？',
        options: [
          { text: '立刻陪在对方身边，给予拥抱', type: 'A' },
          { text: '耐心倾听，让对方说出心里话', type: 'B' },
          { text: '一起分析原因，寻找解决办法', type: 'C' },
          { text: '给对方空间，等对方准备好再交流', type: 'D' },
        ],
      },
      {
        title: '你认为一段感情最重要的是什么？',
        options: [
          { text: '热烈的爱意和持续的心动', type: 'A' },
          { text: '理解、陪伴和情绪上的回应', type: 'B' },
          { text: '精神交流和共同成长', type: 'C' },
          { text: '忠诚、责任和稳定的未来', type: 'D' },
        ],
      },
      {
        title: '发生争执时，你更倾向于？',
        options: [
          { text: '尽快说清楚，不希望冷战', type: 'A' },
          { text: '先平复情绪，再认真沟通', type: 'B' },
          { text: '弄清楚问题本身，找到双方都能接受的办法', type: 'C' },
          { text: '需要一些时间独处，想清楚再谈', type: 'D' },
        ],
      },
      {
        title: '你理想中的亲密关系是？',
        options: [
          { text: '充满激情，一起体验丰富的生活', type: 'A' },
          { text: '温柔安心，彼此理解和珍惜', type: 'B' },
          { text: '有趣自由，既相爱也能共同探索世界', type: 'C' },
          { text: '踏实长久，能够一起面对现实生活', type: 'D' },
        ],
      },
    ],
    profiles: {
      A: { title: '热烈主动型', slogan: '爱就要表达，让感情保持温度。', detail: '你重视感情中的回应和互动，愿意主动创造浪漫，也希望两个人能够坦诚表达喜欢。', strength: '表达直接，愿意付出，能够为关系注入热情。', growth: '感情不必时时保持高强度，给彼此稳定和空间同样重要。' },
      B: { title: '深情共鸣型', slogan: '你渴望的，是被真正理解的感觉。', detail: '你在意情感交流的深度，关注细节，也重视双方能否理解彼此的情绪。', strength: '善于倾听，重视陪伴，能够感知关系中的细微变化。', growth: '不要期待对方自动读懂你的心思，清楚表达自己的需要很重要。' },
      C: { title: '精神同行型', slogan: '好的爱情，是两个独立灵魂的相互吸引。', detail: '你看重思想交流和共同成长，希望伴侣既是爱人，也是能够分享观点、探索世界的同行者。', strength: '尊重独立，重视交流，愿意与伴侣共同成长。', growth: '理性沟通之外，也要照顾情感需求，让关心被对方真实感受到。' },
      D: { title: '稳定守护型', slogan: '比起短暂的心动，你更珍惜长久的安心。', detail: '你重视承诺和实际行动，希望关系有清晰的方向。你愿意通过长期经营建立可靠的感情基础。', strength: '认真负责，忠诚度高，重视长期关系。', growth: '稳定不意味着一成不变，偶尔创造新鲜感，也能让关系保持活力。' },
    },
  },
  {
    id: 'soul',
    title: '内在需求',
    subtitle: '探索你真正看重的东西',
    icon: '☾',
    description: '看看什么样的生活状态，最能让你感到满足。',
    questions: [
      {
        title: '如果可以暂时放下所有压力，你最想做什么？',
        options: [
          { text: '去体验新鲜事物，看看更大的世界', type: 'A' },
          { text: '和重要的人待在一起，享受温暖时光', type: 'B' },
          { text: '读书、思考，深入探索感兴趣的领域', type: 'C' },
          { text: '整理生活，让一切重新变得有序', type: 'D' },
        ],
      },
      {
        title: '什么最容易让你感到疲惫？',
        options: [
          { text: '生活一成不变，缺少新的可能', type: 'A' },
          { text: '关系疏远，缺少理解和陪伴', type: 'B' },
          { text: '没有思考空间，无法做自己', type: 'C' },
          { text: '生活失去秩序，事情无法掌控', type: 'D' },
        ],
      },
      {
        title: '面对人生的选择，你最不愿意失去什么？',
        options: [
          { text: '自由和尝试的机会', type: 'A' },
          { text: '爱与重要关系', type: 'B' },
          { text: '独立思考和自我表达', type: 'C' },
          { text: '安全感和稳定基础', type: 'D' },
        ],
      },
      {
        title: '当你独自安静下来，最常思考什么？',
        options: [
          { text: '还有哪些地方值得去，哪些事值得尝试', type: 'A' },
          { text: '自己与身边人的关系是否足够亲近', type: 'B' },
          { text: '生活的意义，以及自己真正相信什么', type: 'C' },
          { text: '未来如何安排，怎样让生活更踏实', type: 'D' },
        ],
      },
      {
        title: '你希望未来的自己处于怎样的状态？',
        options: [
          { text: '不断探索，拥有丰富的人生体验', type: 'A' },
          { text: '内心温暖，有值得珍惜的人相伴', type: 'B' },
          { text: '保持清醒，拥有独立而丰盈的精神世界', type: 'C' },
          { text: '生活稳定，拥有选择的底气', type: 'D' },
        ],
      },
    ],
    profiles: {
      A: { title: '自由探索需求', slogan: '你的内心需要空间，也需要远方。', detail: '你重视新鲜感、选择权和人生体验。对你来说，生活不仅是完成任务，更是不断发现新的可能。', strength: '开放、有好奇心，面对变化时愿意尝试。', growth: '自由也需要方向，找到值得长期投入的事情，能让探索更有深度。' },
      B: { title: '情感连接需求', slogan: '真正的满足，来自人与人之间的温度。', detail: '你看重关系中的理解、陪伴和归属感。重要的人以及彼此之间的情感连接，是你生活动力的重要来源。', strength: '重视感情，善于建立联系，珍惜身边的人。', growth: '在照顾他人的同时，也要照顾自己的需要，不必用牺牲自己换取关系。' },
      C: { title: '精神成长需求', slogan: '你希望理解世界，也希望理解自己。', detail: '你需要思考、学习和表达的空间。比起表面的成功，你更在意内心是否清醒，生活是否有意义。', strength: '有思考深度，重视自我认识，愿意持续学习。', growth: '不要只停留在思考中，把认知转化为行动，才能真正改变生活。' },
      D: { title: '安全秩序需求', slogan: '拥有稳定的基础，才能安心走向远方。', detail: '你重视生活的确定性，希望拥有可靠的环境、清晰的目标和可以掌控的节奏。', strength: '务实、稳健，擅长规划和长期积累。', growth: '人生并非所有事情都能提前安排，适当接受不确定性，也是一种成长。' },
    },
  },
]

const stage = ref<'list' | 'quiz' | 'result'>('list')
const activeTestId = ref('personality')
const currentIndex = ref(0)
const answers = ref<AnswerType[]>([])

const activeTest = computed(
  () => tests.find(test => test.id === activeTestId.value) ?? tests[0],
)

const currentQuestion = computed(
  () => activeTest.value.questions[currentIndex.value],
)

const progress = computed(
  () => ((currentIndex.value + 1) / activeTest.value.questions.length) * 100,
)

const resultType = computed<AnswerType>(() => {
  const counts: Record<AnswerType, number> = { A: 0, B: 0, C: 0, D: 0 }
  answers.value.forEach(answer => { counts[answer]++ })
  return (['A', 'B', 'C', 'D'] as AnswerType[])
    .sort((a, b) => counts[b] - counts[a])[0]
})

const resultProfile = computed(
  () => activeTest.value.profiles[resultType.value],
)

function startTest(id: string) {
  activeTestId.value = id
  currentIndex.value = 0
  answers.value = []
  stage.value = 'quiz'
}

function chooseAnswer(type: AnswerType) {
  answers.value[currentIndex.value] = type
}

function nextQuestion() {
  if (!answers.value[currentIndex.value]) return

  if (currentIndex.value < activeTest.value.questions.length - 1) {
    currentIndex.value++
  } else {
    stage.value = 'result'
  }
}

function previousQuestion() {
  if (currentIndex.value > 0) currentIndex.value--
}

function backToList() {
  stage.value = 'list'
  answers.value = []
  currentIndex.value = 0
}
</script>

<template>
  <div class="test-page">
    <section class="test-intro">
      <span class="test-eyebrow">DISCOVER YOURSELF</span>
      <h1>认识自己，从一个问题开始</h1>
      <p>有些答案，藏在你不经意的选择里。</p>
    </section>

    <section v-if="stage === 'list'" class="test-list">
      <button
        v-for="test in tests"
        :key="test.id"
        class="test-card"
        @click="startTest(test.id)"
      >
        <span class="test-card-icon">{{ test.icon }}</span>
        <span class="test-card-content">
          <strong>{{ test.title }}</strong>
          <small>{{ test.subtitle }}</small>
          <span class="test-card-description">{{ test.description }}</span>
        </span>
        <span class="test-arrow">→</span>
      </button>

      <div class="test-notice">
        <span>✦ 温馨提示</span>
        <p>测试结果根据你的答题选择生成，帮助你进行自我观察与探索，仅供兴趣参考，不构成心理诊断或专业测评。</p>
      </div>
    </section>

    <section v-else-if="stage === 'quiz'" class="quiz-card">
      <div class="quiz-top">
        <button class="text-back" @click="backToList">← 返回测试列表</button>
        <span>{{ activeTest.title }}</span>
      </div>

      <div class="progress-info">
        <span>问题 {{ currentIndex + 1 }} / {{ activeTest.questions.length }}</span>
        <span>{{ Math.round(progress) }}%</span>
      </div>
      <div class="progress-track">
        <div class="progress-fill" :style="{ width: progress + '%' }"></div>
      </div>

      <div class="question-area">
        <span class="question-number">QUESTION {{ String(currentIndex + 1).padStart(2, '0') }}</span>
        <h2>{{ currentQuestion.title }}</h2>
        <p>请选择最符合你真实想法的一项</p>
      </div>

      <div class="answer-list">
        <button
          v-for="(option, index) in currentQuestion.options"
          :key="option.type"
          class="answer-option"
          :class="{ chosen: answers[currentIndex] === option.type }"
          @click="chooseAnswer(option.type as AnswerType)"
        >
          <span class="answer-letter">{{ ['A', 'B', 'C', 'D'][index] }}</span>
          <span>{{ option.text }}</span>
          <span class="answer-check">{{ answers[currentIndex] === option.type ? '✓' : '' }}</span>
        </button>
      </div>

      <div class="quiz-actions">
        <button
          v-if="currentIndex > 0"
          class="previous-button"
          @click="previousQuestion"
        >上一题</button>
        <span v-else></span>
        <button
          class="next-button"
          :disabled="!answers[currentIndex]"
          @click="nextQuestion"
        >
          {{ currentIndex === activeTest.questions.length - 1 ? '查看测试结果' : '下一题 →' }}
        </button>
      </div>
    </section>

    <section v-else class="result-card">
      <span class="result-eyebrow">YOUR PERSONAL RESULT</span>
      <div class="result-symbol">{{ activeTest.icon }}</div>
      <span class="result-type">你的测试结果</span>
      <h2>{{ resultProfile.title }}</h2>
      <p class="result-slogan">{{ resultProfile.slogan }}</p>

      <div class="result-content">
        <article>
          <span>✧ 你的内在画像</span>
          <p>{{ resultProfile.detail }}</p>
        </article>
        <article>
          <span>♡ 你拥有的优势</span>
          <p>{{ resultProfile.strength }}</p>
        </article>
        <article>
          <span>☾ 可以关注的成长方向</span>
          <p>{{ resultProfile.growth }}</p>
        </article>
      </div>

      <div class="result-disclaimer">
        这是一份基于本次选择生成的趣味结果，不代表固定的人格标签。人的性格与需求会随着经历和环境不断变化。
      </div>

      <div class="result-actions">
        <button class="again-button" @click="startTest(activeTest.id)">重新测试</button>
        <button class="list-button" @click="backToList">其他测试</button>
      </div>
    </section>
  </div>
</template>

<style scoped>
.test-page {
  width: min(100% - 36px, 760px);
  margin: 0 auto;
  padding: 48px 0 110px;
  color: #f5efff;
}
.test-intro { text-align: center; margin-bottom: 34px; }
.test-eyebrow, .result-eyebrow {
  color: #d8bb8a;
  font-size: 11px;
  letter-spacing: 3px;
}
.test-intro h1 {
  margin: 15px 0 10px;
  font-size: clamp(25px, 5vw, 34px);
}
.test-intro p { color: #aaa2bb; font-size: 14px; line-height: 1.8; }
.test-list { display: grid; gap: 16px; }
.test-card {
  width: 100%;
  display: flex;
  align-items: center;
  gap: 18px;
  padding: 23px 20px;
  text-align: left;
  color: #f5efff;
  border: 1px solid rgba(205, 180, 255, .18);
  border-radius: 18px;
  background: linear-gradient(130deg, rgba(53, 43, 83, .85), rgba(22, 25, 50, .9));
  cursor: pointer;
  transition: border-color .2s, transform .2s;
}
.test-card:hover { border-color: rgba(220, 190, 240, .6); transform: translateY(-2px); }
.test-card-icon {
  width: 54px;
  height: 54px;
  flex: 0 0 54px;
  display: grid;
  place-items: center;
  border-radius: 16px;
  background: rgba(196, 166, 224, .12);
  color: #e2c59a;
  font-size: 27px;
}
.test-card-content { display: grid; gap: 6px; flex: 1; }
.test-card-content strong { font-size: 17px; }
.test-card-content small { color: #d8bb8a; font-size: 12px; }
.test-card-description { color: #aaa2bb; font-size: 12px; line-height: 1.6; }
.test-arrow { color: #d8bb8a; font-size: 22px; }
.test-notice {
  margin-top: 10px;
  padding: 19px;
  border-radius: 14px;
  background: rgba(255, 255, 255, .035);
}
.test-notice span { color: #d8bb8a; font-size: 13px; }
.test-notice p { color: #9992aa; font-size: 12px; line-height: 1.9; margin-bottom: 0; }
.quiz-card, .result-card {
  padding: 26px;
  border: 1px solid rgba(205, 180, 255, .2);
  border-radius: 22px;
  background: linear-gradient(145deg, rgba(48, 39, 82, .9), rgba(19, 23, 48, .95));
}
.quiz-top { display: flex; align-items: center; justify-content: space-between; gap: 10px; }
.quiz-top > span { color: #d8bb8a; font-size: 13px; }
.text-back {
  padding: 7px 0;
  border: 0;
  background: transparent;
  color: #aaa2bb;
  cursor: pointer;
  font-size: 12px;
}
.progress-info { display: flex; justify-content: space-between; margin-top: 28px; color: #aaa2bb; font-size: 12px; }
.progress-track { height: 5px; margin-top: 10px; overflow: hidden; border-radius: 5px; background: rgba(255,255,255,.1); }
.progress-fill { height: 100%; border-radius: 5px; background: linear-gradient(90deg, #a98bd0, #e1c18f); transition: width .25s; }
.question-area { padding: 35px 0 24px; }
.question-number { color: #d8bb8a; font-size: 10px; letter-spacing: 2px; }
.question-area h2 { margin: 15px 0 10px; font-size: 22px; line-height: 1.6; }
.question-area p { color: #9992aa; font-size: 12px; }
.answer-list { display: grid; gap: 12px; }
.answer-option {
  width: 100%;
  display: flex;
  align-items: center;
  gap: 13px;
  padding: 15px;
  text-align: left;
  color: #e9e1f3;
  border: 1px solid rgba(205, 180, 255, .16);
  border-radius: 12px;
  background: rgba(255,255,255,.035);
  cursor: pointer;
  font-size: 13px;
  line-height: 1.6;
}
.answer-option.chosen {
  border-color: #c9a9e5;
  background: rgba(171, 135, 207, .16);
}
.answer-letter {
  width: 27px;
  height: 27px;
  flex: 0 0 27px;
  display: grid;
  place-items: center;
  border-radius: 50%;
  background: rgba(205, 180, 255, .12);
  color: #dfc49b;
  font-size: 11px;
}
.answer-check { margin-left: auto; color: #e1c18f; }
.quiz-actions { display: flex; justify-content: space-between; align-items: center; gap: 12px; margin-top: 28px; }
.previous-button, .next-button, .again-button, .list-button {
  padding: 12px 20px;
  border-radius: 10px;
  cursor: pointer;
  font-size: 13px;
}
.previous-button, .list-button {
  border: 1px solid rgba(205, 180, 255, .3);
  background: transparent;
  color: #e8dafa;
}
.next-button, .again-button {
  border: 0;
  background: linear-gradient(100deg, #b99bda, #e1c18f);
  color: #211a35;
  font-weight: 700;
}
.next-button:disabled { opacity: .35; cursor: not-allowed; }
.result-card { text-align: center; }
.result-symbol { margin: 22px auto 12px; color: #e1c18f; font-size: 42px; }
.result-type { color: #aaa2bb; font-size: 12px; }
.result-card h2 { margin: 12px 0; font-size: 28px; }
.result-slogan { color: #e1c18f; font-size: 15px; line-height: 1.8; }
.result-content { margin-top: 28px; text-align: left; }
.result-content article { padding: 18px 0; border-top: 1px solid rgba(205,180,255,.15); }
.result-content article span { color: #d8bb8a; font-size: 14px; }
.result-content article p { margin: 9px 0 0; color: #b8adc9; font-size: 13px; line-height: 1.9; }
.result-disclaimer { margin-top: 18px; padding: 15px; border-radius: 12px; background: rgba(255,255,255,.04); color: #9992aa; font-size: 11px; line-height: 1.8; text-align: left; }
.result-actions { display: flex; justify-content: center; gap: 12px; margin-top: 24px; }
@media (max-width: 480px) {
  .test-page { width: min(100% - 24px, 760px); padding-top: 32px; }
  .quiz-card, .result-card { padding: 20px 15px; }
  .question-area h2 { font-size: 19px; }
  .test-card { padding: 18px 14px; gap: 12px; }
  .test-card-icon { width: 46px; height: 46px; flex-basis: 46px; }
  .result-actions button { padding: 11px 15px; }
}
</style>
