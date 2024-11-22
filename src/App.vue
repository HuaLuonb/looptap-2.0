<template>
  <div class="game-container">
    <div v-if="!developerMode" class="start-screen">
      <h1>Loop Tap</h1>
      <input 
        v-model="developerCode" 
        placeholder="开发者代码" 
        type="password"
        @keyup.enter="checkDeveloperCode"
      >
      <button @click="checkDeveloperCode">进入</button>
    </div>

    <div v-else class="game-screen">
      <div class="developer-panel">
        <h2>开发者模式</h2>
        <div>
          <label>最高分数:</label>
          <input type="number" v-model.number="customSettings.bestScore">
        </div>
        <div>
          <label>旋转速度:</label>
          <input type="number" v-model.number="customSettings.rotationSpeed" step="0.1">
        </div>
        <div>
          <label>球颜色:</label>
          <input type="color" v-model="customSettings.ballColor">
        </div>
        <div>
          <label>背景颜色:</label>
          <input type="color" v-model="customSettings.backgroundColor">
        </div>
        <div>
          <label>得分区域最小角度:</label>
          <input type="number" v-model.number="customSettings.minScoreAngle" min="0" max="360">
        </div>
        <div>
          <label>得分区域最大角度:</label>
          <input type="number" v-model.number="customSettings.maxScoreAngle" min="0" max="360">
        </div>
        <div>
          <label>球大小:</label>
          <input type="number" v-model.number="customSettings.ballSize" min="10" max="100">
        </div>
        <button @click="applySettings">应用设置</button>
      </div>

      <div class="game-main">
        <svg 
          ref="gameSvg" 
          width="500" 
          height="500"
          @mousedown="tap" 
          @touchstart.prevent="tap"
        >
          <path 
            :d="arcPath" 
            fill="none" 
            :stroke="gameSettings.backgroundColor" 
            stroke-width="20"
          />
          
          <circle 
            :cx="ballPosition.x" 
            :cy="ballPosition.y" 
            :r="gameSettings.ballSize" 
            :fill="gameSettings.ballColor"
          />
        </svg>

        <div class="game-info">
          <p>分数: {{ score }}</p>
          <p>最高分: {{ bestScore }}</p>
          <button @click="startGame">开始游戏</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'

// 状态管理
const developerCode = ref('')
const developerMode = ref(false)
const score = ref(0)
const bestScore = ref(0)
const gameState = ref('init')
const ballPosition = reactive({ x: 250, y: 250 })
const arcAngles = ref([0, 180])

// 游戏设置
const gameSettings = reactive({
  bestScore: 0,
  rotationSpeed: 1,
  ballColor: '#FF0000',
  backgroundColor: '#CCCCCC',
  minScoreAngle: 30,
  maxScoreAngle: 150,
  ballSize: 20
})

const customSettings = reactive({...gameSettings})

// 生命周期钩子
onMounted(() => {
  const storedBestScore = localStorage.getItem('bestScore')
  if (storedBestScore) {
    bestScore.value = Number(storedBestScore)
  }
  
  const storedSettings = localStorage.getItem('gameSettings')
  if (storedSettings) {
    Object.assign(gameSettings, JSON.parse(storedSettings))
  }
})

// 计算属性
const arcPath = computed(() => {
  return calculateArcPath(
    250, 250, 200, 
    arcAngles.value[0], 
    arcAngles.value[1]
  )
})

// 方法
function checkDeveloperCode() {
  if (developerCode.value === '54188') {
    developerMode.value = true
    Object.assign(customSettings, gameSettings)
  } else {
    alert('开发者代码错误')
  }
}

function applySettings() {
  Object.assign(gameSettings, customSettings)
  localStorage.setItem('gameSettings', JSON.stringify(gameSettings))
}

function startGame() {
  gameState.value = 'playing'
  score.value = 0
  updateBallPosition()
}

function tap(event) {
  if (gameState.value !== 'playing') return

  const angle = calculateTapAngle(event)
  if (isValidTap(angle)) {
    score.value++
    updateArcAngles()
  } else {
    endGame()
  }
}

function updateBallPosition() {
  const centerX = 250
  const centerY = 250
  const radius = 200
  const angle = Math.random() * 360
  const radians = angle * Math.PI / 180
  
  ballPosition.x = centerX + radius * Math.cos(radians)
  ballPosition.y = centerY + radius * Math.sin(radians)
}

function updateArcAngles() {
  const minAngle = gameSettings.minScoreAngle
  const maxAngle = gameSettings.maxScoreAngle
  
  arcAngles.value = [
    Math.random() * (360 - maxAngle + minAngle),
    Math.random() * (maxAngle - minAngle) + minAngle
  ]
  
  updateBallPosition()
}

function isValidTap(angle) {
  return angle >= arcAngles.value[0] && angle <= arcAngles.value[1]
}

function endGame() {
  gameState.value = 'ended'
  if (score.value > bestScore.value) {
    bestScore.value = score.value
    localStorage.setItem('bestScore', bestScore.value.toString())
  }
}

function calculateArcPath(centerX, centerY, radius, startAngle, endAngle) {
  const start = polarToCartesian(centerX, centerY, radius, endAngle)
  const end = polarToCartesian(centerX, centerY, radius, startAngle)
  const arcFlag = endAngle - startAngle <= 180 ? "0" : "1"
  
  return [
    "M", start.x, start.y, 
    "A", radius, radius, 0, arcFlag, 0, end.x, end.y
  ].join(" ")
}

function polarToCartesian(centerX, centerY, radius, angleInDegrees) {
  const angleInRadians = (angleInDegrees - 90) * Math.PI / 180.0
  
  return {
    x: centerX + radius * Math.cos(angleInRadians),
    y: centerY + radius * Math.sin(angleInRadians)
  }
}

function calculateTapAngle(event) {
  const svg = event.currentTarget
  const rect = svg.getBoundingClientRect()
  const x = event.clientX - rect.left
  const y = event.clientY - rect.top
  
  const centerX = 250
  const centerY = 250
  
  const angle = Math.atan2(y - centerY, x - centerX) * 180 / Math.PI + 90
  return angle < 0 ? angle + 360 : angle
}
</script>

<style scoped>
.game-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background-color: #f0f0f0;
}

.start-screen, .game-screen {
  text-align: center;
}

.developer-panel {
  margin-bottom: 20px;
}

.game-main {
  display: flex;
  flex-direction: column;
  align-items: center;
}

svg {
  border: 2px solid #333;
  border-radius: 50%;
}

.game-info {
  margin-top: 20px;
}

button {
  margin: 10px;
  padding: 10px 20px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

input {
  margin: 5px;
  padding: 5px;
}
</style>
