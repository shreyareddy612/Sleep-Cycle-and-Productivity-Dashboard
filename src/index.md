---
theme: dashboard
title: How Exercise Duration Affects Sleep, Stress & Productivity
toc: false
---

<div class="dashboard-title">How Exercise Duration Affects Sleep, Stress & Productivity</div>

---

**Name:** Shreya Juka Reddy  
**Student ID:** 801391084

<p class="dashboard-description">
  This dashboard analyzes how daily exercise duration influences key well-being metrics—sleep quality, productivity, and stress—across different age groups. Through interactive visualizations, it highlights patterns that suggest optimal activity levels for balancing mental and physical health.
</p>


<style>
.dashboard-title {
  width: 100vw;
  max-width: 100vw;
  font-size: 2.8rem;
  font-weight: bold;
  letter-spacing: 0.01em;
  white-space: nowrap;
  overflow-x: auto;
  text-align: left;
  margin: 0 0 6px 0;
  padding: 12px 0 0 0;
}
.dashboard-description {
  width: 100vw;
  max-width: 100vw;
  font-size: 1.2rem;
  margin-bottom: 10px;
  margin-top: 2px;
  text-align: left;
  padding: 2px;
}
</style>

```js
import * as Plot from "@observablehq/plot";
import * as d3 from "d3";

const sleepData = await FileAttachment("/data/sleep_cycle_productivity_verified_final.csv").csv();

// Define the desired order of exercise bins
const binOrder = ["0-15", "16-30", "31-45", "46-60", "61-75", "75+"];

const validAgeGroups = ["18-24", "25-34", "35-44", "45-54", "55+"];

const groupedData = Array.from(
  d3.group(
    sleepData,
    d => d["Age Group"].trim(),
    d => d["Exercise Duration Bins"].trim()
  ),
  ([ageGroup, groupByBin]) =>
    Array.from(groupByBin, ([exerciseBin, values]) => ({
      AgeGroup: ageGroup,
      ExerciseBin: exerciseBin,
      AvgProductivity: d3.mean(values, d => +d["Productivity Score"])
    }))
).flat()
 .filter(d => binOrder.includes(d.ExerciseBin) && validAgeGroups.includes(d.AgeGroup));

const completeData = [];
for (const age of validAgeGroups) {
  for (const bin of binOrder) {
    const found = groupedData.find(d => d.AgeGroup === age && d.ExerciseBin === bin);
    completeData.push({
      AgeGroup: age,
      ExerciseBin: bin,
      AvgProductivity: found ? found.AvgProductivity : null
    });
  }
}

function productivityLineChart(data, {width} = {}) {
  return Plot.plot({
    title: "Average Productivity Score by Exercise Duration",
    width: Math.min(width || 500, 500),
    height: 250,
    x: {
      label: "Exercise Duration Bin",
      domain: binOrder, // Force correct order
    },
    y: {
      label: "Avg Productivity Score",
      domain: [4.6,6.2 ]
    },
    color: {legend: true},
    marks: [
      Plot.lineY(data, {
        x: "ExerciseBin",
        y: "AvgProductivity",
        stroke: "AgeGroup",
        curve: "linear"
      }),
      Plot.dot(data, {
        x: "ExerciseBin",
        y: "AvgProductivity",
        fill: "AgeGroup",
        tip: true
      }),
      Plot.ruleY([0])
    ]
  });
}

// --- Top-Right Chart Data ---
const avgByBin = binOrder.map(bin => {
  const filtered = sleepData.filter(d => d["Exercise Duration Bins"].trim() === bin);
  return {
    ExerciseBin: bin,
    AvgSleep: d3.mean(filtered, d => +d["Sleep Quality"]),
    AvgProductivity: d3.mean(filtered, d => +d["Productivity Score"])
  };
});

function sleepProductivityComboChart(data, {width} = {}) {
  return Plot.plot({
    title: "Moderate Exercise Optimizes Both Sleep Quality & Productivity",
    width: Math.min(width || 500, 500),
    height: 250,
    x: {label: "Exercise Duration Bin", domain: binOrder},
    y: {label: "Avg Sleep Quality (1–10 Likert Scale)", domain: [0, 7]},
    marks: [
      Plot.barY(data, {
        x: "ExerciseBin",
        y: "AvgSleep",
        fill: "#c44",
        tip: true
      }),
      Plot.lineY(data, {
        x: "ExerciseBin",
        y: "AvgProductivity",
        stroke: "#7a7",
        strokeWidth: 2,
        curve: "linear"
      }),
      Plot.dot(data, {
        x: "ExerciseBin",
        y: "AvgProductivity",
        fill: "#7a7",
        tip: true
      })
    ]
  });
}

// --- Bottom Chart Data ---
const wellBeingData = [];
for (const bin of binOrder) {
  for (const age of validAgeGroups) {
    const filtered = sleepData.filter(
      d => d["Age Group"].trim() === age && d["Exercise Duration Bins"].trim() === bin
    );
    wellBeingData.push({
      AgeGroup: age,
      ExerciseBin: bin,
      Metric: "Avg Productivity Score",
      Value: d3.mean(filtered, d => +d["Productivity Score"])
    });
    wellBeingData.push({
      AgeGroup: age,
      ExerciseBin: bin,
      Metric: "Avg Sleep Quality",
      Value: d3.mean(filtered, d => +d["Sleep Quality"])
    });
  }
}

function wellBeingBarChart(data, {width} = {}) {
  return Plot.plot({
    title: "Impact of Exercise Duration on Overall Well-being Metrics",
    width: Math.min(width || 500, 500),
    height: 250,
    x: {label: "Exercise Duration", domain: binOrder, padding: 0.3},
    y: {label: "Average Scores (1–10 Likert Scale)", domain: [0, 30]},
    color: {
      domain: validAgeGroups,
      legend: true
    },
    marks: [
      Plot.barY(data.filter(d => d.Metric === "Avg Productivity Score"), {
        x: "ExerciseBin",
        y: "Value",
        fill: "AgeGroup",
        dx: -0.18,
        tip: true
      }),
      Plot.barY(data.filter(d => d.Metric === "Avg Sleep Quality"), {
        x: "ExerciseBin",
        y: "Value",
        fill: "AgeGroup",
        dx: 0.18,
        tip: true,
        fillOpacity: 0.7
      }),
      Plot.ruleY([0])
    ]
  });
}

```
```html

<style>
.dashboard-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  grid-gap: 24px;
  max-width: 1300px;
  margin: 0 auto;
  padding: 20px;
}

.chart-card {
  background: #111; /* or leave transparent if using dark theme */
  padding: 10px;
  border-radius: 8px;
  box-shadow: 0 0 8px rgba(0,0,0,0.4);
}

.full-width {
  grid-column: span 2;
}

@media (max-width: 1000px) {
  .dashboard-grid {
    grid-template-columns: 1fr;
  }
  .full-width {
    grid-column: span 1;
  }
}
</style>

<div class="dashboard-grid">
  <div class="chart-card">
    ${resize(width => productivityLineChart(completeData, { width }))}
  </div>
  <div class="chart-card">
    ${resize(width => sleepProductivityComboChart(avgByBin, { width }))}
  </div>
  <div class="chart-card full-width">
    ${resize(width => wellBeingBarChart(wellBeingData, { width }))}
  </div>
</div>


```


