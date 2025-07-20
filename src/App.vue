<template>
  <div class="container wider">
    <div class="top-bar">
      <div class="probabilities">
        <table class="prob-table">
          <thead>
          <tr>
            <th>Вгадано</th>
            <th v-for="row in probabilities" :key="'k' + row.k">{{ row.k }}</th>
          </tr>
          </thead>
          <tbody>
          <tr>
            <td>Ймовірність</td>
            <td v-for="row in probabilities" :key="'p' + row.k">{{ (row.prob * 100).toFixed(5) }}%</td>
          </tr>
          </tbody>
        </table>
      </div>
      <div class="controls">
        <div class="button-container">
          <button @click="checkMatches" :disabled="checked">Перевірити</button>
          <div v-if="checked" class="disabled-message">Перевірку вже виконано</div>
        </div>
        <div><strong>Вгадано {{ correctCount }} з 12</strong></div>
      </div>
    </div>

    <h2 class="title">Перетягни знак зодіаку на відповідний опис</h2>

    <div class="signs-row">
      <Draggable
          v-model="availableSigns"
          group="signs"
          item-key="sign"
          class="signs-inline wrap"
          :swap="true"
          :swapThreshold="0.5"
          @start="startDragging"
          @end="endDragging"
      >
        <template #item="{ element }">
          <div class="sign-card">{{ element }}</div>
        </template>
      </Draggable>
    </div>

    <div class="descriptions">
      <div v-for="(item, index) in shuffledDescriptions" :key="index" class="description-block wider-text">
        <Draggable
            v-model="assignedSigns[index]"
            group="signs"
            item-key="sign"
            class="dropzone"
            :swap="true"
            :swapThreshold="0.5"
            :move="checkMove"
            @start="startDragging"
            @end="endDragging"
        >
          <template #item="{ element }">
            <div class="sign-card" :class="{ 
              'correct': !isDragging && !colorsHiddenPermanently && verificationResults[index] === true, 
              'incorrect': !isDragging && !colorsHiddenPermanently && verificationResults[index] === false 
            }">{{ element }}</div>
          </template>
        </Draggable>
        <div class="description-text">{{ item.text }}</div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import Draggable from 'vuedraggable';
import descriptionsJson from './data/descriptions.json';
import probabilityData from './data/probabilities.json';

interface Description {
  sign: string;
  text: string;
}

interface ProbabilityRow {
  k: number;
  prob: number;
}

const shuffledDescriptions = ref<Description[]>([]);
const availableSigns = ref<string[]>([]);
const assignedSigns = ref<(string[])[]>([]);
const correctCount = ref(0);
const probabilities = ref<ProbabilityRow[]>([]);
const verificationResults = ref<{[index: number]: boolean | null}>({}); // Store verification results
const checked = ref(false);
const isDragging = ref(false); // Track if any card is currently being dragged
const colorsHiddenPermanently = ref(false); // Track if colors should be permanently hidden

onMounted(() => {
  console.log('Ініціалізація додатку...');

  console.log('Завантаження та перемішування описів знаків зодіаку');
  shuffledDescriptions.value = shuffleArray([...descriptionsJson]);

  console.log('Завантаження та перемішування знаків зодіаку');
  availableSigns.value = shuffleArray(descriptionsJson.map(d => d.sign));

  console.log('Створення порожніх слотів для розміщення знаків');
  assignedSigns.value = Array(12).fill(null).map(() => []);

  console.log('Завантаження даних про ймовірності');
  probabilities.value = probabilityData;

  // Initialize verification results to null (not verified yet)
  console.log('Ініціалізація результатів перевірки');
  shuffledDescriptions.value.forEach((_, index) => {
    verificationResults.value[index] = null;
  });

  console.log('Додаток готовий до використання');
});

function shuffleArray<T>(array: T[]): T[] {
  console.log(`Перемішування масиву з ${array.length} елементів`);
  return [...array].sort(() => Math.random() - 0.5);
}

function checkMatches() {
  if (checked.value) {
    console.log('Перевірка вже була виконана');
    return;
  }

  console.log('Початок перевірки відповідей');
  let correct = 0;
  shuffledDescriptions.value.forEach((desc, index) => {
    const assigned = assignedSigns.value[index][0];
    // Check if the assigned sign matches the correct sign
    const isCorrect = assigned === desc.sign;
    // Update verification results
    verificationResults.value[index] = isCorrect;
    if (isCorrect) correct++;
  });
  correctCount.value = correct;
  checked.value = true;
  console.log(`Перевірка завершена: вгадано ${correct} з 12 знаків`);
}

function checkMove(evt) {
  const draggedElement = evt.draggedContext.element;
  const targetElement = evt.relatedContext.element;
  const targetList = evt.relatedContext.list;

  if (draggedElement && targetElement) {
    return true;
  }

  return !targetElement && targetList.length === 0;
}

function startDragging() {
  isDragging.value = true;

  if (checked.value && !colorsHiddenPermanently.value) {
    colorsHiddenPermanently.value = true;
  }
}

function endDragging() {
  isDragging.value = false;
}
</script>

<style scoped>
.container.wider {
  width: 100%;
  padding: 20px;
  background: #121212;
  color: #eee;
  font-family: sans-serif;
  box-sizing: border-box;
}

.top-bar {
  display: flex;
  flex-direction: column;
  gap: 20px;
  margin-bottom: 20px;
  align-items: center;
}

@media (min-width: 768px) {
  .top-bar {
    flex-direction: row;
    justify-content: space-between;
    align-items: start;
  }
}

.controls {
  display: flex;
  gap: 20px;
  align-items: center;
}

.button-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
}

.disabled-message {
  font-size: 14px;
  color: #ff9800;
  font-style: italic;
  text-align: center;
}

.probabilities {
  background: #1e1e1e;
  padding: 12px;
  border-radius: 8px;
  border: 1px solid #333;
  overflow-x: auto;
}

.prob-table {
  border-collapse: collapse;
  color: #eee;
}

.prob-table th,
.prob-table td {
  border: 1px solid #333;
  padding: 4px 8px;
  text-align: center;
  font-size: 13px;
  background: #2a2a2a;
}

button:disabled {
  opacity: 0.7;
  cursor: not-allowed;
  background-color: #555;
  border: 1px solid #777;
  color: #ccc;
  text-decoration: line-through;
}

button {
  padding: 10px 16px;
  background-color: #1976d2;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: transform 0.1s ease, background-color 0.3s ease;
}

button:active {
  transform: scale(0.95);
}

.signs-row {
  margin: 20px 0;
}

.signs-inline {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  justify-content: center;
}

.sign-card {
  background: #333;
  color: #fff8e1;
  padding: 10px 16px;
  border-radius: 6px;
  border: 1px solid #555;
  cursor: grab;
  font-weight: 500;
  font-size: 16px;
  min-width: 100px;
  text-align: center;
  transition: background-color 0.3s, border-color 0.3s;
}

.sign-card.correct {
  background-color: #4caf50;
  border-color: #2e7d32;
  color: white;
}

.sign-card.incorrect {
  background-color: #f44336;
  border-color: #c62828;
  color: white;
}

.descriptions {
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  gap: 20px;
  width: 100%;
}

@media (max-width: 1200px) {
  .descriptions {
    grid-template-columns: repeat(3, 1fr);
  }
}

@media (max-width: 768px) {
  .descriptions {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 480px) {
  .descriptions {
    grid-template-columns: 1fr;
  }
}

.description-block.wider-text {
  background: #1c1c1c;
  padding: 20px;
  border-radius: 10px;
  border: 1px solid #333;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.4);
  width: 100%;
  box-sizing: border-box;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.dropzone {
  min-height: 40px;
  padding: 8px;
  border: 2px dashed #444;
  background: #2a2a2a;
  border-radius: 6px;
  margin-bottom: 10px;
  text-align: center;
  flex-shrink: 0;
}

.description-text {
  font-size: 18px;
  line-height: 1.75;
  color: #ddd;
  flex-grow: 1;
  text-align: justify;
}

.title {
  text-align: center;
  margin: 20px 0;
  font-size: 24px;
}
</style>
