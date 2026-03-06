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
      
      <details class="info-box" open>
        <summary>What this shows</summary>
        <div class="info-content">
          <p>Every time the clocks go forward or back I find myself counting on my fingers to figure out what happens to daylight.</p>
          <p>This page might help show the relation between sunset, sunrise, and daylight saving. (Clocks go forward, but sunrise is later.)</p>
          <p>For extra goodness this shows the difference of the impact of DST depending on how far North or South you are, and its impact on morning and evening commutes.</p>
          <ul>
            <li><strong>LATITUDE:</strong> Move north or south to see how day length changes.</li>
            <li><strong>DAY OF YEAR:</strong> See hours of daylight change with the seasons.</li>
            <li><strong>DAYLIGHT SAVING OFFSET:</strong> Shift the clock axis to see how "Spring Forward" or "Fall Back" affects when you experience sunlight.</li>
            <li><strong>ANIMATION MODE:</strong> Toggle between <em>Move clock</em> (the reality) or <em>Move sun</em> (how we experience the shift).</li>
          </ul>
        </div>
      </details>
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

.info-box {
  margin: 1.5rem auto 0;
  max-width: 600px;
  background: rgba(15, 23, 42, 0.6);
  border: 1px solid #0ea5e9;
  border-radius: 4px;
  text-align: left;
  overflow: hidden;
  transition: all 0.3s ease;
}

.info-box summary {
  padding: 0.8rem 1rem;
  color: #0ea5e9;
  font-family: 'Fira Code', monospace;
  font-size: 0.9rem;
  font-weight: bold;
  letter-spacing: 1px;
  cursor: pointer;
  background: rgba(30, 41, 59, 0.8);
  outline: none;
  user-select: none;
}

.info-box summary:hover {
  background: rgba(14, 165, 233, 0.2);
  color: #00ffcc;
}

.info-box[open] summary {
  border-bottom: 1px solid rgba(14, 165, 233, 0.4);
}

.info-content {
  padding: 1rem;
  color: #cbd5e1;
  font-size: 0.95rem;
  line-height: 1.5;
}

.info-content p {
  margin-bottom: 0.8rem;
}

.info-content ul {
  padding-left: 1.5rem;
  margin: 0;
}

.info-content li {
  margin-bottom: 0.5rem;
}

.info-content strong {
  color: #00ffcc;
  font-family: 'Fira Code', monospace;
  font-size: 0.85rem;
}
</style>