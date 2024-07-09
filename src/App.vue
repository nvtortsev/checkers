<script setup>
import { reactive, ref } from 'vue'
const fields = [...'ABCDEFGH'].flatMap((letter) => [...Array(8)].map((_, i) => `${letter}${i + 1}`))
const audio = ref(null)
// Отображение клеток
const determineColorForCss = (field) => {
  const [letter, number] = field.split('')
  const letters = 'ABCDEFGH'
  const isDark = (letters.indexOf(letter) + +number) % 2 === 1
  return isDark ? 'bg-stone-600 hover:bg-stone-500' : 'bg-yellow-200 hover:bg-amber-100'
}

// Определение текущих позиций шашек
const initialPositions = {
  A1: 'black',
  A3: 'black',
  A5: 'black',
  A7: 'black',
  B2: 'black',
  B4: 'black',
  B6: 'black',
  B8: 'black',
  C1: 'black',
  C3: 'black',
  C5: 'black',
  C7: 'black',
  F2: 'white',
  F4: 'white',
  F6: 'white',
  F8: 'white',
  G1: 'white',
  G3: 'white',
  G5: 'white',
  G7: 'white',
  H2: 'white',
  H4: 'white',
  H6: 'white',
  H8: 'white'
}
const checkerPositions = reactive({ ...initialPositions })

const currentPlayer = ref('white')
const progressGame = reactive([])
const moveChecker = reactive({
  from: '',
  to: ''
})

const validateMove = (from, to) => {
  if (from && to) {
    const fromLetter = from[0]
    const fromNumber = parseInt(from[1])
    const toLetter = to[0]
    const toNumber = parseInt(to[1])
    const letterDiff = Math.abs(fromLetter.charCodeAt(0) - toLetter.charCodeAt(0))
    const numberDiff = Math.abs(fromNumber - toNumber)

    const enemyChecker = currentPlayer.value === 'white' ? 'black' : 'white'

    // Проверка на рубку (ход через клетку диагонально)
    if (letterDiff === 2 && numberDiff === 2) {
      const middleLetterCode = (fromLetter.charCodeAt(0) + toLetter.charCodeAt(0)) / 2
      const middleNumber = (fromNumber + toNumber) / 2
      const middleLetter = String.fromCharCode(middleLetterCode)

      const betweenField = middleLetter + middleNumber
      if (checkerPositions[betweenField] === enemyChecker) {
        delete checkerPositions[betweenField]
        progressGame.push({
          step: progressGame.length + 1,
          player: currentPlayer.value,
          move_from: from,
          move_to: to,
          move_type: 'рубка'
        })
        return true
      }
    }

    // Обычный ход
    if (letterDiff === 1 && numberDiff === 1) {
      progressGame.push({
        step: progressGame.length + 1,
        player: currentPlayer.value,
        move_from: from,
        move_to: to,
        move_type: 'ход'
      })
      return true
    }
  }
  return false
}

const togglePlayer = () => {
  currentPlayer.value = currentPlayer.value === 'white' ? 'black' : 'white'
  moveChecker.from = ''
  moveChecker.to = ''
}
const gameOver = ref(false)
const winner = ref('')

const checkWinner = () => {
  const countChekers = Object.values(checkerPositions)
  const countWhite = countChekers.filter((checker) => checker === 'white').length
  const countBlack = countChekers.filter((checker) => checker === 'black').length

  if (countWhite === 0) {
    gameOver.value = true
    winner.value = 'Черные победили!'
    audio.value.play()
  } else if (countBlack === 0) {
    gameOver.value = true
    winner.value = 'Белые победили!'
    audio.value.play()
  }
}

const handleOnField = (field) => {
  if (checkerPositions[field] === currentPlayer.value) {
    moveChecker.from = field
  } else if (!checkerPositions[field] && moveChecker.from) {
    moveChecker.to = field
    if (validateMove(moveChecker.from, moveChecker.to)) {
      checkerPositions[moveChecker.to] = currentPlayer.value
      delete checkerPositions[moveChecker.from]
      togglePlayer()
      checkWinner()
    }
    moveChecker.from = ''
    moveChecker.to = ''
  }
}
</script>

<template>
  <audio ref="audio" class="hidden">
    <source src="/audio.mp3" type="audio/mpeg" />
  </audio>
  <div class="absolute z-20 w-full h-full" v-if="winner">
    <img
      class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-80"
      src="/winner.png"
      alt="winner"
    />
    <div class="center border p-8 bg-fuchsia-50 rounded-xl absolute font-medium text-6xl">
      <h1 class="font-medium text-6xl">
        {{ winner }}
      </h1>
      <h2 class="text-2xl text-center mt-5">Чтобы начать заного F5</h2>
    </div>
  </div>
  <!-- Экран  -->
  <div
    :class="`relative flex justify-around items-start p-16 w-full h-screen bg-slate-700 z-10 ${winner.length ? 'blur-sm' : ''}`"
  >
    <!-- Правила -->
    <div class="flex w-max justify-center items-center text-2xl text-white pt-20">
      <div class="flex flex-col gap-10 border-2 border-white p-4 rounded-lg">
        <h2 class="flex gap-5 items-center text-yellow-200">
          Правила для игры в шашки <img src="/checkers.png" alt="checkers" />
        </h2>
        <ul class="text-xl flex flex-col gap-3 text-yellow-100">
          <li>Игру начинает игрок с белыми шашками.</li>
          <li>
            Шашки перемещаются на одну клетку вперед или назад <br />
            по диагонали в любом направлении.
          </li>
          <li>
            Если на пути шашки находится другая шашка этого <br />игрока, игрок не может сделать
            шаг.
          </li>
          <li>
            Если на пути шашки находится шашка второго игрока, <br />
            первый игрок может ее срубить.
          </li>
          <li>Рубить шашки можно как и спереди, так и сзади.</li>
          <li>
            Если ты выбрал шашку, а у нее не оказалось <br />
            пути для рубки или хода, ты должен выбрать <br />
            другую шашку, нажав на клетку, перед <br />
            заблокированной шашкой.
          </li>
        </ul>
      </div>
    </div>

    <div class="flex flex-col gap-10">
      <!-- Информация -->
      <h1 class="text-center text-3xl text-yellow-200">
        Ход делает:
        <span
          :class="`border-none rounded-lg p-1 ${currentPlayer === 'white' ? 'text-black bg-slate-400' : 'text-white bg-slate-800'}`"
          >{{ currentPlayer === 'white' ? 'Белый' : 'Черный' }}</span
        >
      </h1>

      <!-- Доска -->
      <div class="w-max grid grid-cols-8 border-4 border-slate-400">
        <div v-for="(field, index) in fields" :key="index">
          <!-- Клетка -->
          <div
            @click="handleOnField(field)"
            :class="`relative w-20 h-20 cursor-pointer ${determineColorForCss(field)}`"
          >
            {{ field }}
            <img
              v-if="checkerPositions[field]"
              class="center absolute"
              :src="
                checkerPositions[field] === 'white' ? '/white_checker.png' : '/black_checker.png'
              "
              :alt="checkerPositions[field]"
            />
          </div>
        </div>
      </div>
    </div>

    <!-- Ход событий -->
    <div
      class="w-1/4 h-4/5 text-2xl flex flex-col gap-8 justify-between text-white border-2 mt-20 p-5 border-white rounded-lg z-10 overflow-hidden"
    >
      <h2 class="text-center">Текущий прогресс игры</h2>
      <div class="overflow-y-scroll flex-1">
        <div
          class="w-full flex items-center justify-center gap-8"
          v-for="progress in progressGame"
          :key="progress.step"
        >
          <span>{{ progress.move_type }}</span>
          <img
            width="50"
            :src="progress.player === 'white' ? '/white_player.png' : '/black_player.png'"
            alt=""
          />
          <p>{{ progress.move_from }}</p>
          <img width="45" src="/arrow.png" alt="" />
          <p>{{ progress.move_to }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.center {
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
.right {
  left: 43%;
  top: 30%;
  transform: translate(100%, -100%);
}
</style>
