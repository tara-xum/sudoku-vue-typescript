<script setup lang="ts">
import type { MappedSudokuBoardCell } from '../App.vue'

defineProps<{
    editable: Boolean
    board: MappedSudokuBoardCell[][]
}>();

const emits = defineEmits(["updatedInput"])

</script>
<template>
    <div v-if="editable">
      <div
        class="row" 
        v-for="(row, rowIndex) in board" 
        :key="rowIndex"
        >
        <div 
          class="cell"
          v-for="(cell, colIndex) in row"
          :key="colIndex"
          :class="{
            'bold-right-border': (colIndex + 1) % 3 === 0, /* Thick right border every 3rd column */
            'bold-bottom-border': (rowIndex + 1) % 3 === 0, /* Thick bottom border every 3rd row */
            'bold-left-border': colIndex === 0, /* Thick left border */
            'bold-top-border': rowIndex === 0  /* Thick top border */
          }"
        >
          <input
            v-if="!cell.givenCell && !!editable"
            v-model="board[rowIndex][colIndex].value"
            maxlength="1"
            @input="$emit('updatedInput', $event, rowIndex, colIndex)"
          />
          <span 
            v-else 
            :disabled="cell.givenCell" 
            :class="{'given': cell.givenCell }"
          >
            {{ cell.value }}
          </span>
        </div>
      </div>
    </div>
    <div v-else>
      <div
        class="row" 
        v-for="(row, rowIndex) in board" 
        :key="rowIndex"
        >
        <div 
          class="cell"
          v-for="(cell, colIndex) in row"
          :key="colIndex"
          :class="{
            'bold-right-border': (colIndex + 1) % 3 === 0, /* Thick right border every 3rd column */
            'bold-bottom-border': (rowIndex + 1) % 3 === 0, /* Thick bottom border every 3rd row */
            'bold-left-border': colIndex === 0, /* Thick left border */
            'bold-top-border': rowIndex === 0  /* Thick top border */
          }"
        >
          <span 
            :disabled="true" 
            class="given"
          >
            {{ cell }}
          </span>
        </div>
      </div>
    </div>
</template>
<style scoped>
.row {
  display: flex;
}

.cell {
  width: 40px;
  height: 40px;
  display: flex;
  justify-content: center;
  align-items: center;
  border: 1px solid #ddd;
}

.cell input {
  width: 100%;
  text-align: center;
  font-size: 18px;
  border: none;
 }

.cell .given  {
  justify-content: center;
  display: flex;
  align-items: center;
  background-color: lightgray;
  width: 100%;
  height: 100%;
}

/* Borders for 3x3 boxes */
.bold-right-border {
  border-right: 3px solid black;
}

.bold-bottom-border {
  border-bottom: 3px solid black;
}

/* Outer border */
.bold-left-border {
  border-left: 3px solid black;
}

.bold-top-border {
  border-top: 3px solid black;
}
 </style>
