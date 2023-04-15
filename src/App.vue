<script lang="ts" setup>
import { ref, watch, onMounted, onUnmounted, computed } from 'vue';
import * as d3 from 'd3';
import { LineChart } from 'vue-chart-3';
import axios from 'axios';

interface IAgent {
  id: number;
  attributes: IAttribute[];
  x: number;
  y: number;
}

interface IAttribute {
  label: string;
  min: number;
  max: number;
}

const numAgents = ref(10);
const svgContainer = ref(null);
const initialValueFields = ref<Array<IAttribute>>([]);
const newFieldLabel = ref('');
const newFieldMin = ref(0);
const newFieldMax = ref(0);
const numIterations = ref(1);
const movable = ref(false);
const moveDistance = ref(1);

// Coordinate ranges
const xMax = ref(100);
const yMax = ref(100);

const coordinateRanges = computed(() => {
  return {
    xMin: 0,
    xMax: xMax.value,
    yMin: 0,
    yMax: yMax.value,
  };
});

const agents = ref<IAgent[]>([]);
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
  const width = coordinateRanges.value.xMax;
  const height = coordinateRanges.value.yMax;
  
  const svg = d3.select(svgContainer.value)
    .append('svg')
    .attr('width', width)
    .attr('height', height);

  // Add agents as SVG circles
  const agentsData = agents.value.map((record: IAgent) => ({
    x: record.x,
    y: record.y,
  }));
  console.log(agentsData)

  svg.selectAll('circle')
    .data(agentsData)
    .join('circle')
    .attr('cx', (d) => d.x)
    .attr('cy', (d) => d.y)
    .attr('r', 5)
    .attr('fill', 'blue');
}

function updateVisualization() {
  const agentsData = agents.value.map((record: IAgent) => ({
    x: record.x,
    y: record.y,
  }));

  console.log(agentsData)

  d3.select(svgContainer.value)
    .selectAll('circle')
    .data(agentsData)
    .join('circle')
    .attr('cx', (d) => d.x)
    .attr('cy', (d) => d.y)
    .attr('r', 5)
    .attr('fill', 'blue');
}

watch(agents.value, () => {
  d3.select(svgContainer.value).selectAll('*').remove();
  initVisualization();
});

onMounted(initVisualization);
onUnmounted(() => {
  d3.select(svgContainer.value).selectAll('*').remove();
});

async function setupSimulation() {
  const payload = {
    numAgents: numAgents.value,
    attributes: initialValueFields.value,
    numIterations: numIterations.value,
    movable: movable.value,
    coordinateRanges: coordinateRanges.value,
    moveDistance: moveDistance.value,
  };

  try {
    const response = await axios.post('http://localhost:3001/simulate', payload);
    if (response.data.success) {
      const results = response.data.results;
      agents.value = results;
      initVisualization();
    } else {
      console.error('Simulation failed');
    }
  } catch (error) {
    console.error('Error while starting simulation:', error);
  }
}

async function startSimulation() {
  const payload = {
    agents: agents.value,
    numIterations: numIterations.value,
    movable: movable.value,
    coordinateRanges: coordinateRanges.value,
    moveDistance: moveDistance.value,
  };

  try {
    const response = await axios.post('http://localhost:3001/next', payload);
    if (response.data.success) {
      const results = response.data.results;
      agents.value = results;
      updateVisualization();
    } else {
      console.error('Simulation failed');
    }
  } catch (error) {
    console.error('Error while starting simulation:', error);
  }
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
      <label for="numIterations">Total Iterations:</label>
      <input type="number" id="numIterations" min="1" v-model="numIterations"/>

      <label for="movable-checkbox">Allow agents to move:</label>
      <input
        id="movable-checkbox"
        type="checkbox"
        v-model="movable"
      />

      <button @click="setupSimulation">Setup</button>
      <button @click="startSimulation">Start</button>
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
