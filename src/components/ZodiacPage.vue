
<script setup lang="ts">
import { ref } from 'vue'

const selected = ref<string | null>(null)

const zodiacs = [
  {
    name: '白羊座', symbol: '♈', date: '3月21日—4月19日',
    element: '火象星座', quality: '基本宫', planet: '火星',
    keywords: '热情、勇敢、行动力',
    description: '白羊座通常给人直接、热情、充满行动力的印象。他们喜欢主动探索，不太愿意被规则束缚。面对机会时，往往更倾向于先行动，再从实践中寻找答案。',
    strengths: '勇敢果断、精力充沛、坦率真诚',
    challenges: '有时容易冲动，缺少耐心',
    love: '在感情中重视热烈与真诚，喜欢明确表达，也希望对方给予积极回应。',
    career: '适合需要行动力、开拓精神和快速决策的领域。'
  },
  {
    name: '金牛座', symbol: '♉', date: '4月20日—5月20日',
    element: '土象星座', quality: '固定宫', planet: '金星',
    keywords: '稳定、务实、感官享受',
    description: '金牛座通常重视稳定、安全感和实际价值。他们不喜欢仓促改变，更愿意通过持续努力建立可靠的生活基础。',
    strengths: '踏实可靠、有耐心、重视承诺',
    challenges: '有时固执，不容易接受变化',
    love: '重视长期关系与实际陪伴，通常通过行动表达在意。',
    career: '适合需要耐心、规划能力、审美或资源管理的领域。'
  },
  {
    name: '双子座', symbol: '♊', date: '5月21日—6月21日',
    element: '风象星座', quality: '变动宫', planet: '水星',
    keywords: '好奇、灵活、善于交流',
    description: '双子座通常对新鲜事物充满兴趣，思维活跃，喜欢交流和获取信息。他们能够快速适应不同环境，也乐于从多个角度理解问题。',
    strengths: '反应敏捷、表达能力强、适应力好',
    challenges: '有时容易分心，缺少持续性',
    love: '重视思想交流与精神共鸣，希望关系中保有轻松和新鲜感。',
    career: '适合传播、沟通、创意、信息处理等领域。'
  },
  {
    name: '巨蟹座', symbol: '♋', date: '6月22日—7月22日',
    element: '水象星座', quality: '基本宫', planet: '月亮',
    keywords: '细腻、保护、重视家庭',
    description: '巨蟹座通常比较重视情感和归属感，对熟悉的人与环境容易产生深厚依恋。他们善于察觉情绪变化，也希望建立稳定而温暖的关系。',
    strengths: '体贴细腻、有同理心、重视责任',
    challenges: '有时容易敏感，倾向于反复回想',
    love: '重视安全感和情感回应，希望关系中有稳定的陪伴。',
    career: '适合教育、服务、照顾他人及需要情感理解的工作。'
  },
  {
    name: '狮子座', symbol: '♌', date: '7月23日—8月22日',
    element: '火象星座', quality: '固定宫', planet: '太阳',
    keywords: '自信、热情、创造力',
    description: '狮子座通常希望充分展现自己的能力，重视尊严、认可和创造价值。他们容易表现出热情，也愿意承担责任。',
    strengths: '大方自信、富有感染力、敢于担当',
    challenges: '有时过于在意评价，不容易示弱',
    love: '重视浪漫、忠诚与被重视的感觉，也愿意热情地表达爱意。',
    career: '适合管理、创意表达、组织领导及需要公众影响力的领域。'
  },
  {
    name: '处女座', symbol: '♍', date: '8月23日—9月22日',
    element: '土象星座', quality: '变动宫', planet: '水星',
    keywords: '细致、理性、追求完善',
    description: '处女座通常善于观察细节，重视秩序和效率。他们倾向于分析问题、寻找改进方法，希望事情能够更加合理和完善。',
    strengths: '认真负责、分析力强、注重细节',
    challenges: '有时容易苛求完美，过度担忧',
    love: '更重视日常相处中的细节，通过关心和实际行动表达感情。',
    career: '适合研究、分析、规划、编辑及需要精细管理的工作。'
  },
  {
    name: '天秤座', symbol: '♎', date: '9月23日—10月23日',
    element: '风象星座', quality: '基本宫', planet: '金星',
    keywords: '平衡、审美、关系协调',
    description: '天秤座通常重视公平、和谐与人际关系。他们擅长观察不同立场，希望找到各方都能接受的方式。',
    strengths: '善于沟通、富有审美、重视公平',
    challenges: '有时犹豫不决，容易顾虑过多',
    love: '重视平等、尊重和相互理解，希望两个人共同经营关系。',
    career: '适合设计、沟通、咨询、合作及需要协调能力的领域。'
  },
  {
    name: '天蝎座', symbol: '♏', date: '10月24日—11月22日',
    element: '水象星座', quality: '固定宫', planet: '冥王星、火星',
    keywords: '深刻、专注、洞察力',
    description: '天蝎座通常喜欢深入理解事物，不满足于表面答案。他们重视信任和真实，也倾向于把精力集中在真正重要的目标上。',
    strengths: '意志坚定、观察敏锐、专注持久',
    challenges: '有时容易防备，情绪不易表达',
    love: '重视忠诚与信任，希望建立深入、稳定而有深度的关系。',
    career: '适合研究、调查、策略分析及需要专注力的领域。'
  },
  {
    name: '射手座', symbol: '♐', date: '11月23日—12月21日',
    element: '火象星座', quality: '变动宫', planet: '木星',
    keywords: '自由、探索、乐观',
    description: '射手座通常热爱探索未知，重视自由与个人成长。他们喜欢接触不同文化和观点，也愿意不断拓宽自己的生活边界。',
    strengths: '乐观开朗、视野开阔、勇于尝试',
    challenges: '有时不够细致，容易缺乏耐心',
    love: '重视共同成长与个人空间，希望感情中保留自由。',
    career: '适合教育、旅行、文化交流、传播及探索型工作。'
  },
  {
    name: '摩羯座', symbol: '♑', date: '12月22日—1月19日',
    element: '土象星座', quality: '基本宫', planet: '土星',
    keywords: '责任、目标、长期积累',
    description: '摩羯座通常重视责任、秩序和长期成果。他们倾向于制定目标，再通过持续努力逐步实现，不太依赖短期运气。',
    strengths: '自律坚韧、责任心强、目标明确',
    challenges: '有时过于严肃，不善于表达情绪',
    love: '重视承诺与稳定，更愿意用长期行动证明自己的感情。',
    career: '适合管理、规划、金融、工程及需要长期积累的领域。'
  },
  {
    name: '水瓶座', symbol: '♒', date: '1月20日—2月18日',
    element: '风象星座', quality: '固定宫', planet: '天王星、土星',
    keywords: '独立、创新、理性',
    description: '水瓶座通常重视独立思考，不喜欢被传统观念限制。他们对新思想和社会变化有兴趣，倾向于寻找不同于常规的解决方式。',
    strengths: '思想独立、富有创意、包容开放',
    challenges: '有时显得疏离，不容易表达情感',
    love: '重视精神交流与彼此独立，希望伴侣既亲密又能保有自我。',
    career: '适合科技、创新、研究及需要独立思考的领域。'
  },
  {
    name: '双鱼座', symbol: '♓', date: '2月19日—3月20日',
    element: '水象星座', quality: '变动宫', planet: '海王星、木星',
    keywords: '想象、共情、感受力',
    description: '双鱼座通常富有想象力，对情绪和氛围比较敏感。他们容易理解他人的感受，也常常通过艺术、想象和直觉理解世界。',
    strengths: '富有同理心、想象力丰富、温柔包容',
    challenges: '有时容易理想化，边界感不足',
    love: '重视情感共鸣和浪漫体验，希望关系中有理解与包容。',
    career: '适合艺术、创作、服务、心理支持及需要想象力的领域。'
  }
]

function backToList() {
  selected.value = null
}
</script>

<template>
  <div class="zodiac-page">
    <template v-if="!selected">
      <div class="zodiac-hero">
        <span>TWELVE CONSTELLATIONS</span>
        <h1>十二星座</h1>
        <p>认识星座，也认识不同的自己</p>
      </div>

      <div class="zodiac-list">
        <button
          v-for="item in zodiacs"
          :key="item.name"
          class="zodiac-card"
          @click="selected = item.name"
        >
          <span class="zodiac-card-symbol">{{ item.symbol }}</span>
          <span class="zodiac-card-info">
            <strong>{{ item.name }}</strong>
            <small>{{ item.date }}</small>
          </span>
          <span class="zodiac-card-element">{{ item.element }}</span>
          <span class="zodiac-arrow">›</span>
        </button>
      </div>
    </template>

    <template v-else>
      <button class="zodiac-back" @click="backToList">← 返回十二星座</button>
      <article v-for="item in zodiacs.filter(z => z.name === selected)" :key="item.name" class="zodiac-detail">
        <div class="detail-hero">
          <span class="detail-symbol">{{ item.symbol }}</span>
          <span class="detail-kicker">YOUR CONSTELLATION</span>
          <h1>{{ item.name }}</h1>
          <p>{{ item.date }}</p>
          <div class="detail-pills">
            <span>{{ item.element }}</span>
            <span>{{ item.quality }}</span>
            <span>守护星：{{ item.planet }}</span>
          </div>
        </div>

        <section class="detail-block">
          <span class="detail-label">星座关键词</span>
          <h2>{{ item.keywords }}</h2>
          <p>{{ item.description }}</p>
        </section>

        <section class="detail-block">
          <span class="detail-label">性格特点</span>
          <div class="trait-row">
            <div><span>优势特质</span><strong>{{ item.strengths }}</strong></div>
            <div><span>成长课题</span><strong>{{ item.challenges }}</strong></div>
          </div>
        </section>

        <section class="detail-block">
          <span class="detail-label">爱情观</span>
          <p>{{ item.love }}</p>
        </section>

        <section class="detail-block">
          <span class="detail-label">事业倾向</span>
          <p>{{ item.career }}</p>
        </section>

        <p class="zodiac-disclaimer">以上内容依据传统西方占星学的星座象征整理，仅供兴趣探索与娱乐参考，不代表对个人性格或未来的科学判断。</p>
      </article>
    </template>
  </div>
</template>

<style scoped>
.zodiac-page { max-width: 850px; margin: 0 auto; padding-top: 45px; }
.zodiac-hero { text-align: center; padding: 15px 0 35px; }
.zodiac-hero > span, .detail-kicker { color: #d9bd83; font-size: 10px; letter-spacing: 3px; }
.zodiac-hero h1 { color: #f1ecfa; font-size: 32px; font-weight: 500; margin: 12px 0; }
.zodiac-hero p { color: #9995b2; font-size: 13px; }
.zodiac-list { display: grid; grid-template-columns: repeat(2, 1fr); gap: 12px; }
.zodiac-card { display: flex; align-items: center; gap: 13px; min-width: 0; padding: 16px; border: 1px solid rgba(255,255,255,.09); border-radius: 12px; background: rgba(24,27,55,.78); color: #eeeaf8; text-align: left; cursor: pointer; transition: .2s; }
.zodiac-card:hover { border-color: rgba(218,190,132,.6); transform: translateY(-2px); }
.zodiac-card-symbol { color: #d9bd83; font-size: 30px; width: 38px; text-align: center; }
.zodiac-card-info { display: flex; flex-direction: column; gap: 5px; flex: 1; }
.zodiac-card-info strong { font-size: 13px; font-weight: 500; }
.zodiac-card-info small { color: #85819d; font-size: 10px; }
.zodiac-card-element { color: #b4a9c8; font-size: 10px; white-space: nowrap; }
.zodiac-arrow { color: #d9bd83; font-size: 22px; }
.zodiac-back { margin: 0 0 22px; border: 0; background: transparent; color: #d9bd83; cursor: pointer; font-size: 13px; }
.detail-hero { padding: 30px 20px; border: 1px solid rgba(218,190,132,.25); border-radius: 16px; background: radial-gradient(ellipse at 50% 0%, rgba(112,84,160,.32), transparent 65%), #191b38; text-align: center; }
.detail-symbol { display: block; color: #e0c58f; font-size: 65px; margin-bottom: 12px; }
.detail-hero h1 { color: #f4e6c8; font-size: 32px; font-weight: 500; margin: 12px 0 6px; }
.detail-hero p { color: #aaa5bd; font-size: 12px; }
.detail-pills { display: flex; justify-content: center; flex-wrap: wrap; gap: 8px; margin-top: 20px; }
.detail-pills span { padding: 6px 12px; border: 1px solid rgba(255,255,255,.1); border-radius: 20px; color: #c9bfd6; font-size: 10px; }
.detail-block { margin-top: 14px; padding: 22px; border: 1px solid rgba(255,255,255,.08); border-radius: 13px; background: rgba(24,27,55,.78); }
.detail-label { color: #d9bd83; font-size: 10px; letter-spacing: 2px; }
.detail-block h2 { color: #f1ecfa; font-size: 19px; font-weight: 500; margin: 12px 0; }
.detail-block p { color: #b8b2ca; font-size: 13px; line-height: 2; margin: 12px 0 0; }
.trait-row { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; margin-top: 15px; }
.trait-row div { display: flex; flex-direction: column; gap: 9px; padding: 14px; border-radius: 9px; background: rgba(255,255,255,.035); }
.trait-row span { color: #9995b2; font-size: 11px; }
.trait-row strong { color: #ded6e9; font-size: 12px; line-height: 1.8; font-weight: 500; }
.zodiac-disclaimer { color: #77738e; font-size: 10px; line-height: 1.8; text-align: center; padding: 22px 12px; }
@media (max-width: 600px) {
  .zodiac-page { padding-top: 25px; }
  .zodiac-list { grid-template-columns: 1fr; gap: 9px; }
  .zodiac-card { padding: 13px; }
  .trait-row { grid-template-columns: 1fr; }
}
</style>
