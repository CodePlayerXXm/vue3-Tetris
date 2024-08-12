<script setup lang="ts" generic="T extends any, O extends any">
defineOptions({
  name: 'IndexPage',
})

onMounted(init)

const mesh = ref()

const square = ref([
  [1, 1, 1],
  [0, 1, 0],
])

let _startLine = 0

let currentIndex = [
  [0, square.value.length],
  [4, square.value[0].length],
]

let speed = 1000

let interval: number | NodeJS.Timeout

let isLocked = false

let isStoped = false

function fillMesh() {
  mesh.value = Array.from({ length: 20 }, () => Array.from({ length: 10 }).fill(0))
}

function render() {
  interval = setInterval(() => {
    falling()
  }, speed)
}

function checkStop() {
  const isLastRow = currentIndex[0][0] + currentIndex[0][1] === mesh.value.length
  const isOnOtherSquare =
  isStoped = isLastRow || isOnOtherSquare
  if (isStoped) {
    // clearInterval(interval)
    createNewSquare()
  }
}

function createNewSquare() {
  _startLine = 0
  currentIndex = [
    [0, square.value.length],
    [4, square.value[0].length],
  ]
}

function clearTetromino(line: number) {
  if (line < 0)
    return
  let count = 1
  for (let meshLineIndex = line; meshLineIndex >= 0; meshLineIndex--) {
    if (count > square.value.length)
      break
    const meshLine = mesh.value[meshLineIndex]
    const squareLine = square.value[square.value.length - count]

    for (let index = 0; index < squareLine.length; index++) {
      if (squareLine[index] === 1)
        meshLine[currentIndex[1][0] + index] = 0
    }
    count++
  }
}

function falling() {
  isLocked = true

  clearTetromino(_startLine - 1)
  let _count = 1

  for (let meshLineIndex = _startLine; meshLineIndex >= 0; meshLineIndex--) {
    if (_count > square.value.length)
      break
    currentIndex[0][0] = (_startLine - square.value.length + 1) < 0 ? 0 : (_startLine - square.value.length + 1)
    const meshLine = mesh.value[meshLineIndex]
    const squareLine = square.value[square.value.length - _count]

    for (let index = 0; index < squareLine.length; index++)
      meshLine[currentIndex[1][0] + index] = squareLine[index]

    _count++
  }
  _startLine++
  isLocked = false
  checkStop()
}

function bindKey() {
  document.addEventListener('keydown', (e: KeyboardEvent) => {
    const currentRowIndex = currentIndex[0][0]
    const currentRowLength = currentIndex[0][1]

    const currentColIndex = currentIndex[1][0]
    const currentColLength = currentIndex[1][1]

    if (isLocked || isStoped)
      return
    if (e.code === 'ArrowLeft') {
      for (let rowIndex = 0; rowIndex < currentRowLength; rowIndex++) {
        for (let colIndex = 0; colIndex < currentColLength; colIndex++) {
          if (currentColIndex + colIndex === 0)
            return
          mesh.value[currentRowIndex + rowIndex][currentColIndex + colIndex - 1] = mesh.value[currentRowIndex + rowIndex][currentColIndex + colIndex]
          mesh.value[currentRowIndex + rowIndex][currentColIndex + colIndex] = 0
        }
      }
      currentIndex[1][0] = currentColIndex - 1
    }
    if (e.code === 'ArrowRight') {
      for (let rowIndex = 0; rowIndex < currentRowLength; rowIndex++) {
        for (let colIndex = currentColLength - 1; colIndex >= 0; colIndex--) {
          if (currentColIndex + colIndex === mesh.value[0].length - 1)
            return
          mesh.value[currentRowIndex + rowIndex][currentColIndex + colIndex + 1] = mesh.value[currentRowIndex + rowIndex][currentColIndex + colIndex]
          mesh.value[currentRowIndex + rowIndex][currentColIndex + colIndex] = 0
        }
      }
      currentIndex[1][0] = currentColIndex + 1
    }
    if (e.code === 'ArrowDown') {
      if (speed === 10)
        return

      changeSpeed(10)
    }
  })
  document.addEventListener('keyup', (e: KeyboardEvent) => {
    if (isStoped)
      return
    if (e.code === 'ArrowDown') {
      if (speed === 1000)
        return
      changeSpeed(1000)
    }
  })
}

function changeSpeed(newSpeed: number) {
  speed = newSpeed
  clearInterval(interval)
  render()
}

function init() {
  fillMesh()
  render()
  bindKey()
}
</script>

<template>
  <div>
    <div v-for="(item, index) in mesh" :key="index" style="display: flex;">
      <div
        v-for="(_item, _index) in item" :key="_index" style="width: 20px; height:20px; border:1px solid"
        :style="{ background: _item ? 'black' : null }"
      />
    </div>
  </div>
</template>
