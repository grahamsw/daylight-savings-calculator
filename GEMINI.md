# Gemini AI Context: Daylight Savings Calculator

## Project Overview
This project is an interactive, single-page Vue 3 application designed to visualize the sun's trajectory and demonstrate the impact of Daylight Saving Time (DST) on perceived sunrise/sunset times. 

## Architectural & Stylistic Decisions
- **Stack:** Vue 3 via Vite, built with `<script setup>` syntax.
- **Styling:** Vanilla CSS, heavily utilizing a dark, "sci-fi terminal" aesthetic (e.g., Fira Code font, glowing neon cyan `#00ffcc`, deep slate backgrounds, glitch effects on headers).
- **Visualization Component (`SunGraph.vue`):**
  - Generates the chart using pure mathematical approximations for solar declination and altitude based on latitude and day of the year.
  - Rendered entirely via native SVG elements bound to reactive Vue computed properties (no external dependencies like D3 or Chart.js).
  - Features dual X-axes at the bottom: an absolute "Solar" time axis and an offset "Clock" time axis.
- **State Management:** Simple reactive `ref` variables handle global state (latitude, dayOfYear, dstOffset).

## Operating Context
- **OS:** win32
- **Shell Commands:** Use PowerShell syntax. Use semicolons (`;`) for chaining commands, not `&&`.