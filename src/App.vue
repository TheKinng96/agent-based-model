<script lang="ts" setup>
import { ref, watch, onMounted, onUnmounted } from 'vue';
import * as d3 from 'd3';
import { LineChart } from 'vue-chart-3';

const numAgents = ref(10);
const svgContainer = ref(null);
const initialValueFields = ref<Array<{ label: string; min: number; max: number }>>([]);
const newFieldLabel = ref('');
const newFieldMin = ref(0);
const newFieldMax = ref(0);
const agents = ref([
  { id: 1, attribute1: 5, attribute2: 10, chartData: { /* ... */ } },
  { id: 2, attribute1: 8, attribute2: 12, chartData: { /* ... */ } },
  // ... more agents ...
]);
const selectedAgent = ref<{id: number, attribute1: number, chartData: any} | null>(null);

function showAgentDetails(agent: any) {
  selectedAgent.value = agent;
}

function closeAgentDetails() {
  selectedAgent.value = null;
}

function addInitialValueField() {
  if (
    newFieldLabel.value.trim() &&
    newFieldMin.value !== null &&
    newFieldMax.value !== null &&
    newFieldMin.value <= newFieldMax.value
  ) {
    initialValueFields.value.push({
      label: newFieldLabel.value,
      min: newFieldMin.value,
      max: newFieldMax.value,
    });
    newFieldLabel.value = '';
    newFieldMin.value = 0;
    newFieldMax.value = 0;
  }
}

function initVisualization() {
  const width = 600;
  const height = 400;
  
  const svg = d3.select(svgContainer.value)
    .append('svg')
    .attr('width', width)
    .attr('height', height);

  // Add agents as SVG circles
  const agents = Array.from({ length: numAgents.value }, () => ({
    x: Math.random() * width,
    y: Math.random() * height,
  }));

  svg.selectAll('circle')
    .data(agents)
    .join('circle')
    .attr('cx', (d) => d.x)
    .attr('cy', (d) => d.y)
    .attr('r', 5)
    .attr('fill', 'blue');
}

watch(numAgents, () => {
  d3.select(svgContainer.value).selectAll('*').remove();
  initVisualization();
});

onMounted(initVisualization);
onUnmounted(() => {
  d3.select(svgContainer.value).selectAll('*').remove();
});

function startSimulation() {
  console.log('Starting simulation with', numAgents.value, 'agents');
  // Call the API to start the simulation and update the visualization
}

function pauseSimulation() {
  console.log('Pausing simulation');
  // Pause the simulation
}

function stopSimulation() {
  console.log('Stopping simulation');
  // Stop the simulation
}

function resetSimulation() {
  console.log('Resetting simulation');
  // Reset the simulation and visualization
}
</script>

<template>
  <div id="app">
    <header>
      <h1>Agent-Based Model Web Application</h1>
    </header>

    <div class="parameters">
      <h2>Model Parameters</h2>
      <label for="agents">Number of Agents:</label>
      <input id="agents" type="number" v-model.number="numAgents" min="1" />
      <br />
      <h3>Initial Agent Values</h3>
    <div v-for="(field, index) in initialValueFields" :key="index">
      <label>{{ field.label }}</label>
      <input type="number" :min="field.min" :max="field.max" v-model.number="field.min" />
      <input type="number" :min="field.min" :max="field.max" v-model.number="field.max" />
    </div>
    <form @submit.prevent="addInitialValueField">
      <input type="text" v-model="newFieldLabel" placeholder="Field label" />
      <input type="number" v-model.number="newFieldMin" placeholder="Min value" />
      <input type="number" v-model.number="newFieldMax" placeholder="Max value" />
      <button type="submit">Add Field</button>
    </form>
  </div>

    <div class="visualization">
      <h2>Visualization</h2>
      <div ref="svgContainer"></div>
    </div>

    <div class="controls">
      <h2>Control Panel</h2>
      <button @click="startSimulation">Start</button>
      <button @click="pauseSimulation">Pause</button>
      <button @click="stopSimulation">Stop</button>
      <button @click="resetSimulation">Reset</button>
    </div>

    <div class="output">
    <h2>Output and Data Analysis</h2>
    <div class="agent-table">
      <table>
        <thead>
          <tr>
            <th>ID</th>
            <th>Attribute 1</th>
            <th>Attribute 2</th>
            <!-- Add more columns for additional attributes -->
          </tr>
        </thead>
        <tbody>
          <tr v-for="agent in agents" :key="agent.id" @click="showAgentDetails(agent)">
            <td>{{ agent.id }}</td>
            <td>{{ agent.attribute1 }}</td>
            <td>{{ agent.attribute2 }}</td>
            <!-- Add more cells for additional attributes -->
          </tr>
        </tbody>
      </table>
    </div>

    <div v-if="selectedAgent" class="agent-details-modal">
      <h3>Agent {{ selectedAgent.id }} Details</h3>
      <button @click="closeAgentDetails">Close</button>
      <line-chart :chart-data="selectedAgent.chartData"></line-chart>
    </div>
  </div>

    <div class="instructions">
      <h2>Instructions and Help</h2>
      <p>
        This is a simple agent-based model web application. Adjust the model
        parameters to customize the simulation, and use the control panel to
        start, pause, stop, or reset the simulation. The visualization section
        shows the behavior of the agents in the model, and the output section
        displays relevant data and analysis.
      </p>
    </div>
  </div>
</template>

<style>
body {
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 0;
}

header {
  padding: 1rem;
}

h1 {
  margin: 0;
  font-size: 1.5rem;
}

.parameters,
.visualization,
.controls,
.output,
.instructions {
  padding: 1rem;
  margin-bottom: 1rem;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  background-color: #4CAF50;
  border: none;
  color: white;
  padding: 0.5rem 1rem;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 1rem;
  margin: 0.25rem 0.5rem;
  cursor: pointer;
  border-radius: 4px;
}

button:hover {
  background-color: #45a049;
}

.agent-table table {
  width: 100%;
  border-collapse: collapse;
}

.agent-table th,
.agent-table td {
  border: 1px solid #ccc;
  padding: 0.5rem;
  text-align: left;
}

.agent-details-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}
</style>
