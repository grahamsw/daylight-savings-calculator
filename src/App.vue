<script setup>
import SunGraph from './components/SunGraph.vue'
</script>

<template>
  <div class="app-container">
    <header>
      <div class="glitch-wrapper">
        <h1 class="glitch" data-text="DAYLIGHT SAVINGS CALCULATOR">DAYLIGHT SAVINGS CALCULATOR</h1>
      </div>
      <p class="subtitle">Clocks go forward, so sunrise is...?</p>
    </header>
    <main>
      <SunGraph />
    </main>
  </div>
</template>

<style scoped>
.app-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 100vh;
}

header {
  margin-bottom: 2rem;
  text-align: center;
}

.subtitle {
  color: #00ffcc;
  font-size: 1.4rem;
  letter-spacing: 1px;
  margin-top: 5px;
  text-shadow: 0 0 10px rgba(0, 255, 204, 0.5);
  font-weight: 500;
}

.glitch-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
}

.glitch {
  position: relative;
  font-size: 2.5rem;
  font-weight: 700;
  line-height: 1.2;
  color: #fff;
  letter-spacing: 5px;
  text-transform: uppercase;
  z-index: 1;
  text-shadow: 0 0 10px rgba(0, 255, 204, 0.8), 0 0 20px rgba(0, 255, 204, 0.5);
}

.glitch::before,
.glitch::after {
  display: block;
  content: attr(data-text);
  position: absolute;
  top: 0;
  left: 0;
  opacity: 0.8;
}

.glitch::before {
  animation: glitch-color 0.3s cubic-bezier(0.25, 0.46, 0.45, 0.94) both infinite;
  color: #0ff;
  z-index: -1;
}

.glitch::after {
  animation: glitch-color 0.3s cubic-bezier(0.25, 0.46, 0.45, 0.94) reverse both infinite;
  color: #ff00ff;
  z-index: -2;
}

@keyframes glitch-color {
  0% {
    transform: translate(0);
  }
  20% {
    transform: translate(-2px, 2px);
  }
  40% {
    transform: translate(-2px, -2px);
  }
  60% {
    transform: translate(2px, 2px);
  }
  80% {
    transform: translate(2px, -2px);
  }
  100% {
    transform: translate(0);
  }
}

main {
  width: 100%;
}
</style>