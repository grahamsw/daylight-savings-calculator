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

main {
  width: 100%;
}
</style>