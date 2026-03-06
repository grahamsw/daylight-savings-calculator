# Daylight Savings Calculator

A lightweight, interactive Vue 3 application that visualizes the sun's trajectory across the sky over a 24-hour period. It demonstrates how Daylight Saving Time (DST) offsets affect clock time relative to true solar time.

## Features

- **Interactive Sun Graph:** A custom SVG visualization showing the sun's altitude plotted against a 24-hour horizontal axis.
- **Dual Time Axes:** Compare true "Solar Time" (where solar noon is always 12:00) with adjusted "Clock Time".
- **Dynamic Parameters:** 
  - Change the **Latitude** to see how the sun's path varies from the Equator to the Poles. Includes quick presets for major global cities.
  - Change the **Day of the Year** to see the effect of orbital declination across seasons (summer vs. winter).
- **DST Toggles:** Apply a +1H or +2H offset to see how the clock time shifts the perceived times for sunrise and sunset.
- **Sci-Fi Aesthetic:** A dark, glowing, terminal-inspired UI.

## Technology Stack

- **Framework:** Vue 3 (Composition API, `<script setup>`)
- **Build Tool:** Vite
- **Styling:** Vanilla CSS
- **Visualization:** Native SVG (No external charting libraries)

## Development

To run this project locally:

1. Clone the repository:
   ```bash
   git clone https://github.com/grahamsw/daylight-savings-calculator.git
   cd daylight-savings-calculator
   ```
2. Install dependencies:
   ```bash
   npm install
   ```
3. Start the development server:
   ```bash
   npm run dev
   ```

## Build for Production

To build the project for deployment (e.g., Firebase Hosting):

```bash
npm run build
```
This will compile and minify the assets into the `dist` directory.