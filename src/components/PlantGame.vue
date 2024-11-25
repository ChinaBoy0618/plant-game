<template>
  <div class="plant-game">
    <div class="game-container">
      <h1 class="game-title">🌿 植物小花园 🌿</h1>
      <div class="seed-selector">
        <div v-for="(seed, type) in seedTypes" 
             :key="type"
             :class="['seed-option', { selected: selectedSeedType === type }]"
             @click="selectedSeedType = type">
          <span class="seed-emoji">{{ seed.emoji }}</span>
          <span class="seed-name">{{ seed.name }}</span>
          <span class="seed-count">x{{ inventory[type] }}</span>
        </div>
      </div>
      <div class="garden">
        <div v-for="(plot, index) in gardenPlots" 
             :key="index" 
             class="plot"
             @click="plantSeed(index)">
          <template v-if="plot.plant">
            <div :class="['plant', getPlantState(plot)]">
              <div class="plant-emoji">{{ getPlantEmoji(plot) }}</div>
              <div v-if="plot.plant.isWatering && isWatering" class="cloud-container">
                <div class="cloud">☁️</div>
                <div class="cloud-drops">
                  <div class="cloud-drop">💧</div>
                  <div class="cloud-drop">💧</div>
                  <div class="cloud-drop">💧</div>
                </div>
              </div>
              <div v-if="plot.plant.isWatering && isPlaneWatering" class="water-drop"></div>
              <div v-if="plot.plant.isFertilizing" class="fertilizer-effect">
                <div class="sparkle">✨</div>
                <div class="sparkle">✨</div>
                <div class="sparkle">✨</div>
                <div class="fertilizer-particles">
                  <div v-for="i in 8" :key="i" class="particle"></div>
                </div>
              </div>
            </div>
          </template>
          <template v-else>
            <div class="empty-plot">🟫</div>
          </template>
        </div>
        <div v-if="isPlaneWatering" class="plane-container">
          <div class="plane">✈️</div>
          <div class="rain-container">
            <div v-for="i in 10" :key="i" class="raindrop"></div>
          </div>
        </div>
      </div>
      <div class="controls">
        <button @click="waterPlants" :disabled="isWatering || isPlaneWatering" class="control-btn water-btn">浇水 💧</button>
        <button @click="fertilizePlants" :disabled="isFertilizing" class="control-btn fertilizer-btn">施肥 🌿</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const BASE_GROWTH_RATE = 0.6
const BASE_WATER_CONSUMPTION = 0.033

const seedTypes = {
  sunflower: {
    name: '向日葵',
    emoji: '🌻',
    growthRate: 1.2,
    waterConsumption: 1,
    maxGrowth: 100,
    stages: {
      seed: '🌱',
      growing: '🌿',
      dry: '🥀',
      mature: '🌻'
    }
  },
  rose: {
    name: '玫瑰',
    emoji: '🌹',
    growthRate: 0.8,
    waterConsumption: 1.2,
    maxGrowth: 100,
    stages: {
      seed: '🌱',
      growing: '🌿',
      dry: '🥀',
      mature: '🌹'
    }
  },
  tulip: {
    name: '郁金香',
    emoji: '🌷',
    growthRate: 1,
    waterConsumption: 0.8,
    maxGrowth: 100,
    stages: {
      seed: '🌱',
      growing: '🌿',
      dry: '🥀',
      mature: '🌷'
    }
  }
}

const selectedSeedType = ref('sunflower')
const inventory = ref({
  sunflower: 5,
  rose: 3,
  tulip: 3
})

const gardenPlots = ref(Array(9).fill().map(() => ({
  plant: null
})))

const isWatering = ref(false)
const isFertilizing = ref(false)
const isPlaneWatering = ref(false)

const plantSeed = (index) => {
  if (gardenPlots.value[index].plant) return
  
  if (inventory.value[selectedSeedType.value] <= 0) {
    alert('种子不足！')
    return
  }
  
  inventory.value[selectedSeedType.value]--
  
  gardenPlots.value[index].plant = {
    type: selectedSeedType.value,
    growth: 0,
    water: 100,
    lastWatered: Date.now(),
    isWatering: false,
    isFertilizing: false
  }
}

const waterPlants = async () => {
  if (isWatering.value || isPlaneWatering.value) return
  
  const plantedCount = getPlantedCount()
  
  console.log('已种植数量:', plantedCount)  // 调试信息
  console.log('判断条件: 3')  // 调试信息
  
  // 当种植数量大于3个时使用飞机浇水，否则使用云朵浇水
  if (plantedCount > 3) {
    console.log('使用飞机浇水')  // 调试信息
    // 使用飞机浇水
    isPlaneWatering.value = true
    
    // 所有植物同时开始浇水动画
    gardenPlots.value.forEach(plot => {
      if (plot.plant) {
        plot.plant.isWatering = true
      }
    })
    
    // 等待飞机飞过动画完成
    await new Promise(resolve => setTimeout(resolve, 3000))
    
    // 更新所有植物的水分
    gardenPlots.value.forEach(plot => {
      if (plot.plant) {
        plot.plant.water = 100
        plot.plant.isWatering = false
      }
    })
    
    isPlaneWatering.value = false
  } else {
    console.log('使用云朵浇水')  // 调试信息
    // 使用云朵浇水
    isWatering.value = true
    
    for (const plot of gardenPlots.value) {
      if (plot.plant) {
        plot.plant.isWatering = true
        await new Promise(resolve => setTimeout(resolve, 1500))
        plot.plant.water = 100
        plot.plant.isWatering = false
      }
    }
    
    isWatering.value = false
  }
}

const fertilizePlants = async () => {
  if (isFertilizing.value) return
  isFertilizing.value = true
  
  // 找出所有种植了植物的土地
  const plotsWithPlants = gardenPlots.value
    .map((plot, index) => ({ plot, index }))
    .filter(item => item.plot.plant)
  
  // 同时给所有植物施肥
  plotsWithPlants.forEach(({ plot }) => {
    plot.plant.isFertilizing = true
    // 增加生长值
    plot.plant.growth = Math.min(100, plot.plant.growth + 20)
  })
  
  // 等待动画完成
  await new Promise(resolve => setTimeout(resolve, 1500))
  
  // 同时结束所有植物的施肥状态
  plotsWithPlants.forEach(({ plot }) => {
    plot.plant.isFertilizing = false
  })
  
  isFertilizing.value = false
}

const addFertilizer = async () => {
  if (isFertilizing.value) return
  isFertilizing.value = true

  // 为每个植物添加施肥动画
  gardenPlots.value.forEach(plot => {
    if (plot.plant) {
      plot.plant.isFertilizing = true
      plot.plant.growth = Math.min(100, plot.plant.growth + 10)
    }
  })

  // 等待动画完成
  await new Promise(resolve => setTimeout(resolve, 1000))

  // 移除动画
  gardenPlots.value.forEach(plot => {
    if (plot.plant) {
      plot.plant.isFertilizing = false
    }
  })

  isFertilizing.value = false
}

const getPlantState = (plot) => {
  if (plot.plant.water < 20) return 'withered'
  if (plot.plant.water < 50) return 'dry'
  if (plot.plant.growth > 80) return 'mature'
  return ''
}

const getPlantEmoji = (plot) => {
  const seedType = seedTypes[plot.plant.type]
  
  if (plot.plant.water < 20) {
    return seedType.stages.dry
  }
  if (plot.plant.water < 50) {
    return plot.plant.growth > 80 ? seedType.stages.mature : seedType.stages.growing
  }
  if (plot.plant.growth > 80) {
    return seedType.stages.mature
  }
  return seedType.stages.seed
}

const getPlantedCount = () => {
  return gardenPlots.value.filter(plot => plot.plant).length
}

// 游戏循环
const gameLoop = () => {
  gardenPlots.value.forEach(plot => {
    if (plot.plant) {
      // 水分减少速度根据植物类型调整
      const waterLoss = seedTypes[plot.plant.type].waterConsumption * BASE_WATER_CONSUMPTION
      plot.plant.water = Math.max(0, plot.plant.water - waterLoss)
      
      // 生长速度根据植物类型调整
      if (plot.plant.water > 30) {
        const growthIncrease = seedTypes[plot.plant.type].growthRate * BASE_GROWTH_RATE
        plot.plant.growth = Math.min(100, plot.plant.growth + growthIncrease)
      }
    }
  })
}

let gameInterval
onMounted(() => {
  gameInterval = setInterval(gameLoop, 1000)
})

onUnmounted(() => {
  clearInterval(gameInterval)
})
</script>

<style scoped>
.plant-game {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background-color: #f0f8ff;
  padding: 20px;
}

.game-container {
  background-color: white;
  padding: 30px;
  border-radius: 20px;
  box-shadow: 0 4px 20px rgba(0,0,0,0.1);
  text-align: center;
}

.game-title {
  color: #2c3e50;
  margin-bottom: 30px;
  font-size: 2em;
}

.garden {
  position: relative;
  display: grid;
  grid-template-columns: repeat(3, 100px);
  gap: 0;
  padding: 20px;
  background: #8B4513;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0,0,0,0.2);
}

.plot {
  width: 100px;
  height: 100px;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  position: relative;
  background: linear-gradient(45deg, #654321, #8B4513);
  border: 1px solid #543210;
  transition: all 0.3s ease;
}

.empty-plot {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 24px;
  background: linear-gradient(45deg, #654321, #8B4513);
  color: rgba(255,255,255,0.1);
  transition: all 0.3s ease;
}

.plot:hover {
  background: linear-gradient(45deg, #7B5321, #9B5523);
}

.plant {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  transition: all 0.5s ease;
}

.plant-emoji {
  font-size: 40px;
  position: relative;
  z-index: 1;
  transition: all 0.3s ease;
}

.water-drop {
  position: absolute;
  width: 10px;
  height: 10px;
  background: #4fa3ff;
  border-radius: 50%;
  opacity: 0.7;
  animation: fall 0.5s linear infinite;
}

.cloud-container {
  position: absolute;
  top: -40px;
  left: 50%;
  transform: translateX(-50%);
  width: 40px;
  height: 60px;
  pointer-events: none;
  z-index: 3;
}

.cloud {
  position: absolute;
  top: 0;
  left: 0;
  font-size: 24px;
  animation: cloud-float 1.5s ease-in-out;
}

.cloud-drops {
  position: absolute;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  width: 30px;
  height: 30px;
}

.cloud-drop {
  position: absolute;
  font-size: 12px;
  opacity: 0;
  animation: drop-fall 0.6s linear infinite;
}

.cloud-drop:nth-child(1) {
  left: 0;
  animation-delay: 0s;
}

.cloud-drop:nth-child(2) {
  left: 10px;
  animation-delay: 0.2s;
}

.cloud-drop:nth-child(3) {
  left: 20px;
  animation-delay: 0.4s;
}

.plane-container {
  position: absolute;
  top: -50px;
  left: -50px;
  width: calc(100% + 100px);
  height: calc(100% + 50px);
  pointer-events: none;
  animation: fly-diagonal 3s linear;
}

.plane {
  position: absolute;
  top: 0;
  left: 0;
  font-size: 24px;
  transform: rotate(15deg);
}

.rain-container {
  position: absolute;
  top: 30px;
  left: 40px;
  width: 60px;
  height: 100%;
  transform: rotate(-15deg);
}

.raindrop {
  position: absolute;
  width: 2px;
  height: 10px;
  background: #4fa3ff;
  border-radius: 50%;
  opacity: 0.7;
  animation: fall 0.5s linear infinite;
}

.raindrop:nth-child(1) { left: 10%; animation-delay: 0.1s; }
.raindrop:nth-child(2) { left: 20%; animation-delay: 0.2s; }
.raindrop:nth-child(3) { left: 30%; animation-delay: 0.3s; }
.raindrop:nth-child(4) { left: 40%; animation-delay: 0.4s; }
.raindrop:nth-child(5) { left: 50%; animation-delay: 0.5s; }
.raindrop:nth-child(6) { left: 60%; animation-delay: 0.1s; }
.raindrop:nth-child(7) { left: 70%; animation-delay: 0.2s; }
.raindrop:nth-child(8) { left: 80%; animation-delay: 0.3s; }
.raindrop:nth-child(9) { left: 90%; animation-delay: 0.4s; }
.raindrop:nth-child(10) { left: 100%; animation-delay: 0.5s; }

.fertilizer-effect {
  position: absolute;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 2;
  animation: fertilizer-glow 1.5s ease-out;
}

.sparkle {
  position: absolute;
  font-size: 20px;
  opacity: 0;
  animation: sparkle-float 1.5s ease-out;
  transform-origin: center;
}

.sparkle:nth-child(1) {
  left: 10%;
  top: 20%;
  animation-delay: 0.1s;
}

.sparkle:nth-child(2) {
  left: 50%;
  top: 10%;
  animation-delay: 0.3s;
}

.sparkle:nth-child(3) {
  left: 80%;
  top: 30%;
  animation-delay: 0.5s;
}

.fertilizer-particles {
  position: absolute;
  width: 100%;
  height: 100%;
}

.particle {
  position: absolute;
  width: 4px;
  height: 4px;
  background: #90EE90;
  border-radius: 50%;
  opacity: 0;
  animation: particle-rise 1.5s ease-out;
}

.particle:nth-child(1) { left: 20%; animation-delay: 0.1s; }
.particle:nth-child(2) { left: 40%; animation-delay: 0.2s; }
.particle:nth-child(3) { left: 60%; animation-delay: 0.3s; }
.particle:nth-child(4) { left: 80%; animation-delay: 0.4s; }
.particle:nth-child(5) { left: 30%; animation-delay: 0.2s; }
.particle:nth-child(6) { left: 50%; animation-delay: 0.3s; }
.particle:nth-child(7) { left: 70%; animation-delay: 0.4s; }
.particle:nth-child(8) { left: 90%; animation-delay: 0.5s; }

@keyframes fly-diagonal {
  0% {
    transform: translate(0, 0);
  }
  100% {
    transform: translate(100%, 50%);
  }
}

@keyframes fall {
  0% {
    transform: translateY(-10px);
    opacity: 0;
  }
  50% {
    opacity: 0.7;
  }
  100% {
    transform: translateY(100px);
    opacity: 0;
  }
}

@keyframes cloud-float {
  0% {
    transform: translate(-20px, 0);
  }
  50% {
    transform: translate(0, 0);
  }
  100% {
    transform: translate(20px, 0);
  }
}

@keyframes drop-fall {
  0% {
    transform: translateY(0);
    opacity: 0;
  }
  30% {
    opacity: 1;
  }
  80% {
    opacity: 1;
  }
  100% {
    transform: translateY(30px);
    opacity: 0;
  }
}

@keyframes fertilizer-glow {
  0% {
    filter: brightness(1);
  }
  50% {
    filter: brightness(1.5);
  }
  100% {
    filter: brightness(1);
  }
}

@keyframes sparkle-float {
  0% {
    transform: translateY(20px) scale(0.5) rotate(0deg);
    opacity: 0;
  }
  20% {
    opacity: 1;
  }
  80% {
    opacity: 1;
  }
  100% {
    transform: translateY(-30px) scale(1.2) rotate(360deg);
    opacity: 0;
  }
}

@keyframes particle-rise {
  0% {
    transform: translateY(0) scale(0.5);
    opacity: 0;
  }
  20% {
    opacity: 0.8;
  }
  80% {
    opacity: 0.8;
  }
  100% {
    transform: translateY(-40px) scale(1.5);
    opacity: 0;
  }
}

.controls {
  display: flex;
  gap: 20px;
  justify-content: center;
}

.control-btn {
  padding: 12px 24px;
  border: none;
  border-radius: 10px;
  font-size: 16px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-weight: bold;
}

.water-btn {
  background-color: #4dabf7;
  color: white;
}

.fertilizer-btn {
  background-color: #40c057;
  color: white;
}

.control-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}

.control-btn:active {
  transform: translateY(0);
}

.control-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none;
}

.seed-selector {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  justify-content: center;
}

.seed-option {
  display: flex;
  align-items: center;
  gap: 5px;
  padding: 5px 10px;
  border: 2px solid #8B4513;
  border-radius: 5px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.seed-option:hover {
  background: rgba(139, 69, 19, 0.1);
}

.seed-option.selected {
  background: #8B4513;
  color: white;
}

.seed-emoji {
  font-size: 24px;
}

.seed-name {
  font-size: 14px;
}

.seed-count {
  font-size: 14px;
  color: #666;
}

.seed-option.selected .seed-count {
  color: #fff;
}

.plant.withered .plant-emoji {
  transform: rotate(45deg);
  opacity: 0.7;
}

.plant.dry .plant-emoji {
  transform: rotate(-15deg);
  opacity: 0.8;
}

.plant.mature .plant-emoji {
  transform: scale(1.2);
}
</style>
