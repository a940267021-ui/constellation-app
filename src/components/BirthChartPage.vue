
<script setup lang="ts">
import { computed, onMounted, onBeforeUnmount, ref } from 'vue'
import { Temporal } from '@js-temporal/polyfill'
import init, { Engine, julianDay } from 'astroceleste-engine'
import wasmUrl from 'astroceleste-engine/astroceleste_engine_wasm_bg.wasm?url'

interface City {
  name: string
  name_en: string
  lat: number
  lon: number
  population: number
}

interface Planet {
  name: string
  sign: string
  degree: number
  minute: number
  house: number
}

interface House {
  house_number: number
  sign: string
  degree: number
  minute: number
}

interface Star {
  id: number
  x: number
  y: number
  size: number
  delay: number
  opacity: number
}

const birthDate = ref('')
const birthTime = ref('')
const citySearch = ref('')
const selectedCity = ref<City | null>(null)
const showCityDropdown = ref(false)

const cities = ref<City[]>([])
const submitted = ref(false)
const isLoadingCities = ref(true)
const isCalculating = ref(false)
const errorMessage = ref('')
const cityLoadError = ref('')

const chartResult = ref<any>(null)
const utcResult = ref('')

let engine: Engine | null = null

// 生成固定位置的星点，避免每次渲染时随机跳动
const stars: Star[] = Array.from({ length: 115 }, (_, index) => ({
  id: index,
  x: (index * 37.13 + 7.7) % 100,
  y: (index * 61.71 + 13.2) % 100,
  size: index % 13 === 0 ? 3 : index % 5 === 0 ? 2 : 1,
  delay: (index % 17) * 0.37,
  opacity: 0.3 + (index % 7) * 0.1,
}))

const zodiacSigns = [
  'Aries',
  'Taurus',
  'Gemini',
  'Cancer',
  'Leo',
  'Virgo',
  'Libra',
  'Scorpio',
  'Sagittarius',
  'Capricorn',
  'Aquarius',
  'Pisces',
]

const zodiacSymbols: Record<string, string> = {
  Aries: '♈',
  Taurus: '♉',
  Gemini: '♊',
  Cancer: '♋',
  Leo: '♌',
  Virgo: '♍',
  Libra: '♎',
  Scorpio: '♏',
  Sagittarius: '♐',
  Capricorn: '♑',
  Aquarius: '♒',
  Pisces: '♓',
}

const planetSymbols: Record<string, string> = {
  Sun: '☉',
  Moon: '☾',
  Mercury: '☿',
  Venus: '♀',
  Mars: '♂',
  Jupiter: '♃',
  Saturn: '♄',
  Uranus: '♅',
  Neptune: '♆',
  Pluto: '♇',
  Ascendant: 'ASC',
  Midheaven: 'MC',
}

const planetColors: Record<string, string> = {
  Sun: '#F5CB79',
  Moon: '#C9D9FF',
  Mercury: '#A8D9DB',
  Venus: '#F1AFCB',
  Mars: '#F08B83',
  Jupiter: '#D9B5F4',
  Saturn: '#B7A6D8',
  Uranus: '#91D6E8',
  Neptune: '#9FAEFF',
  Pluto: '#D49AC8',
  Ascendant: '#F5DCA0',
  Midheaven: '#F5DCA0',
}

const filteredCities = computed(() => {
  if (!showCityDropdown.value) return []

  const keyword = citySearch.value.trim().toLowerCase()

  const results = keyword
    ? cities.value.filter(city =>
        city.name.toLowerCase().includes(keyword) ||
        city.name_en.toLowerCase().includes(keyword)
      )
    : cities.value

  return [...results]
    .sort((a, b) =>
      b.population - a.population ||
      a.name.localeCompare(b.name, 'zh-CN')
    )
    .slice(0, 10)
})

const canSubmit = computed(() =>
  Boolean(
    birthDate.value &&
    birthTime.value &&
    selectedCity.value
  ) &&
  !isCalculating.value
)

const formattedDate = computed(() => {
  if (!birthDate.value) return ''

  return birthDate.value
    .replace(/^(\d{4})-(\d{2})-(\d{2})$/, '$1年$2月$3日')
})

const planets = computed<Planet[]>(() =>
  chartResult.value?.planets ?? []
)

const houses = computed<House[]>(() =>
  chartResult.value?.houses ?? []
)

function getPlanet(name: string) {
  return planets.value.find(planet => planet.name === name)
}

const sun = computed(() => getPlanet('Sun'))
const moon = computed(() => getPlanet('Moon'))
const ascendant = computed(() => getPlanet('Ascendant'))
const midheaven = computed(() => getPlanet('Midheaven'))

const mainPlanets = computed(() =>
  planets.value.filter(planet =>
    [
      'Mercury',
      'Venus',
      'Mars',
      'Jupiter',
      'Saturn',
      'Uranus',
      'Neptune',
      'Pluto',
    ].includes(planet.name)
  )
)

const wheelPlanets = computed(() =>
  planets.value
    .filter(planet =>
      [
        'Sun',
        'Moon',
        'Mercury',
        'Venus',
        'Mars',
        'Jupiter',
        'Saturn',
        'Uranus',
        'Neptune',
        'Pluto',
      ].includes(planet.name)
    )
    .map((planet, index) => {
      const point = planetPoint(planet, index)

      return {
        ...point,
        name: planet.name,
        symbol: planetSymbols[planet.name] ?? '✦',
        color: planetColors[planet.name] ?? '#E8D6A9',
        sign: planet.sign,
      }
    })
)

const chineseNames: Record<string, string> = {
  Sun: '太阳',
  Moon: '月亮',
  Mercury: '水星',
  Venus: '金星',
  Mars: '火星',
  Jupiter: '木星',
  Saturn: '土星',
  Uranus: '天王星',
  Neptune: '海王星',
  Pluto: '冥王星',
  Ascendant: '上升点',
  Midheaven: '天顶',
}

const signNames: Record<string, string> = {
  Aries: '白羊座',
  Taurus: '金牛座',
  Gemini: '双子座',
  Cancer: '巨蟹座',
  Leo: '狮子座',
  Virgo: '处女座',
  Libra: '天秤座',
  Scorpio: '天蝎座',
  Sagittarius: '射手座',
  Capricorn: '摩羯座',
  Aquarius: '水瓶座',
  Pisces: '双鱼座',
}

const signElements: Record<string, string> = {
  Aries: '火象',
  Leo: '火象',
  Sagittarius: '火象',
  Taurus: '土象',
  Virgo: '土象',
  Capricorn: '土象',
  Gemini: '风象',
  Libra: '风象',
  Aquarius: '风象',
  Cancer: '水象',
  Scorpio: '水象',
  Pisces: '水象',
}

const elementStats = computed(() => {
  const counts: Record<string, number> = {
    火象: 0,
    土象: 0,
    风象: 0,
    水象: 0,
  }

  mainPlanets.value.forEach(planet => {
    const element = signElements[planet.sign]
    if (element) counts[element]++
  })

  return Object.entries(counts).map(([name, count]) => ({
    name,
    count,
    percent: mainPlanets.value.length
      ? Math.round(count / mainPlanets.value.length * 100)
      : 0,
  }))
})

function signName(sign: string) {
  return signNames[sign] ?? sign
}

function planetName(name: string) {
  return chineseNames[name] ?? name
}

function signSymbol(sign: string) {
  return zodiacSymbols[sign] ?? '✦'
}

// 将星座、度数转换为黄道经度
function zodiacLongitude(sign: string, degree: number, minute: number) {
  const index = zodiacSigns.indexOf(sign)
  if (index < 0) return 0
  return index * 30 + degree + minute / 60
}

// 星盘 SVG 坐标计算
function polarPoint(longitude: number, radius: number) {
  const angle = (longitude - 90) * Math.PI / 180

  return {
    x: 300 + Math.cos(angle) * radius,
    y: 300 + Math.sin(angle) * radius,
  }
}

function zodiacPoint(index: number, radius: number) {
  return polarPoint(index * 30 + 15, radius)
}

function planetPoint(planet: Planet, index: number) {
  const longitude = zodiacLongitude(
    planet.sign,
    planet.degree,
    planet.minute
  )

  // 轻微错开半径，减少相近行星标记重叠
  const radius = 151 + (index % 3) * 13
  return polarPoint(longitude, radius)
}

function housePoint(house: House, radius: number) {
  return polarPoint(
    zodiacLongitude(house.sign, house.degree, house.minute),
    radius
  )
}

function selectCity(city: City) {
  selectedCity.value = city
  citySearch.value = city.name
  showCityDropdown.value = false
  errorMessage.value = ''
}

function clearCity() {
  selectedCity.value = null
  citySearch.value = ''
  showCityDropdown.value = true
  errorMessage.value = ''
}

function closeCityDropdown() {
  showCityDropdown.value = false
}

async function loadCities() {
  try {
    const response = await fetch(
      `${import.meta.env.BASE_URL}data/china_cities.json`
    )

    if (!response.ok) {
      throw new Error('城市数据文件读取失败')
    }

    const data = await response.json() as Array<Record<string, unknown>>

    if (!Array.isArray(data)) {
      throw new Error('城市数据格式不正确')
    }

    cities.value = data.map(item => ({
      name: String(item.name ?? ''),
      name_en: String(item.name_en ?? ''),
      lat: Number(item.lat),
      lon: Number(item.lon),
      population: Number(item.population) || 0,
    })).filter(city =>
      city.name &&
      Number.isFinite(city.lat) &&
      Number.isFinite(city.lon) &&
      city.lat >= -90 &&
      city.lat <= 90 &&
      city.lon >= -180 &&
      city.lon <= 180
    )

    if (!cities.value.length) {
      throw new Error('城市数据为空或格式不正确')
    }
  } catch (error) {
    cityLoadError.value =
      error instanceof Error ? error.message : '城市数据加载失败'
  } finally {
    isLoadingCities.value = false
  }
}

async function initializeEngine() {
  if (engine) return engine

  const wasmResponse = await fetch(wasmUrl)

  if (!wasmResponse.ok) {
    throw new Error('星盘计算模块加载失败')
  }

  const wasmBytes = new Uint8Array(
    await wasmResponse.arrayBuffer()
  )

  await init({
    module_or_path: wasmBytes,
  })

  const kernelResponse = await fetch(
    `${import.meta.env.BASE_URL}ephemeris/de440s-1900-2100.bsp`
  )

  if (!kernelResponse.ok) {
    throw new Error('星历文件加载失败，请检查本地星历文件')
  }

  const kernel = new Uint8Array(
    await kernelResponse.arrayBuffer()
  )

  engine = new Engine()

  engine.addKernel(
    'de440s-1900-2100.bsp',
    kernel
  )

  return engine
}

function convertToUTC() {
  const localDateTime = Temporal.PlainDateTime.from(
    `${birthDate.value}T${birthTime.value}`
  )

  const zonedDateTime = localDateTime.toZonedDateTime(
    'Asia/Shanghai',
    { disambiguation: 'reject' }
  )

  return zonedDateTime.toInstant().toString()
}

async function generateChart() {
  if (!canSubmit.value || !selectedCity.value) return

  isCalculating.value = true
  errorMessage.value = ''
  chartResult.value = null

  try {
    const utc = convertToUTC()
    utcResult.value = utc

    const chartEngine = await initializeEngine()
    const jd = julianDay(utc)

    if (!chartEngine.supports(jd)) {
      throw new Error(
        '该出生日期超出当前星历文件的支持范围，请选择1900年至2100年之间的日期。'
      )
    }

    const chart = chartEngine.chart({
      utc,
      latitude: selectedCity.value.lat,
      longitude: selectedCity.value.lon,
      house_system: 'P',
      zodiac_type: 'tropical',
    })

    if (
      !chart.planets ||
      !chart.houses ||
      !chart.planets.length ||
      chart.houses.length !== 12
    ) {
      throw new Error('星盘计算结果不完整，请稍后重试。')
    }

    chartResult.value = chart
    submitted.value = true
  } catch (error) {
    errorMessage.value =
      error instanceof RangeError
        ? '出生时间存在夏令时或时间歧义，请核对出生记录。'
        : error instanceof Error
          ? error.message
          : '星盘计算失败，请稍后重试。'
  } finally {
    isCalculating.value = false
  }
}

function editInformation() {
  submitted.value = false
  chartResult.value = null
  errorMessage.value = ''
}

onMounted(() => {
  loadCities()
})

onBeforeUnmount(() => {
  if (engine) {
    engine.free()
    engine = null
  }
})
</script>

<template>
  <div class="birth-chart-page">
    <!-- 星空背景 -->
    <div class="cosmos-background" aria-hidden="true">
      <div class="nebula nebula-one"></div>
      <div class="nebula nebula-two"></div>
      <div class="nebula nebula-three"></div>

      <span
        v-for="star in stars"
        :key="star.id"
        class="background-star"
        :style="{
          left: `${star.x}%`,
          top: `${star.y}%`,
          width: `${star.size}px`,
          height: `${star.size}px`,
          opacity: star.opacity,
          animationDelay: `${star.delay}s`
        }"
      ></span>

      <div class="cosmos-grid"></div>
    </div>

    <div class="page-content">
      <header class="cosmic-header">
        <div class="brand-mark">
          <span class="brand-icon">✧</span>
          <span>ASTRO · CELESTIAL ATLAS</span>
        </div>

        <div class="header-line"></div>

        <div class="header-caption">
          <span>星 屿</span>
          <span class="caption-dot">✦</span>
          <span>探索属于你的宇宙</span>
        </div>
      </header>

      <section class="chart-intro">
        <div class="intro-orbit">
          <span class="orbit-star">✦</span>
          <span class="orbit-small">✧</span>
        </div>

        <span class="chart-eyebrow">YOUR PERSONAL UNIVERSE</span>
        <h1>每个人都是一片独特的星空</h1>
        <p>在浩瀚宇宙中，寻找你出生那一刻的天体印记。</p>

        <div class="intro-decoration">
          <span></span>
          <i>✧</i>
          <span></span>
        </div>
      </section>

      <section v-if="!submitted" class="birth-form-card">
        <div class="section-topline">
          <span>01 / BIRTH INFORMATION</span>
          <span>出生资料</span>
        </div>

        <div class="form-heading">
          <div class="form-symbol">☼</div>
          <div>
            <h2>开启你的星图</h2>
            <p>准确的出生资料，是绘制本命星盘的基础。</p>
          </div>
        </div>

        <div class="form-fields">
          <label class="field">
            <span class="field-label">
              <i>✧</i>
              出生日期
            </span>
            <input
              v-model="birthDate"
              type="date"
              min="1900-01-01"
              max="2100-12-31"
            />
          </label>

          <label class="field">
            <span class="field-label">
              <i>☾</i>
              出生时间
            </span>
            <input v-model="birthTime" type="time" />
            <small>请尽量填写出生证明或家人记录的准确时间。</small>
          </label>

          <div class="field city-field">
            <span class="field-label">
              <i>⌖</i>
              出生城市
            </span>

            <div class="city-input-wrap">
              <input
                :value="citySearch"
                type="text"
                placeholder="搜索你的出生城市..."
                :disabled="isLoadingCities || !!cityLoadError"
                @focus="showCityDropdown = true"
                @click="showCityDropdown = true"
                @input="
                  citySearch = ($event.target as HTMLInputElement).value;
                  selectedCity = null;
                  showCityDropdown = true
                "
                @blur="closeCityDropdown"
              />

              <button
                v-if="selectedCity"
                class="clear-city"
                type="button"
                @mousedown.prevent
                @click="clearCity"
              >
                ×
              </button>

              <span v-else class="dropdown-arrow">⌄</span>
            </div>

            <small v-if="isLoadingCities">正在载入中国城市星图数据...</small>

            <small v-else-if="cityLoadError" class="error-text">
              {{ cityLoadError }}
            </small>

            <small v-else-if="selectedCity" class="selected-city-note">
              <span class="status-dot"></span>
              已定位：{{ selectedCity.name }} ·
              {{ selectedCity.lat }}° N，
              {{ selectedCity.lon }}° E
            </small>

            <small v-else>输入城市名称，或点击展开常用城市。</small>

            <div v-if="filteredCities.length" class="city-results">
              <button
                v-for="city in filteredCities"
                :key="city.name_en"
                type="button"
                class="city-option"
                @mousedown.prevent="selectCity(city)"
              >
                <span class="city-name">{{ city.name }}</span>
                <small>{{ city.name_en }}</small>
              </button>
            </div>

            <div
              v-else-if="showCityDropdown && citySearch.trim() && !selectedCity && !isLoadingCities"
              class="city-empty"
            >
              没有找到匹配的城市，请检查名称。
            </div>
          </div>
        </div>

        <div class="timezone-note">
          <span class="note-icon">◷</span>
          <div>
            <strong>时间标准 · 中国标准时间</strong>
            <p>系统将根据出生日期自动处理历史时区规则，并转换为星历计算所需的 UTC 时间。</p>
          </div>
        </div>

        <p v-if="errorMessage" class="error-message">
          {{ errorMessage }}
        </p>

        <button
          class="generate-button"
          :disabled="!canSubmit || isLoadingCities || !!cityLoadError"
          @click="generateChart"
        >
          <span v-if="isCalculating" class="loading-orbit">✧</span>
          <span>{{ isCalculating ? '正在计算你的星空...' : '绘制我的本命星盘' }}</span>
          <span v-if="!isCalculating">→</span>
        </button>

        <div class="privacy-note">
          <span>✦</span>
          <p>出生资料仅在当前浏览器中参与计算，不会主动上传到服务器。</p>
        </div>
      </section>

      <section v-else class="chart-result">
        <div class="result-heading">
          <span class="chart-eyebrow">YOUR NATAL CHART</span>
          <h2>你的宇宙星图</h2>
          <p>这是你出生那一刻，天空中天体位置的计算结果。</p>
        </div>

        <div class="birth-summary">
          <div>
            <span>DATE</span>
            <strong>{{ formattedDate }}</strong>
          </div>
          <div>
            <span>LOCAL TIME</span>
            <strong>{{ birthTime }}</strong>
          </div>
          <div>
            <span>LOCATION</span>
            <strong>{{ selectedCity?.name }}</strong>
          </div>
        </div>

        <!-- 动态本命星盘 -->
        <section class="wheel-section">
          <div class="module-heading">
            <div>
              <span class="module-index">01 / CELESTIAL MAP</span>
              <h3>你的本命星盘</h3>
            </div>
            <span class="module-decoration">✧ ✦ ✧</span>
          </div>

          <div class="wheel-container">
            <div class="wheel-glow"></div>

            <svg
              class="natal-wheel"
              viewBox="0 0 600 600"
              role="img"
              aria-label="根据出生资料计算的本命星盘示意图"
            >
              <defs>
                <radialGradient id="wheelCenterGlow">
                  <stop offset="0%" stop-color="#B9A0F5" stop-opacity=".22"/>
                  <stop offset="100%" stop-color="#B9A0F5" stop-opacity="0"/>
                </radialGradient>
                <linearGradient id="wheelStroke" x1="0" y1="0" x2="1" y2="1">
                  <stop offset="0%" stop-color="#D9C5FF" stop-opacity=".8"/>
                  <stop offset="100%" stop-color="#8CA8E8" stop-opacity=".35"/>
                </linearGradient>
              </defs>

              <circle cx="300" cy="300" r="286" fill="none" stroke="#C8B5F4" stroke-opacity=".12" stroke-width="1"/>
              <circle cx="300" cy="300" r="270" fill="none" stroke="#C8B5F4" stroke-opacity=".3" stroke-width="1"/>
              <circle cx="300" cy="300" r="226" fill="none" stroke="#C8B5F4" stroke-opacity=".45" stroke-width="1"/>
              <circle cx="300" cy="300" r="178" fill="none" stroke="#C8B5F4" stroke-opacity=".25" stroke-width="1"/>
              <circle cx="300" cy="300" r="112" fill="url(#wheelCenterGlow)" stroke="#C8B5F4" stroke-opacity=".25"/>

              <g v-for="index in 12" :key="`zodiac-line-${index}`">
                <line
                  :x1="polarPoint((index - 1) * 30, 178).x"
                  :y1="polarPoint((index - 1) * 30, 178).y"
                  :x2="polarPoint((index - 1) * 30, 270).x"
                  :y2="polarPoint((index - 1) * 30, 270).y"
                  stroke="#C8B5F4"
                  stroke-opacity=".35"
                  stroke-width="1"
                />
              </g>

              <g v-for="(sign, index) in zodiacSigns" :key="sign">
                <text
                  :x="zodiacPoint(index, 247).x"
                  :y="zodiacPoint(index, 247).y"
                  text-anchor="middle"
                  dominant-baseline="central"
                  class="zodiac-glyph"
                >
                  {{ zodiacSymbols[sign] }}
                </text>
              </g>

              <g v-for="house in houses" :key="`house-${house.house_number}`">
                <line
                  :x1="housePoint(house, 112).x"
                  :y1="housePoint(house, 112).y"
                  :x2="housePoint(house, 226).x"
                  :y2="housePoint(house, 226).y"
                  stroke="#A9B9E8"
                  stroke-opacity=".4"
                  stroke-width="1"
                />
                <text
                  :x="housePoint(house, 143).x"
                  :y="housePoint(house, 143).y"
                  text-anchor="middle"
                  dominant-baseline="central"
                  class="house-number"
                >
                  {{ house.house_number }}
                </text>
              </g>

              <g v-for="planet in wheelPlanets" :key="`planet-${planet.name}`">
                <line
                  x1="300"
                  y1="300"
                  :x2="planet.x"
                  :y2="planet.y"
                  :stroke="planet.color"
                  stroke-opacity=".17"
                  stroke-width="1"
                />
                <circle
                  :cx="planet.x"
                  :cy="planet.y"
                  r="12"
                  :fill="planet.color"
                  fill-opacity=".12"
                  :stroke="planet.color"
                  stroke-opacity=".75"
                  stroke-width="1"
                />
                <text
                  :x="planet.x"
                  :y="planet.y + 1"
                  text-anchor="middle"
                  dominant-baseline="central"
                  :fill="planet.color"
                  class="planet-glyph"
                >
                  {{ planet.symbol }}
                </text>
              </g>

              <circle cx="300" cy="300" r="5" fill="#E8D6A9"/>
              <circle cx="300" cy="300" r="19" fill="none" stroke="#E8D6A9" stroke-opacity=".45"/>
              <circle cx="300" cy="300" r="35" fill="none" stroke="#C8B5F4" stroke-opacity=".2" stroke-dasharray="2 7"/>

              <text x="300" y="283" text-anchor="middle" class="wheel-center-label">NATAL</text>
              <text x="300" y="322" text-anchor="middle" class="wheel-center-sub">CHART</text>
            </svg>

            <div class="wheel-caption">
              <span>✦</span>
              <span>十二星座 · 十二宫位 · 行星落点</span>
              <span>✦</span>
            </div>
          </div>

          <div class="wheel-legend">
            <span><i class="legend-dot sun-dot"></i>太阳</span>
            <span><i class="legend-dot moon-dot"></i>月亮</span>
            <span><i class="legend-dot planet-dot"></i>其他行星</span>
          </div>
          <p class="wheel-disclaimer">
            星盘图依据星座黄道经度绘制，用于展示星体分布；当前为简化展示图，并非完整的专业相位盘。
          </p>
        </section>

        <!-- 核心星体 -->
        <section class="core-section">
          <div class="module-heading">
            <div>
              <span class="module-index">02 / THE BIG FOUR</span>
              <h3>你的核心星体</h3>
            </div>
            <span class="module-decoration">✧ ✦ ✧</span>
          </div>

          <div class="core-grid">
            <article class="core-card sun-card">
              <div class="core-card-top">
                <span class="core-symbol">☉</span>
                <span class="core-tag">IDENTITY</span>
              </div>
              <small>太阳 · 核心自我</small>
              <strong>{{ sun ? signName(sun.sign) : '—' }}</strong>
              <span v-if="sun" class="core-degree">
                {{ sun.degree }}°{{ String(sun.minute).padStart(2, '0') }}′
              </span>
            </article>

            <article class="core-card moon-card">
              <div class="core-card-top">
                <span class="core-symbol">☾</span>
                <span class="core-tag">EMOTION</span>
              </div>
              <small>月亮 · 内在情绪</small>
              <strong>{{ moon ? signName(moon.sign) : '—' }}</strong>
              <span v-if="moon" class="core-degree">
                {{ moon.degree }}°{{ String(moon.minute).padStart(2, '0') }}′
              </span>
            </article>

            <article class="core-card asc-card">
              <div class="core-card-top">
                <span class="core-symbol">↗</span>
                <span class="core-tag">ASCENDANT</span>
              </div>
              <small>上升 · 外在表达</small>
              <strong>{{ ascendant ? signName(ascendant.sign) : '—' }}</strong>
              <span v-if="ascendant" class="core-degree">
                {{ ascendant.degree }}°{{ String(ascendant.minute).padStart(2, '0') }}′
              </span>
            </article>

            <article class="core-card mc-card">
              <div class="core-card-top">
                <span class="core-symbol">✧</span>
                <span class="core-tag">MIDHEAVEN</span>
              </div>
              <small>天顶 · 事业方向</small>
              <strong>{{ midheaven ? signName(midheaven.sign) : '—' }}</strong>
              <span v-if="midheaven" class="core-degree">
                {{ midheaven.degree }}°{{ String(midheaven.minute).padStart(2, '0') }}′
              </span>
            </article>
          </div>
        </section>

        <!-- 行星分布 -->
        <section class="data-section">
          <div class="module-heading">
            <div>
              <span class="module-index">03 / PLANETARY POSITIONS</span>
              <h3>行星落座档案</h3>
            </div>
            <span class="module-decoration">✦</span>
          </div>

          <div class="planet-list">
            <div
              v-for="planet in mainPlanets"
              :key="planet.name"
              class="planet-row"
            >
              <span
                class="planet-row-symbol"
                :style="{ color: planetColors[planet.name] }"
              >
                {{ planetSymbols[planet.name] }}
              </span>
              <span class="planet-row-name">{{ planetName(planet.name) }}</span>
              <span class="planet-row-sign">
                {{ signSymbol(planet.sign) }} {{ signName(planet.sign) }}
              </span>
              <span class="planet-row-degree">
                {{ planet.degree }}°{{ String(planet.minute).padStart(2, '0') }}′
              </span>
              <span class="planet-row-house">第{{ planet.house }}宫</span>
            </div>
          </div>
        </section>

        <!-- 元素比例 -->
        <section class="element-section">
          <div class="module-heading">
            <div>
              <span class="module-index">04 / ELEMENT BALANCE</span>
              <h3>四元素分布</h3>
            </div>
            <span class="module-decoration">✧</span>
          </div>

          <div class="element-grid">
            <div
              v-for="element in elementStats"
              :key="element.name"
              class="element-card"
            >
              <div class="element-top">
                <span>{{ element.name }}</span>
                <strong>{{ element.percent }}%</strong>
              </div>
              <div class="element-track">
                <div
                  class="element-fill"
                  :style="{ width: `${element.percent}%` }"
                ></div>
              </div>
              <small>{{ element.count }} 颗主要行星</small>
            </div>
          </div>
          <p class="element-note">
            统计基于太阳、月亮及八大行星的星座元素归属，不代表人格或能力的科学测量。
          </p>
        </section>

        <!-- 十二宫 -->
        <section class="data-section">
          <div class="module-heading">
            <div>
              <span class="module-index">05 / THE TWELVE HOUSES</span>
              <h3>十二宫位分布</h3>
            </div>
            <span class="module-decoration">✦ ✧</span>
          </div>

          <div class="house-grid">
            <div
              v-for="house in houses"
              :key="house.house_number"
              class="house-card"
            >
              <span class="house-card-number">HOUSE {{ String(house.house_number).padStart(2, '0') }}</span>
              <strong>
                {{ signSymbol(house.sign) }}
                {{ signName(house.sign) }}
              </strong>
              <small>
                {{ house.degree }}°{{ String(house.minute).padStart(2, '0') }}′
              </small>
            </div>
          </div>
        </section>

        <!-- 计算参数 -->
        <section class="calculation-info">
          <div class="module-heading">
            <div>
              <span class="module-index">06 / CALCULATION DATA</span>
              <h3>星历计算参数</h3>
            </div>
            <span class="module-decoration">⌘</span>
          </div>

          <div class="calculation-grid">
            <div>
              <span>黄道系统</span>
              <strong>热带黄道 Tropical</strong>
            </div>
            <div>
              <span>宫位系统</span>
              <strong>Placidus 普拉西德</strong>
            </div>
            <div>
              <span>时间标准</span>
              <strong>Asia / Shanghai</strong>
            </div>
            <div>
              <span>星历范围</span>
              <strong>1900 — 2100</strong>
            </div>
          </div>

          <div class="utc-line">
            <span>UTC / UNIVERSAL TIME</span>
            <strong>{{ utcResult }}</strong>
          </div>
        </section>

        <button class="edit-button" @click="editInformation">
          <span>↶</span>
          修改出生资料 · 重新绘制星图
        </button>
      </section>

      <section class="chart-explanation">
        <div class="explanation-heading">
          <span class="chart-eyebrow">THE LANGUAGE OF THE STARS</span>
          <h3>读懂星空的语言</h3>
          <p>从天体位置，认识占星体系中的不同象征。</p>
        </div>

        <div class="explanation-grid">
          <article>
            <span>☉</span>
            <strong>太阳星座</strong>
            <p>象征自我意识、核心特质与生命方向。</p>
          </article>
          <article>
            <span>☾</span>
            <strong>月亮星座</strong>
            <p>象征情绪需求、内在感受与安全感。</p>
          </article>
          <article>
            <span>↗</span>
            <strong>上升星座</strong>
            <p>关联外在表现、第一印象与面对世界的方式。</p>
          </article>
          <article>
            <span>✧</span>
            <strong>行星与宫位</strong>
            <p>展示不同天体在星座和人生领域中的位置。</p>
          </article>
        </div>
      </section>

      <footer class="cosmic-footer">
        <div class="footer-ornament">✧ ───── ✦ ───── ✧</div>
        <p>THE UNIVERSE WITHIN YOU</p>
        <span>星空辽阔，探索仍在继续。</span>
      </footer>

      <p class="disclaimer">
        星盘属于占星文化体系，相关解释仅供兴趣探索与娱乐参考，不属于经过科学验证的人格评估或未来预测。
      </p>
    </div>
  </div>
</template>

<style scoped>
.birth-chart-page {
  --space-deep: #070b1b;
  --space-blue: #101a35;
  --space-purple: #8e78c9;
  --space-gold: #e5c995;
  --space-text: #f2edff;
  --space-muted: #a6a7c4;

  position: relative;
  width: 100%;
  min-height: 100vh;
  overflow: hidden;
  isolation: isolate;
  color: var(--space-text);
  background: var(--space-deep);
}

.cosmos-background {
  position: absolute;
  inset: 0;
  z-index: -1;
  overflow: hidden;
  pointer-events: none;
  background:
    radial-gradient(ellipse at 48% 8%, rgba(68, 66, 132, .27), transparent 38%),
    radial-gradient(ellipse at 5% 45%, rgba(48, 75, 137, .18), transparent 38%),
    radial-gradient(ellipse at 90% 65%, rgba(98, 57, 135, .17), transparent 36%),
    linear-gradient(180deg, #080d20 0%, #0a1025 48%, #080b1b 100%);
}

.nebula {
  position: absolute;
  border-radius: 50%;
  filter: blur(65px);
  opacity: .24;
  animation: nebula-drift 24s ease-in-out infinite alternate;
}

.nebula-one {
  width: 520px;
  height: 360px;
  left: -180px;
  top: 180px;
  background: radial-gradient(ellipse, #5967bb, #393b83 45%, transparent 72%);
}

.nebula-two {
  width: 600px;
  height: 400px;
  right: -220px;
  top: 720px;
  background: radial-gradient(ellipse, #8957b9, #473d8b 48%, transparent 72%);
  animation-delay: -8s;
}

.nebula-three {
  width: 500px;
  height: 360px;
  left: 20%;
  top: 1600px;
  background: radial-gradient(ellipse, #385b9c, #343b7a 48%, transparent 72%);
  animation-delay: -14s;
}

.background-star {
  position: absolute;
  display: block;
  border-radius: 50%;
  background: #e7e6ff;
  box-shadow: 0 0 5px rgba(204, 210, 255, .7);
  animation: star-twinkle 4s ease-in-out infinite;
}

.cosmos-grid {
  position: absolute;
  inset: 0;
  opacity: .1;
  background-image:
    linear-gradient(rgba(170, 170, 220, .08) 1px, transparent 1px),
    linear-gradient(90deg, rgba(170, 170, 220, .08) 1px, transparent 1px);
  background-size: 90px 90px;
  mask-image: linear-gradient(to bottom, transparent, black 12%, black 80%, transparent);
}

.page-content {
  position: relative;
  z-index: 1;
  width: min(100% - 36px, 820px);
  margin: 0 auto;
  padding: 28px 0 90px;
}

.cosmic-header {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  margin-bottom: 56px;
}

.brand-mark {
  display: flex;
  align-items: center;
  gap: 9px;
  color: #d8c8f2;
  font-size: 10px;
  letter-spacing: 3px;
}

.brand-icon {
  color: var(--space-gold);
  font-size: 19px;
}

.header-line {
  width: min(100%, 260px);
  height: 1px;
  background: linear-gradient(90deg, transparent, rgba(218, 198, 255, .55), transparent);
}

.header-caption {
  display: flex;
  align-items: center;
  gap: 10px;
  color: #a8a6c6;
  font-size: 11px;
  letter-spacing: 2px;
}

.header-caption span:first-child {
  color: var(--space-gold);
  font-size: 15px;
}

.caption-dot {
  color: #b49ce7;
}

.chart-intro {
  position: relative;
  text-align: center;
  margin-bottom: 44px;
}

.intro-orbit {
  position: relative;
  width: 70px;
  height: 70px;
  margin: 0 auto 18px;
  display: grid;
  place-items: center;
  border: 1px solid rgba(211, 192, 255, .3);
  border-radius: 50%;
  box-shadow: 0 0 30px rgba(148, 123, 218, .14), inset 0 0 25px rgba(148, 123, 218, .1);
  animation: orbit-glow 5s ease-in-out infinite;
}

.intro-orbit::before,
.intro-orbit::after {
  content: "";
  position: absolute;
  inset: -8px;
  border: 1px solid rgba(211, 192, 255, .13);
  border-radius: 50%;
  transform: rotateX(68deg);
}

.intro-orbit::after {
  transform: rotateY(68deg);
}

.orbit-star {
  color: #f0d59f;
  font-size: 31px;
}

.orbit-small {
  position: absolute;
  top: 2px;
  right: 4px;
  color: #c4b3f4;
  font-size: 14px;
}

.chart-eyebrow {
  color: #d7bd91;
  font-size: 10px;
  letter-spacing: 4px;
}

.chart-intro h1 {
  margin: 16px 0 12px;
  color: #f3eaff;
  font-family: Georgia, "Noto Serif SC", serif;
  font-size: clamp(25px, 5vw, 38px);
  font-weight: 400;
  letter-spacing: 3px;
  text-shadow: 0 0 28px rgba(190, 165, 255, .28);
}

.chart-intro p {
  color: #aaa8c7;
  font-size: 13px;
  line-height: 1.9;
  letter-spacing: 1px;
}

.intro-decoration {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
  margin-top: 24px;
  color: var(--space-gold);
}

.intro-decoration span {
  width: 48px;
  height: 1px;
  background: linear-gradient(90deg, transparent, rgba(224, 200, 151, .7));
}

.intro-decoration span:last-child {
  transform: rotate(180deg);
}

.birth-form-card,
.chart-result {
  padding: 30px;
  border: 1px solid rgba(193, 174, 245, .24);
  border-radius: 22px;
  background:
    linear-gradient(145deg, rgba(34, 38, 78, .82), rgba(13, 19, 43, .91));
  box-shadow:
    0 20px 70px rgba(0, 0, 0, .2),
    inset 0 1px 0 rgba(255, 255, 255, .07);
  backdrop-filter: blur(16px);
}

.section-topline {
  display: flex;
  justify-content: space-between;
  padding-bottom: 15px;
  border-bottom: 1px solid rgba(198, 180, 244, .17);
  color: #a9a3ca;
  font-size: 9px;
  letter-spacing: 2px;
}

.section-topline span:last-child {
  color: #d7bd91;
  letter-spacing: 1px;
}

.form-heading {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-top: 24px;
}

.form-symbol {
  width: 54px;
  height: 54px;
  display: grid;
  place-items: center;
  border: 1px solid rgba(225, 198, 147, .4);
  border-radius: 50%;
  background: radial-gradient(circle, rgba(224, 190, 131, .18), rgba(224, 190, 131, .02));
  color: #e6c993;
  font-size: 27px;
  box-shadow: 0 0 24px rgba(225, 198, 147, .12);
}

.form-heading h2 {
  margin: 0 0 7px;
  font-size: 19px;
  font-weight: 500;
  letter-spacing: 1px;
}

.form-heading p {
  margin: 0;
  color: #a6a4c1;
  font-size: 11px;
}

.form-fields {
  display: grid;
  gap: 24px;
  margin: 30px 0;
}

.field {
  position: relative;
  display: grid;
  gap: 11px;
}

.field-label {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #e5dcf6;
  font-size: 12px;
  letter-spacing: 1px;
}

.field-label i {
  color: #e4c48e;
  font-size: 16px;
  font-style: normal;
}

.field input {
  width: 100%;
  box-sizing: border-box;
  padding: 15px;
  border: 1px solid rgba(192, 174, 237, .23);
  border-radius: 10px;
  outline: none;
  background: rgba(5, 10, 28, .45);
  color: #f6efff;
  font-size: 13px;
  color-scheme: dark;
  transition: border-color .25s, box-shadow .25s;
}

.field input:focus {
  border-color: rgba(211, 183, 245, .8);
  box-shadow: 0 0 0 3px rgba(164, 132, 222, .1);
}

.field input::placeholder {
  color: #777993;
}

.field small {
  color: #9395b2;
  font-size: 10px;
  line-height: 1.7;
}

.city-input-wrap {
  position: relative;
}

.city-input-wrap input {
  padding-right: 42px;
}

.clear-city {
  position: absolute;
  top: 50%;
  right: 12px;
  transform: translateY(-50%);
  border: 0;
  background: transparent;
  color: #e2c18e;
  font-size: 22px;
  cursor: pointer;
}

.dropdown-arrow {
  position: absolute;
  top: 50%;
  right: 14px;
  transform: translateY(-50%);
  color: #d7bd91;
  font-size: 20px;
  pointer-events: none;
}

.city-results {
  position: absolute;
  z-index: 5;
  top: calc(100% + 6px);
  left: 0;
  right: 0;
  max-height: 260px;
  overflow-y: auto;
  border: 1px solid rgba(193, 174, 245, .3);
  border-radius: 12px;
  background: #141a36;
  box-shadow: 0 18px 38px rgba(0, 0, 0, .4);
}

.city-option {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  padding: 13px 15px;
  border: 0;
  border-bottom: 1px solid rgba(193, 174, 245, .1);
  background: transparent;
  color: #f1eaff;
  text-align: left;
  cursor: pointer;
}

.city-option:hover {
  background: rgba(170, 146, 224, .13);
}

.city-option small {
  color: #9a9bb8;
}

.city-name {
  font-size: 12px;
}

.city-empty {
  padding: 13px;
  border-radius: 9px;
  background: rgba(255, 255, 255, .04);
  color: #aaa8c7;
  font-size: 11px;
}

.selected-city-note {
  color: #dfc48f !important;
}

.status-dot {
  display: inline-block;
  width: 6px;
  height: 6px;
  margin-right: 5px;
  border-radius: 50%;
  background: #a8d6c1;
  box-shadow: 0 0 8px #a8d6c1;
}

.timezone-note,
.privacy-note {
  display: flex;
  align-items: flex-start;
  gap: 13px;
  margin-top: 21px;
  padding: 16px;
  border: 1px solid rgba(192, 174, 237, .13);
  border-radius: 12px;
  background: rgba(7, 12, 31, .35);
}

.note-icon {
  color: #e4c48e;
  font-size: 19px;
}

.timezone-note strong {
  color: #ddc596;
  font-size: 11px;
  font-weight: 500;
}

.timezone-note p {
  margin: 7px 0 0;
  color: #999bb8;
  font-size: 10px;
  line-height: 1.8;
}

.privacy-note {
  align-items: center;
  justify-content: center;
  border: 0;
  background: transparent;
}

.privacy-note span {
  color: #d8bd8c;
}

.privacy-note p {
  margin: 0;
  color: #8f91ad;
  font-size: 10px;
}

.generate-button {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 13px;
  width: 100%;
  margin-top: 25px;
  padding: 16px;
  border: 1px solid rgba(245, 220, 176, .6);
  border-radius: 11px;
  background: linear-gradient(105deg, #b9a1df, #e5c995);
  color: #17172e;
  font-size: 13px;
  font-weight: 700;
  letter-spacing: 1px;
  cursor: pointer;
  box-shadow: 0 8px 28px rgba(177, 146, 221, .18);
  transition: transform .2s, box-shadow .2s, opacity .2s;
}

.generate-button:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 12px 35px rgba(177, 146, 221, .3);
}

.generate-button:disabled {
  opacity: .42;
  cursor: not-allowed;
}

.loading-orbit {
  display: inline-block;
  animation: rotate-star 1.5s linear infinite;
}

.error-message,
.error-text {
  color: #ffaaa8 !important;
}

.error-message {
  margin: 15px 0;
  font-size: 12px;
  line-height: 1.8;
}

/* 结果页 */

.result-heading {
  text-align: center;
  margin-bottom: 28px;
}

.result-heading h2 {
  margin: 13px 0 9px;
  font-family: Georgia, "Noto Serif SC", serif;
  font-size: 27px;
  font-weight: 400;
  letter-spacing: 3px;
}

.result-heading p {
  color: #a7a5c3;
  font-size: 11px;
  line-height: 1.8;
}

.birth-summary {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
  padding: 17px 0;
  border-top: 1px solid rgba(195, 177, 241, .18);
  border-bottom: 1px solid rgba(195, 177, 241, .18);
}

.birth-summary div {
  display: flex;
  flex-direction: column;
  gap: 8px;
  min-width: 0;
}

.birth-summary span {
  color: #999bb8;
  font-size: 9px;
  letter-spacing: 1px;
}

.birth-summary strong {
  color: #e8ddf7;
  font-size: 11px;
  font-weight: 500;
  overflow-wrap: anywhere;
}

.module-heading {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  gap: 10px;
  margin-bottom: 20px;
}

.module-index {
  color: #c7ad8a;
  font-size: 9px;
  letter-spacing: 2px;
}

.module-heading h3 {
  margin: 8px 0 0;
  color: #f0e8ff;
  font-size: 17px;
  font-weight: 500;
  letter-spacing: 1px;
}

.module-decoration {
  color: #d8bd91;
  font-size: 15px;
  letter-spacing: 5px;
}

/* 星盘轮盘 */

.wheel-section {
  margin-top: 34px;
  padding: 22px 14px 18px;
  border: 1px solid rgba(193, 174, 245, .16);
  border-radius: 18px;
  background:
    radial-gradient(ellipse at center, rgba(92, 76, 146, .18), transparent 67%),
    rgba(5, 10, 28, .3);
  overflow: hidden;
}

.wheel-container {
  position: relative;
  width: min(100%, 540px);
  margin: 0 auto;
}

.wheel-glow {
  position: absolute;
  inset: 15%;
  border-radius: 50%;
  background: radial-gradient(circle, rgba(126, 102, 201, .2), transparent 68%);
  filter: blur(25px);
}

.natal-wheel {
  position: relative;
  display: block;
  width: 100%;
  height: auto;
  overflow: visible;
  animation: wheel-appear 1.2s ease both;
}

.zodiac-glyph {
  fill: #e3d3fa;
  font-size: 23px;
  filter: drop-shadow(0 0 5px rgba(187, 163, 245, .55));
}

.house-number {
  fill: #a9a9ca;
  font-size: 11px;
}

.planet-glyph {
  font-size: 13px;
  font-weight: 600;
}

.wheel-center-label {
  fill: #e4d0a4;
  font-size: 12px;
  letter-spacing: 3px;
}

.wheel-center-sub {
  fill: #a9a5c6;
  font-size: 9px;
  letter-spacing: 3px;
}

.wheel-caption {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
  margin-top: 2px;
  color: #c5b6e1;
  font-size: 10px;
  letter-spacing: 1px;
}

.wheel-caption span {
  color: #dfc28e;
}

.wheel-legend {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 19px;
  margin-top: 20px;
}

.wheel-legend span {
  display: flex;
  align-items: center;
  gap: 6px;
  color: #a8a7c3;
  font-size: 10px;
}

.legend-dot {
  width: 7px;
  height: 7px;
  border-radius: 50%;
}

.sun-dot {
  background: #f5cb79;
  box-shadow: 0 0 8px #f5cb79;
}

.moon-dot {
  background: #c9d9ff;
  box-shadow: 0 0 8px #c9d9ff;
}

.planet-dot {
  background: #b69ce8;
  box-shadow: 0 0 8px #b69ce8;
}

.wheel-disclaimer {
  margin: 17px 5px 0;
  color: #81839f;
  font-size: 9px;
  line-height: 1.8;
  text-align: center;
}

/* 核心星体 */

.core-section {
  margin-top: 35px;
}

.core-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 12px;
}

.core-card {
  position: relative;
  min-height: 145px;
  padding: 17px;
  overflow: hidden;
  border: 1px solid rgba(193, 174, 245, .18);
  border-radius: 15px;
  background: linear-gradient(145deg, rgba(56, 52, 94, .58), rgba(14, 22, 48, .7));
}

.core-card::after {
  content: "";
  position: absolute;
  width: 90px;
  height: 90px;
  right: -30px;
  bottom: -40px;
  border-radius: 50%;
  background: radial-gradient(circle, rgba(193, 164, 244, .22), transparent 70%);
}

.core-card-top {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.core-symbol {
  color: #e7ca96;
  font-size: 25px;
}

.core-tag {
  color: #9393b4;
  font-size: 8px;
  letter-spacing: 1px;
}

.core-card > small {
  display: block;
  margin-top: 12px;
  color: #aaa8c5;
  font-size: 10px;
}

.core-card > strong {
  display: block;
  margin-top: 7px;
  color: #f2e9ff;
  font-size: 18px;
  font-weight: 500;
}

.core-degree {
  display: block;
  margin-top: 6px;
  color: #c7b8dc;
  font-size: 10px;
}

.moon-card .core-symbol {
  color: #c9d9ff;
}

.asc-card .core-symbol {
  color: #b6d9df;
}

.mc-card .core-symbol {
  color: #e6b9d9;
}

/* 行星列表 */

.data-section,
.element-section {
  margin-top: 36px;
}

.planet-list {
  overflow: hidden;
  border: 1px solid rgba(193, 174, 245, .17);
  border-radius: 13px;
  background: rgba(7, 12, 30, .3);
}

.planet-row {
  display: grid;
  grid-template-columns: 30px 1fr 1.2fr 1fr 65px;
  align-items: center;
  gap: 7px;
  min-height: 48px;
  padding: 9px 12px;
  border-bottom: 1px solid rgba(193, 174, 245, .1);
}

.planet-row:last-child {
  border-bottom: 0;
}

.planet-row-symbol {
  font-size: 18px;
  text-align: center;
}

.planet-row-name {
  color: #e9e1f7;
  font-size: 11px;
}

.planet-row-sign {
  color: #d5c3eb;
  font-size: 10px;
}

.planet-row-degree,
.planet-row-house {
  color: #a6a4c1;
  font-size: 10px;
}

.planet-row-house {
  text-align: right;
}

/* 四元素 */

.element-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 11px;
}

.element-card {
  padding: 15px;
  border: 1px solid rgba(193, 174, 245, .15);
  border-radius: 12px;
  background: rgba(7, 12, 30, .32);
}

.element-top {
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #ded5ef;
  font-size: 11px;
}

.element-top strong {
  color: #e4c995;
  font-size: 13px;
}

.element-track {
  height: 5px;
  margin: 12px 0 9px;
  overflow: hidden;
  border-radius: 8px;
  background: rgba(255, 255, 255, .09);
}

.element-fill {
  height: 100%;
  border-radius: inherit;
  background: linear-gradient(90deg, #8276bd, #e2c48f);
  box-shadow: 0 0 10px rgba(195, 166, 230, .35);
}

.element-card small {
  color: #9395b1;
  font-size: 9px;
}

.element-note {
  margin: 12px 2px 0;
  color: #82849f;
  font-size: 9px;
  line-height: 1.8;
}

/* 十二宫 */

.house-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 9px;
}

.house-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  min-width: 0;
  padding: 14px 5px;
  border: 1px solid rgba(193, 174, 245, .15);
  border-radius: 11px;
  background: rgba(7, 12, 30, .32);
}

.house-card-number {
  color: #9294b2;
  font-size: 8px;
  letter-spacing: 1px;
}

.house-card strong {
  color: #e5d4f3;
  font-size: 11px;
  font-weight: 500;
}

.house-card small {
  color: #b4a6c9;
  font-size: 9px;
}

/* 计算参数 */

.calculation-info {
  margin-top: 35px;
  padding: 19px;
  border: 1px solid rgba(193, 174, 245, .16);
  border-radius: 14px;
  background: rgba(7, 12, 30, .35);
}

.calculation-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 17px;
}

.calculation-grid div {
  display: flex;
  flex-direction: column;
  gap: 7px;
}

.calculation-grid span,
.utc-line span {
  color: #9695b3;
  font-size: 9px;
  letter-spacing: 1px;
}

.calculation-grid strong {
  color: #e4d8f1;
  font-size: 11px;
  font-weight: 500;
}

.utc-line {
  display: flex;
  flex-direction: column;
  gap: 7px;
  margin-top: 19px;
  padding-top: 15px;
  border-top: 1px solid rgba(193, 174, 245, .12);
}

.utc-line strong {
  color: #c4b7dc;
  font-size: 10px;
  font-weight: 400;
  overflow-wrap: anywhere;
}

.edit-button {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  width: 100%;
  margin-top: 23px;
  padding: 14px;
  border: 1px solid rgba(193, 174, 245, .3);
  border-radius: 10px;
  background: rgba(15, 22, 49, .55);
  color: #e6d9f6;
  font-size: 11px;
  letter-spacing: 1px;
  cursor: pointer;
  transition: background .2s, border-color .2s;
}

.edit-button:hover {
  border-color: rgba(225, 198, 147, .6);
  background: rgba(47, 41, 77, .7);
}

/* 说明区域 */

.chart-explanation {
  margin-top: 48px;
}

.explanation-heading {
  margin-bottom: 20px;
  text-align: center;
}

.explanation-heading h3 {
  margin: 12px 0 8px;
  color: #eee4fa;
  font-size: 21px;
  font-weight: 400;
  letter-spacing: 2px;
}

.explanation-heading p {
  color: #a3a1bf;
  font-size: 11px;
}

.explanation-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 12px;
}

.explanation-grid article {
  padding: 19px;
  border: 1px solid rgba(193, 174, 245, .15);
  border-radius: 14px;
  background: linear-gradient(145deg, rgba(36, 39, 77, .52), rgba(12, 18, 40, .7));
}

.explanation-grid article > span {
  display: block;
  margin-bottom: 13px;
  color: #e5c995;
  font-size: 25px;
}

.explanation-grid strong {
  color: #e9e0f6;
  font-size: 12px;
  font-weight: 500;
}

.explanation-grid p {
  margin: 9px 0 0;
  color: #a2a0bd;
  font-size: 10px;
  line-height: 1.9;
}

.cosmic-footer {
  margin-top: 54px;
  text-align: center;
}

.footer-ornament {
  color: #d9bd8c;
  font-size: 12px;
  letter-spacing: 3px;
}

.cosmic-footer p {
  margin: 16px 0 8px;
  color: #d8c5a0;
  font-size: 9px;
  letter-spacing: 3px;
}

.cosmic-footer > span {
  color: #9694b2;
  font-size: 10px;
  letter-spacing: 2px;
}

.disclaimer {
  margin: 35px auto 0;
  max-width: 650px;
  color: #777b99;
  font-size: 9px;
  line-height: 1.9;
  text-align: center;
}

@keyframes star-twinkle {
  0%, 100% {
    transform: scale(.75);
    filter: brightness(.7);
  }
  50% {
    transform: scale(1.5);
    filter: brightness(1.8);
  }
}

@keyframes nebula-drift {
  from {
    transform: translate3d(-15px, 0, 0) scale(1);
  }
  to {
    transform: translate3d(25px, 35px, 0) scale(1.15);
  }
}

@keyframes orbit-glow {
  0%, 100% {
    box-shadow: 0 0 22px rgba(148, 123, 218, .12), inset 0 0 20px rgba(148, 123, 218, .08);
  }
  50% {
    box-shadow: 0 0 38px rgba(148, 123, 218, .3), inset 0 0 30px rgba(148, 123, 218, .15);
  }
}

@keyframes rotate-star {
  to {
    transform: rotate(360deg);
  }
}

@keyframes wheel-appear {
  from {
    opacity: 0;
    transform: scale(.94);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}

@media (max-width: 600px) {
  .page-content {
    width: min(100% - 24px, 820px);
    padding-top: 23px;
  }

  .cosmic-header {
    margin-bottom: 40px;
  }

  .chart-intro {
    margin-bottom: 32px;
  }

  .chart-intro h1 {
    letter-spacing: 1px;
  }

  .birth-form-card,
  .chart-result {
    padding: 21px 15px;
    border-radius: 17px;
  }

  .birth-summary {
    gap: 7px;
  }

  .birth-summary strong {
    font-size: 10px;
  }

  .wheel-section {
    padding: 17px 7px;
  }

  .wheel-container {
    width: 100%;
  }

  .zodiac-glyph {
    font-size: 24px;
  }

  .core-card {
    min-height: 135px;
    padding: 14px;
  }

  .core-card > strong {
    font-size: 16px;
  }

  .planet-row {
    grid-template-columns: 23px 1fr 1.2fr .9fr 52px;
    gap: 4px;
    padding: 10px 7px;
  }

  .planet-row-symbol {
    font-size: 15px;
  }

  .planet-row-name,
  .planet-row-sign,
  .planet-row-degree,
  .planet-row-house {
    font-size: 9px;
  }

  .house-grid {
    gap: 7px;
  }

  .house-card {
    padding: 12px 3px;
  }

  .house-card strong {
    font-size: 10px;
  }

  .explanation-grid {
    gap: 9px;
  }

  .explanation-grid article {
    padding: 15px;
  }
}

@media (prefers-reduced-motion: reduce) {
  .background-star,
  .nebula,
  .intro-orbit,
  .loading-orbit,
  .natal-wheel {
    animation: none !important;
  }
}
</style>
