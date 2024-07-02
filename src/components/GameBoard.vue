<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { BehaviorSubject } from 'rxjs'

const BOARD_SIZE = 12
const MINE_COUNT = 10

interface Cell {
  isBoom: boolean
  isRevealed: boolean
  isFlagged: boolean
  adjacentMinesAmt: number
}

interface GameState {
  board: Cell[]
  gameStatus: 'playing' | 'won' | 'lost'
}

const board = ref<Cell[]>([])
const gameStatus = ref<'playing' | 'won' | 'lost'>('playing')
const gameState$ = new BehaviorSubject<GameState>({ board: [], gameStatus: 'playing' })

const totalCells = computed(() => BOARD_SIZE * BOARD_SIZE)

const initializeBoard = (): void => {
  const newBoard = Array(totalCells.value)
    .fill(null)
    .map(() => ({
      isBoom: false,
      isRevealed: false,
      isFlagged: false,
      adjacentMinesAmt: 0
    }))

  // Place mines
  let minesPlaced = 0
  while (minesPlaced < MINE_COUNT) {
    const idx = Math.floor(Math.random() * totalCells.value)
    if (!newBoard[idx].isBoom) {
      newBoard[idx].isBoom = true
      minesPlaced++
    }
  }

  // Calculate adjacent mines
  for (let i = 0; i < totalCells.value; i++) {
    if (!newBoard[i].isBoom) {
      newBoard[i].adjacentMinesAmt = getAdjacentMines(newBoard, i)
    }
  }

  board.value = newBoard
  gameState$.next({ board: newBoard, gameStatus: 'playing' })
}

const getAdjacentMines = (board: Cell[], idx: number): number => {
  const adjacentIndices = getAdjacentIndices(idx)
  return adjacentIndices.filter((i) => board[i].isBoom).length
}

const getAdjacentIndices = (idx: number): number[] => {
  const row = Math.floor(idx / BOARD_SIZE)
  const col = idx % BOARD_SIZE
  const indices: number[] = []

  for (let r = Math.max(0, row - 1); r <= Math.min(BOARD_SIZE - 1, row + 1); r++) {
    for (let c = Math.max(0, col - 1); c <= Math.min(BOARD_SIZE - 1, col + 1); c++) {
      if (r !== row || c !== col) {
        indices.push(r * BOARD_SIZE + c)
      }
    }
  }

  return indices
}

const revealCell = (idx: number): void => {
  if (gameStatus.value !== 'playing' || board.value[idx].isRevealed || board.value[idx].isFlagged) {
    return
  }

  const newBoard = [...board.value]
  newBoard[idx].isRevealed = true

  if (newBoard[idx].isBoom) {
    gameStatus.value = 'lost'
    revealAllMines(newBoard)
  } else if (newBoard[idx].adjacentMinesAmt === 0) {
    revealAdjacentCells(newBoard, idx)
  }

  if (checkWinCondition(newBoard)) {
    gameStatus.value = 'won'
  }

  board.value = newBoard
  gameState$.next({ board: newBoard, gameStatus: gameStatus.value })
}

const revealAdjacentCells = (board: Cell[], idx: number): void => {
  const adjacentIndices = getAdjacentIndices(idx)
  adjacentIndices.forEach((i) => {
    if (!board[i].isRevealed && !board[i].isFlagged) {
      board[i].isRevealed = true
      if (board[i].adjacentMinesAmt === 0) {
        revealAdjacentCells(board, i)
      }
    }
  })
}

const toggleFlag = (idx: number): void => {
  if (gameStatus.value !== 'playing' || board.value[idx].isRevealed) {
    return
  }

  const newBoard = [...board.value]
  newBoard[idx].isFlagged = !newBoard[idx].isFlagged
  board.value = newBoard
  gameState$.next({ board: newBoard, gameStatus: gameStatus.value })
}

const revealAllMines = (board: Cell[]): void => {
  board.forEach((cell) => {
    if (cell.isBoom) {
      cell.isRevealed = true
    }
  })
}

const checkWinCondition = (board: Cell[]): boolean => {
  return board.every((cell) => cell.isRevealed || cell.isBoom)
}

const resetGame = (): void => {
  gameStatus.value = 'playing'
  initializeBoard()
}

const getCellClass = (cell: Cell): string => {
  if (!cell.isRevealed) {
    return cell.isFlagged ? 'bg-yellow-500' : 'bg-gray-400 hover:bg-gray-500'
  }
  if (cell.isBoom) {
    return 'bg-red-500'
  }
  return 'bg-white'
}

const getCellContent = (cell: Cell): string => {
  if (!cell.isRevealed) {
    return cell.isFlagged ? '🚩' : ''
  }
  if (cell.isBoom) {
    return '💣'
  }
  return cell.adjacentMinesAmt > 0 ? cell.adjacentMinesAmt.toString() : ''
}

onMounted(() => {
  initializeBoard()
})
</script>

<template>
  <div class="flex flex-col items-center justify-center min-h-screen">
    <h1 class="text-4xl font-bold mb-4">Minesweeper</h1>
    <div class="grid grid-cols-8 gap-1 bg-gray-300 p-2 rounded">
      <div
        v-for="(cell, idx) in board"
        :key="idx"
        @click="revealCell(idx)"
        @contextmenu.prevent="toggleFlag(idx)"
        class="w-10 h-10 flex items-center justify-center cursor-pointer select-none"
        :class="getCellClass(cell)"
      >
        {{ getCellContent(cell) }}
      </div>
    </div>
    <div class="mt-4 text-xl" v-if="gameStatus !== 'playing'">
      {{ gameStatus === 'won' ? 'You Won!' : 'Game Over!' }}
    </div>
    <button
      @click="resetGame"
      class="mt-4 px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600"
    >
      Reset Game
    </button>
  </div>
</template>
