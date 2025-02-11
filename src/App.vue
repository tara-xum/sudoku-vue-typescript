<script setup lang="ts">
import { ref, Ref } from 'vue'
import BaseRadioGroup from './components/BaseRadioGroup.vue';
import BoardComponent from './components/BoardComponent.vue';

export interface MappedSudokuBoardCell {
  value: number;
  givenCell: boolean;
}

export interface DifficultyLevel {
  value: string;
  label: string;
}

interface UnMappedSudokuBoard {
  "board": number[][]
}

interface BoardError {
  row: number;
  col: number;
  value: number
}

interface InputEventTargetValue {
  target: { value: number }
}

const emptyBoard: UnMappedSudokuBoard = {
  "board": [
    [0,0,0,0,0,0,0,0,0],
    [0,0,0,0,0,0,0,0,0],
    [0,0,0,0,0,0,0,0,0],
    [0,0,0,0,0,0,0,0,0],
    [0,0,0,0,0,0,0,0,0],
    [0,0,0,0,0,0,0,0,0],
    [0,0,0,0,0,0,0,0,0],
    [0,0,0,0,0,0,0,0,0],
    [0,0,0,0,0,0,0,0,0]
  ]
}


const difficultyLevels: DifficultyLevel[] = [
  {value: "easy", label: "Easy"}, 
  {value: "medium", label: "Medium"}, 
  {value: "hard", label: "Hard"}, 
  {value: "random", label: "Random"}
]
const difficultyChoice = ref("")
const unmappedSudokuBoard: Ref<UnMappedSudokuBoard> = ref<UnMappedSudokuBoard>({"board": [[]]})
const mappedSudokuBoard: Ref<MappedSudokuBoardCell[][]> = ref<MappedSudokuBoardCell[][]>([])

const checkingForValidity = ref(false)
const loadingValidation = ref(false)
const boardErrors: Ref<BoardError[]> | Ref<null> = ref(null)
const locallySolveSudokuBoard = ref()
const endMsg = ref("")
const sudokuBoardValidityCheckMsg = ref("")
const sodukoIsValid = ref(false)
const isSolvedSudoku = ref(false)
const showSolution = ref(false)

async function getNewSudokuBoard() {
  sodukoIsValid.value = false
  isSolvedSudoku.value = false
  showSolution.value = false
  checkingForValidity.value = false
  loadingValidation.value = false
  boardErrors.value = []
  endMsg.value = ""
  locallySolveSudokuBoard.value = null
  sudokuBoardValidityCheckMsg.value = ""

  const newBoard = await getSudokuBoard(difficultyChoice.value)

  unmappedSudokuBoard.value = newBoard
  mappedSudokuBoard.value = mapSudokuBoardToAddData(newBoard.board)
}

function mapSudokuBoardToAddData(sudokuBoard: number[][]) {
  return sudokuBoard.map((sudokuLine: number[]) => {
      return sudokuLine.map((sudokuCell: number) =>  {
        return {
          value: sudokuCell,
          givenCell: sudokuCell !== 0 ? true : false 
        } 
      })
    })
}

async function getSudokuBoard(data: string) {
  const chosenDifficulty = data;
  const gameServiceUrl = 'https://sugoku.onrender.com'
  const returnedBoardResponse = await fetch(
    `${gameServiceUrl}/board?difficulty=${chosenDifficulty}`, {
    method: "GET", 
  })
  const sudokuBoard = await returnedBoardResponse.json()
  return sudokuBoard
}

function validateInput(event: InputEventTargetValue, row: number, col: number) {
  const value = event.target.value;

  if (!/^[1-9]$/.test(value.toString())) {
    alert('the inputed value is not a number from 1 to 9')
    event.target.value = 0;
    mappedSudokuBoard.value[row][col].value = 0
  }

  mappedSudokuBoard.value[row][col].value = event.target.value
}

function checkBoardValidity() {
  if(!locallySolveSudokuBoard.value) {
    solveBoardLocally()
  }
  boardErrors.value = [];
    for (let row = 0; row < 9; row++) {
      for (let col = 0; col < 9; col++) {
        if (
          !mappedSudokuBoard.value[row][col].value
          && mappedSudokuBoard.value[row][col].value 
          != locallySolveSudokuBoard.value[row][col]) {
          boardErrors.value.push({ row, col, value: mappedSudokuBoard.value[row][col].value }); // Store incorrect cell positions
        }
      }
    }
}

function solveSudoku(board: number[][]) {
    const size = 9;
    const emptyCell = findEmptyCell(board);
    if (!emptyCell) {
        return true;
    }
    const [row, col] = emptyCell;
    for (let num = 1; num <= size; num++) {
        if (isValidMove(board, row, col, num)) {
            board[row][col] = num;

            if (solveSudoku(board)) {
                return true;
            }

            board[row][col] = 0;
        }
    }

    return false;
}

function findEmptyCell(board: number[][]) {
    for (let row = 0; row < 9; row++) {
        for (let col = 0; col < 9; col++) {
            if (board[row][col] === 0) {
                return [row, col];
            }
        }
    }
    return null;
}

function isValidMove(board: number[][], row: number, col: number, num: number) {
    return !usedInRow(board, row, num) &&
           !usedInCol(board, col, num) &&
           !usedInBox(board, row - (row % 3), col - (col % 3), num);
}

function usedInRow(board: number[][], row: number, num: number) {
    return board[row].includes(num);
}

function usedInCol(board: number[][], col: number, num: number) {
    return board.some((row: number[]) => row[col] === num);
}

function usedInBox(board: number[][], boxStartRow: number, boxStartCol: number, num: number) {
    for (let i = 0; i < 3; i++) {
        for (let j = 0; j < 3; j++) {
            if (board[boxStartRow + i][boxStartCol + j] === num) {
                return true;
            }
        }
    }
    return false;
}

function showSolveBoard() {
  if(!locallySolveSudokuBoard.value) {
    solveBoardLocally()
  } 

  closeCheckingForValidityMsgToTryAgain()
  showSolution.value = true
}

function solveBoardLocally() {
  isSolvedSudoku.value = solveSudoku(unmappedSudokuBoard.value.board)

  locallySolveSudokuBoard.value = unmappedSudokuBoard.value.board

  if (isSolvedSudoku) {
    endMsg.value = "Solved Sudoku!"
  } else {
    endMsg.value = "No solution exists."
  }
}

function validateFilledBoard() {
  loadingValidation.value = true
  checkingForValidity.value = true

  checkBoardValidity()

  if(!boardErrors.value.length) {
    sudokuBoardValidityCheckMsg.value = "Your Sudoku Board is Valid! Congrats!"
    sodukoIsValid.value = true
  } else {
    sudokuBoardValidityCheckMsg.value = "Unfortunately, your Sudoku board is not valid. Try again!"
  }
  loadingValidation.value = false
}

function closeCheckingForValidityMsgToTryAgain() {
  checkingForValidity.value = false
}

function closeCheckingForValidityMsgToPlayAgain() {
  checkingForValidity.value = false
  sodukoIsValid.value = false
}

async function closeMsgToPlayAgain() {
  checkingForValidity.value = false
  sodukoIsValid.value = false
  showSolution.value = false
  getNewSudokuBoard()
}

</script>

<template>
  <h1>Sudoku</h1>
  
  <div>
    <div>Choose a difficulty: </div>
    <BaseRadioGroup 
      v-model="difficultyChoice" 
      :options="difficultyLevels" 
      name="difficultyLevelChoice" 
      :vertical="false"
      @update:modelValue=getNewSudokuBoard()
    />
  </div>

  <div v-if=mappedSudokuBoard.length>
    <div class="separator-line"></div>
    <p>Your chosen level of difficulty: {{ difficultyChoice }}</p>
    <BoardComponent 
      :editable="true" 
      :board="mappedSudokuBoard"
      @updated-input="validateInput"
    />


    <div class="action-buttons-container">
      <button v-if="!loadingValidation" @click="validateFilledBoard()">Validate</button>
      <button v-else>Loading...</button>
      <button @click="showSolveBoard()">Solve</button>
    </div>
    <div v-if="checkingForValidity">
      <p>
        {{ sudokuBoardValidityCheckMsg }}
      </p>  
      <button v-if="!sodukoIsValid" @click="closeCheckingForValidityMsgToTryAgain">
        Got it! I'll try again
      </button>
      <button v-else @click="closeCheckingForValidityMsgToPlayAgain()">Play again</button>
    </div>
  </div>

  <div v-if="showSolution && !checkingForValidity">
    <hr> 
    <h2><span class="solution-word">THE </span>Solution</h2>

    <p>{{ endMsg }}</p>

    <BoardComponent 
      v-if="locallySolveSudokuBoard && showSolution"
      :editable="false"
      :board="locallySolveSudokuBoard"
    />

    <button @click="closeMsgToPlayAgain()">Let me try again</button>
  </div>

</template>
<style scoped>

button {
  margin: 2rem 0;
  background-color: #772251;
  border-radius: 1rem;
  padding: 1rem 2rem;
  color: white;
  font-weight: 800;
}

.action-buttons-container button:nth-child(2) {
  margin: 0 1rem;
}

.solution-word {
  color: #772251
}

.separator-line {
  margin: 1rem 0 0 0;
  border-top: 0.2rem solid #772251;
}
</style>
