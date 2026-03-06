<script setup>
import { ref, computed } from 'vue'

const latitude = ref(40) // Default latitude
const dayOfYear = ref(172) // Default to summer solstice (approx June 21)
const dstOffset = ref(0) // Default DST offset

const cities = [
  { name: 'Custom', lat: null },
  { name: 'Longyearbyen', lat: 78 },
  { name: 'Reykjavik', lat: 64 },
  { name: 'London', lat: 51 },
  { name: 'New York', lat: 40 },
  { name: 'Tokyo', lat: 35 },
  { name: 'Equator', lat: 0 },
  { name: 'Sydney', lat: -34 },
  { name: 'Cape Town', lat: -34 },
  { name: 'Punta Arenas', lat: -53 },
  { name: 'McMurdo Station', lat: -77 }
]
const selectedCity = ref('New York')

const onCityChange = () => {
  const city = cities.find(c => c.name === selectedCity.value)
  if (city && city.lat !== null) {
    latitude.value = city.lat
  }
}

const onLatitudeChange = () => {
  const city = cities.find(c => c.lat === latitude.value && c.name !== 'Custom')
  if (city) {
    selectedCity.value = city.name
  } else {
    selectedCity.value = 'Custom'
  }
}

const width = 800
const height = 400
const margin = { top: 20, right: 30, bottom: 80, left: 80 }

const innerWidth = width - margin.left - margin.right
const innerHeight = height - margin.top - margin.bottom

// Generate data points for every 10 minutes (144 points in a day)
const dataPoints = computed(() => {
  const points = []
  
  // Convert latitude to radians
  const latRad = latitude.value * (Math.PI / 180)
  
  // Approximate solar declination angle based on day of year
  // Formula: 23.45 * sin((360 / 365) * (day - 81))
  const declinationDeg = 23.45 * Math.sin((360 / 365) * (dayOfYear.value - 81) * (Math.PI / 180))
  const declinationRad = declinationDeg * (Math.PI / 180)

  for (let i = 0; i <= 144; i++) {
    const hour = (i / 144) * 24
    
    // Hour angle: 15 degrees per hour, 0 at solar noon (12:00)
    const hourAngleDeg = 15 * (hour - 12)
    const hourAngleRad = hourAngleDeg * (Math.PI / 180)
    
    // Solar altitude angle formula
    // sin(altitude) = sin(lat)*sin(decl) + cos(lat)*cos(decl)*cos(hourAngle)
    const sinAlt = Math.sin(latRad) * Math.sin(declinationRad) + 
                   Math.cos(latRad) * Math.cos(declinationRad) * Math.cos(hourAngleRad)
    const altitudeRad = Math.asin(sinAlt)
    const altitudeDeg = altitudeRad * (180 / Math.PI)
    
    points.push({
      hour,
      altitude: altitudeDeg
    })
  }
  return points
})

// Map data points to SVG coordinates
const mappedPoints = computed(() => {
  return dataPoints.value.map(p => {
    // X axis: 0 to 24 hours
    const x = margin.left + (p.hour / 24) * innerWidth
    
    // Y axis: +90 (top) to -90 (bottom)
    // Map altitude from [-90, 90] to [innerHeight, 0]
    // Normalized altitude: (p.altitude - (-90)) / 180 -> [0, 1]
    // Then flip Y since SVG origin is top-left
    const y = margin.top + innerHeight - ((p.altitude + 90) / 180) * innerHeight
    return { x, y, ...p }
  })
})

const polylinePoints = computed(() => {
  return mappedPoints.value.map(p => `${p.x},${p.y}`).join(' ')
})

// Horizon line (0 degrees altitude)
const horizonY = computed(() => {
  return margin.top + innerHeight - (90 / 180) * innerHeight
})

// Calculate sunrise and sunset times (decimal hours)
const solarEvents = computed(() => {
  const latRad = latitude.value * (Math.PI / 180)
  const declinationDeg = 23.45 * Math.sin((360 / 365) * (dayOfYear.value - 81) * (Math.PI / 180))
  const declinationRad = declinationDeg * (Math.PI / 180)

  const cosHourAngle = -Math.tan(latRad) * Math.tan(declinationRad)

  if (cosHourAngle < -1) {
    return { sunrise: null, sunset: null } // Midnight sun
  } else if (cosHourAngle > 1) {
    return { sunrise: null, sunset: null } // Polar night
  } else {
    const hourAngleRad = Math.acos(cosHourAngle)
    const hourAngleDeg = hourAngleRad * (180 / Math.PI)
    return {
      sunrise: 12 - (hourAngleDeg / 15),
      sunset: 12 + (hourAngleDeg / 15)
    }
  }
})

const formatMonth = (day) => {
  const date = new Date(2023, 0, day); // Using a non-leap year base
  return date.toLocaleDateString('en-US', { month: 'short', day: 'numeric' });
}

const formatTime = (decimalHour, offset = 0) => {
  let adjustedHour = decimalHour + offset;
  if (adjustedHour >= 24) adjustedHour -= 24;
  if (adjustedHour < 0) adjustedHour += 24;
  
  const h = Math.floor(adjustedHour)
  let m = Math.round((adjustedHour - h) * 60)
  let displayH = h
  if (m === 60) {
    m = 0
    displayH += 1
  }
  if (displayH >= 24) displayH -= 24;
  return `${String(displayH).padStart(2, '0')}:${String(m).padStart(2, '0')}`
}
</script>

<template>
  <div class="sun-graph-container">
    <div class="controls-panel">
      <div class="panel-header">SETTINGS</div>
      <div class="controls">
        <div class="control-group">
          <div class="header-row">
            <label for="latitude">LATITUDE [{{ latitude }}&deg;]</label>
            <select v-model="selectedCity" @change="onCityChange" class="city-select">
              <option v-for="city in cities" :key="city.name" :value="city.name">
                {{ city.name }}
              </option>
            </select>
          </div>
          <input 
            id="latitude" 
            type="range" 
            min="-90" 
            max="90" 
            step="1" 
            v-model.number="latitude" 
            @input="onLatitudeChange"
          />
          <div class="labels">
            <span>-90&deg; S</span>
            <span>0&deg; EQ</span>
            <span>+90&deg; N</span>
          </div>
        </div>
        
        <div class="control-group">
          <label for="dayOfYear">DAY_OF_YEAR [{{ dayOfYear }}] / {{ formatMonth(dayOfYear).toUpperCase() }}</label>
          <input 
            id="dayOfYear" 
            type="range" 
            min="1" 
            max="365" 
            step="1" 
            v-model.number="dayOfYear" 
          />
          <div class="labels">
            <span>DAY:001</span>
            <span>DAY:183</span>
            <span>DAY:365</span>
          </div>
        </div>

        <div class="control-group">
          <label>DST_OFFSET [+{{ dstOffset }}H]</label>
          <div class="button-group">
            <button :class="{ active: dstOffset === 0 }" @click="dstOffset = 0">STD</button>
            <button :class="{ active: dstOffset === 1 }" @click="dstOffset = 1">+1H</button>
            <button :class="{ active: dstOffset === 2 }" @click="dstOffset = 2">+2H</button>
          </div>
        </div>
      </div>
    </div>

    <div class="graph-panel">
      <div class="panel-corner top-left"></div>
      <div class="panel-corner top-right"></div>
      <div class="panel-corner bottom-left"></div>
      <div class="panel-corner bottom-right"></div>
      <div class="graph">
        <svg :viewBox="`0 0 ${width} ${height}`" width="100%" height="100%">
          <defs>
            <!-- Glow filter for the sun line -->
            <filter id="glow" x="-20%" y="-20%" width="140%" height="140%">
              <feGaussianBlur stdDeviation="4" result="blur" />
              <feComposite in="SourceGraphic" in2="blur" operator="over" />
            </filter>
            
            <filter id="glow-heavy" x="-50%" y="-50%" width="200%" height="200%">
              <feGaussianBlur stdDeviation="6" result="blur" />
              <feMerge>
                <feMergeNode in="blur" />
                <feMergeNode in="SourceGraphic" />
              </feMerge>
            </filter>
            
            <!-- Grid pattern for background -->
            <pattern id="gridPattern" width="20" height="20" patternUnits="userSpaceOnUse">
              <path d="M 20 0 L 0 0 0 20" fill="none" stroke="#0ea5e9" stroke-width="0.5" opacity="0.1" />
            </pattern>
            
            <clipPath id="axis-clip">
              <rect :x="margin.left - 25" :y="height - margin.bottom + 35" :width="innerWidth + 50" :height="40" />
            </clipPath>
          </defs>

          <!-- Background areas -->
          <!-- Sky (Above horizon) -->
          <rect :x="margin.left" :y="margin.top" :width="innerWidth" :height="horizonY - margin.top" fill="#3b82f6" opacity="0.45" />
          
          <!-- Ground/Night (Below horizon) -->
          <rect :x="margin.left" :y="horizonY" :width="innerWidth" :height="innerHeight - (horizonY - margin.top)" fill="#0f172a" opacity="0.85" />

          <!-- Background Grid pattern overlay -->
          <rect :x="margin.left" :y="margin.top" :width="innerWidth" :height="innerHeight" fill="url(#gridPattern)" />

          <!-- Axes and grid lines -->
          <g class="grid" fill="none">
            <!-- Horizon line -->
            <line :x1="margin.left" :y1="horizonY" :x2="width - margin.right" :y2="horizonY" stroke="#00ffcc" stroke-width="2" filter="url(#glow)" />
            
            <!-- Vertical lines for hours -->
            <line v-for="h in 25" :key="`v-${h}`" 
                  :x1="margin.left + ((h-1) / 24) * innerWidth" 
                  :y1="margin.top" 
                  :x2="margin.left + ((h-1) / 24) * innerWidth" 
                  :y2="height - margin.bottom" 
                  stroke="#0ea5e9" stroke-width="1" stroke-dasharray="4,4" opacity="0.5" />
                  
            <!-- Horizontal lines for degrees -->
            <line v-for="d in 7" :key="`h-${d}`"
                  :x1="margin.left"
                  :y1="margin.top + ((d-1) / 6) * innerHeight"
                  :x2="width - margin.right"
                  :y2="margin.top + ((d-1) / 6) * innerHeight"
                  stroke="#0ea5e9" stroke-width="1" stroke-dasharray="2,6" opacity="0.3" />
          </g>

          <!-- Axis labels -->
          <g class="labels" fill="#00ffcc" font-family="'Fira Code', monospace" font-size="10" opacity="0.8">
            <!-- Bottom X axis 1 (Solar Time) -->
            <g v-for="h in 25" :key="`x-${h}`" v-show="(h-1) % 4 === 0">
              <text 
                    :x="margin.left + ((h-1) / 24) * innerWidth" 
                    :y="height - margin.bottom + 25" 
                    text-anchor="middle">
                {{ String(h-1).padStart(2, '0') }}:00
              </text>
              <line 
                    :x1="margin.left + ((h-1) / 24) * innerWidth" 
                    :y1="height - margin.bottom + 30" 
                    :x2="margin.left + ((h-1) / 24) * innerWidth" 
                    :y2="height - margin.bottom + 35" 
                    stroke="#00ffcc" stroke-width="1.5" opacity="0.5" />
            </g>
            <text :x="margin.left - 35" :y="height - margin.bottom + 25" text-anchor="end" fill="#00ffcc">Solar</text>
            
            <!-- Bottom X axis 2 (Clock Time) with sliding animation -->
            <g clip-path="url(#axis-clip)">
              <g :style="{ transform: `translateX(${-(dstOffset / 24) * innerWidth}px)`, transition: 'transform 0.5s cubic-bezier(0.4, 0, 0.2, 1)' }">
                <g v-for="hour in 73" :key="`clock-x-${hour}`" v-show="(hour - 37) % 4 === 0">
                  <text 
                        :x="margin.left + ((hour - 37) / 24) * innerWidth" 
                        :y="height - margin.bottom + 55" 
                        text-anchor="middle"
                        fill="#ff00ff">
                    {{ String(((hour - 37) % 24 + 24) % 24).padStart(2, '0') }}:00
                  </text>
                  <line 
                        :x1="margin.left + ((hour - 37) / 24) * innerWidth" 
                        :y1="height - margin.bottom + 41" 
                        :x2="margin.left + ((hour - 37) / 24) * innerWidth" 
                        :y2="height - margin.bottom + 46" 
                        stroke="#ff00ff" stroke-width="1.5" opacity="0.5" />
                </g>
              </g>
            </g>
            <text :x="margin.left - 35" :y="height - margin.bottom + 55" text-anchor="end" fill="#ff00ff">Clock</text>

            <!-- Y axis (altitude) -->
            <text v-for="d in 7" :key="`y-${d}`"
                  :x="margin.left - 10"
                  :y="margin.top + ((d-1) / 6) * innerHeight + 4"
                  text-anchor="end">
              {{ 90 - (d-1) * 30 }}&deg;
            </text>
          </g>

          <!-- Horizon Label -->
          <text :x="width - margin.right + 5" :y="horizonY + 3" fill="#00ffcc" font-family="'Fira Code', monospace" font-size="10" filter="url(#glow)">HORIZON</text>

          <!-- Sunrise/Sunset lines and callouts -->
          <g v-if="solarEvents.sunrise !== null" class="solar-event sunrise">
            <line 
              :x1="margin.left + (solarEvents.sunrise / 24) * innerWidth"
              :y1="margin.top"
              :x2="margin.left + (solarEvents.sunrise / 24) * innerWidth"
              :y2="height - margin.bottom + 65"
              stroke="#ffffff" stroke-width="1.5" stroke-dasharray="3,3"
            />
            <!-- Solar Callout -->
            <rect
              :x="margin.left + (solarEvents.sunrise / 24) * innerWidth - 32"
              :y="height - margin.bottom + 8"
              width="64" height="22" rx="4"
              fill="#0f172a" stroke="#00ffcc" stroke-width="1.5"
            />
            <text
              :x="margin.left + (solarEvents.sunrise / 24) * innerWidth"
              :y="height - margin.bottom + 24"
              fill="#00ffcc" font-family="sans-serif" font-weight="bold" font-size="12" text-anchor="middle"
            >
              {{ formatTime(solarEvents.sunrise) }}
            </text>
            <!-- Clock Callout -->
            <rect
              :x="margin.left + (solarEvents.sunrise / 24) * innerWidth - 32"
              :y="height - margin.bottom + 38"
              width="64" height="22" rx="4"
              fill="#0f172a" stroke="#ff00ff" stroke-width="1.5"
            />
            <text
              :x="margin.left + (solarEvents.sunrise / 24) * innerWidth"
              :y="height - margin.bottom + 54"
              fill="#ff00ff" font-family="sans-serif" font-weight="bold" font-size="12" text-anchor="middle"
            >
              {{ formatTime(solarEvents.sunrise, dstOffset) }}
            </text>
          </g>

          <g v-if="solarEvents.sunset !== null" class="solar-event sunset">
            <line 
              :x1="margin.left + (solarEvents.sunset / 24) * innerWidth"
              :y1="margin.top"
              :x2="margin.left + (solarEvents.sunset / 24) * innerWidth"
              :y2="height - margin.bottom + 65"
              stroke="#ffffff" stroke-width="1.5" stroke-dasharray="3,3"
            />
            <!-- Solar Callout -->
            <rect
              :x="margin.left + (solarEvents.sunset / 24) * innerWidth - 32"
              :y="height - margin.bottom + 8"
              width="64" height="22" rx="4"
              fill="#0f172a" stroke="#00ffcc" stroke-width="1.5"
            />
            <text
              :x="margin.left + (solarEvents.sunset / 24) * innerWidth"
              :y="height - margin.bottom + 24"
              fill="#00ffcc" font-family="sans-serif" font-weight="bold" font-size="12" text-anchor="middle"
            >
              {{ formatTime(solarEvents.sunset) }}
            </text>
            <!-- Clock Callout -->
            <rect
              :x="margin.left + (solarEvents.sunset / 24) * innerWidth - 32"
              :y="height - margin.bottom + 38"
              width="64" height="22" rx="4"
              fill="#0f172a" stroke="#ff00ff" stroke-width="1.5"
            />
            <text
              :x="margin.left + (solarEvents.sunset / 24) * innerWidth"
              :y="height - margin.bottom + 54"
              fill="#ff00ff" font-family="sans-serif" font-weight="bold" font-size="12" text-anchor="middle"
            >
              {{ formatTime(solarEvents.sunset, dstOffset) }}
            </text>
          </g>

          <!-- The Sun Path -->
          <!-- Line -->
          <polyline :points="polylinePoints" fill="none" stroke="#ffb703" stroke-width="3" filter="url(#glow-heavy)" />
          
          <!-- Sun Dots (every hour) -->
          <circle v-for="(p, index) in mappedPoints.filter((_, i) => i % 6 === 0)" 
                  :key="`dot-${index}`"
                  :cx="p.x" 
                  :cy="p.y" 
                  r="4" 
                  fill="#fff" 
                  stroke="#fb8500" 
                  stroke-width="2"
                  filter="url(#glow)" />

          <!-- Current Solar Noon indicator (rough approximation for center) -->
          <line :x1="margin.left + 0.5 * innerWidth" :y1="margin.top" :x2="margin.left + 0.5 * innerWidth" :y2="height - margin.bottom" stroke="#fb8500" stroke-width="1" stroke-dasharray="2,2" opacity="0.4" />
          <text :x="margin.left + 0.5 * innerWidth" :y="margin.top - 5" fill="#fb8500" font-family="'Fira Code', monospace" font-size="10" text-anchor="middle" opacity="0.8">SOLAR_NOON</text>
        </svg>
      </div>
    </div>
  </div>
</template>

<style scoped>
.sun-graph-container {
  display: flex;
  flex-direction: column;
  gap: 2rem;
  max-width: 900px;
  width: 100%;
  margin: 0 auto;
}

.controls-panel {
  background: rgba(30, 41, 59, 0.8);
  border: 1px solid #0ea5e9;
  border-radius: 4px;
  padding: 1.5rem;
  position: relative;
  box-shadow: 0 0 15px rgba(14, 165, 233, 0.2);
}

.controls-panel::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 2px;
  background: linear-gradient(90deg, transparent, #00ffcc, transparent);
  opacity: 0.5;
}

.panel-header {
  position: absolute;
  top: -10px;
  left: 15px;
  background: #334155;
  padding: 0 10px;
  color: #0ea5e9;
  font-size: 0.8rem;
  letter-spacing: 2px;
  font-weight: bold;
}

.controls {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
  margin-top: 0.5rem;
}

.control-group {
  display: flex;
  flex-direction: column;
  gap: 0.8rem;
}

.button-group {
  display: flex;
  gap: 0.5rem;
  width: 100%;
}

.button-group button {
  flex: 1;
  background: #050510;
  color: #0ea5e9;
  border: 1px solid #0ea5e9;
  padding: 0.5rem;
  font-family: 'Fira Code', monospace;
  font-size: 0.9rem;
  cursor: pointer;
  transition: all 0.2s;
  border-radius: 2px;
}

.button-group button:hover {
  background: rgba(14, 165, 233, 0.2);
}

.button-group button.active {
  background: rgba(0, 255, 204, 0.2);
  color: #00ffcc;
  border-color: #00ffcc;
  box-shadow: 0 0 10px rgba(0, 255, 204, 0.4);
}

.header-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.city-select {
  background: #050510;
  color: #0ea5e9;
  border: 1px solid #0ea5e9;
  padding: 0.2rem 0.5rem;
  font-family: 'Fira Code', monospace;
  font-size: 0.8rem;
  outline: none;
  cursor: pointer;
}

.city-select:focus {
  border-color: #00ffcc;
  box-shadow: 0 0 5px rgba(0, 255, 204, 0.5);
}

label {
  font-size: 1rem;
  color: #00ffcc;
  text-shadow: 0 0 5px rgba(0, 255, 204, 0.5);
}

.labels {
  display: flex;
  justify-content: space-between;
  font-size: 0.75rem;
  color: #0ea5e9;
  opacity: 0.8;
}

.graph-panel {
  background: rgba(2, 6, 23, 0.8);
  border: 1px solid rgba(14, 165, 233, 0.4);
  position: relative;
  padding: 1rem;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.5), inset 0 0 30px rgba(14, 165, 233, 0.1);
}

/* Sci-fi corners for the graph */
.panel-corner {
  position: absolute;
  width: 15px;
  height: 15px;
  border: 2px solid #00ffcc;
  box-shadow: 0 0 5px #00ffcc;
}

.top-left { top: -1px; left: -1px; border-right: none; border-bottom: none; }
.top-right { top: -1px; right: -1px; border-left: none; border-bottom: none; }
.bottom-left { bottom: -1px; left: -1px; border-right: none; border-top: none; }
.bottom-right { bottom: -1px; right: -1px; border-left: none; border-top: none; }

.graph {
  width: 100%;
  height: 100%;
  overflow-x: auto;
}

svg {
  filter: drop-shadow(0 0 2px rgba(0,0,0,0.5));
}
</style>