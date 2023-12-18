<template>
  <div class="infoModal__Wrapper" v-if="infoModal">
    <div class="infoModal">
      <p>
        {{ endMessage }}
      </p>
      <button @click="startGame">Restart</button>
    </div>
  </div>
  <div>
    <button @click="startGame" class="startGame" v-if="!player.inplay">Start</button>
    <div class="gameOptions">
      <p>
        Set difficulty:
      </p>
      <div>
        <span>Speed: {{ player.speed }}</span>
        <button @click="player.speed++">+</button>
        <button @click="player.speed > 1 ? player.speed-- : player.speed = 1">-</button>
      </div>
      <br>
      <div>
        <span>Wall width:</span>
        <button @click="increaseWidth">+</button>
        <button @click="decreaseWidth">-</button>
      </div>
    </div>
    <div class="gamearea" ref="gamearea">
      <span class="border" v-for="i in 4"></span>
      <div class="player" ref="character"></div>
      <div class="block" v-for="item in 2" ref="block"></div>
      <div class="finish" ref="finish"></div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const infoModal = ref(false)
const endMessage = ref("You lost!")
const character = ref(null)
const finish = ref(null)
const block = ref(null)
const gamearea = ref(null)
const keys = ref({})
const player = ref({
  speed: 10,
  inplay: null,
})

onMounted(() =>
  document.addEventListener("keydown", pressOn),
  document.addEventListener("keyup", pressOff)
)

function pressOn(e) {
  e.preventDefault()
  let tempKey = e.key
  keys.value[tempKey] = true;
}

function pressOff(e) {
  e.preventDefault()
  let tempKey = e.key
  keys.value[tempKey] = false;
}

function startGame() {
  character.value.style.top = 10 + "px"
  character.value.style.left = 10 + "px"
  player.value.inplay = true
  window.requestAnimationFrame(move)
  player.value.x = character.value.offsetLeft
  player.value.y = character.value.offsetTop
  infoModal.value = false;
}

function move() {
  if (player.value.inplay) {
    if (keys.value.ArrowUp && player.value.y > 0) {
      player.value.y -= player.value.speed;
    }
    if (keys.value.ArrowDown && player.value.y < (gamearea.value.offsetHeight)) {
      player.value.y += player.value.speed;
    }
    if (keys.value.ArrowLeft && player.value.x > 0) {
      player.value.x -= player.value.speed;
    }
    if (keys.value.ArrowRight && player.value.x < (gamearea.value.offsetWidth)) {
      player.value.x += player.value.speed;
    }
    character.value.style.left = player.value.x + "px"
    character.value.style.top = player.value.y + "px"
    window.requestAnimationFrame(move)
    let block = document.querySelectorAll(".block")
    let border = document.querySelectorAll(".border")
    block.forEach(function (item) {
      if (isColliding(character.value, item)) {
        endGame()
        endMessage.value = "You lost!"
        player.value.inplay = false
      }
    })
    border.forEach(function (item) {
      if (isColliding(character.value, item)) {
        endGame()
        endMessage.value = "You lost!"
        player.value.inplay = false
      }
    })

    if (isColliding(character.value, finish.value)) {
      endGame()
      endMessage.value = "You won!"
      player.value.inplay = false
    }
  }
}

function increaseWidth() {
  block.value.forEach(function(item) {
    item.style.width = `${item.offsetWidth + 10}px`
  })
}

function decreaseWidth() {
  block.value.forEach(function(item) {
    item.style.width = `${item.offsetWidth - 10}px`
  })
}

function endGame() {
  infoModal.value = true;
}

function isColliding(a, b) {
  let aRect = a.getBoundingClientRect()
  let bRect = b.getBoundingClientRect()
  return !(
    (aRect.bottom < bRect.top) ||
    (aRect.top > bRect.bottom) ||
    (aRect.right < bRect.left) ||
    (aRect.left > bRect.right)
  )
}
</script>
