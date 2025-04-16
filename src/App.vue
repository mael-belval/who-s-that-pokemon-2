<script setup lang="ts">
import { ref, onMounted, watch, nextTick } from 'vue'
let ConfettiGenerator: unknown

const pokemon = ref<{ name: string; image: string } | null>(null)
const guess = ref('')
const showModal = ref(false)
const guessed = ref(false)
const isCorrect = ref(false)
const loading = ref(true)
const dialogRef = ref<HTMLDialogElement | null>(null)
const confettiRef = ref<HTMLCanvasElement | null>(null)
let confettiInstance: unknown = null

async function fetchRandomPokemon() {
  loading.value = true
  guessed.value = false
  isCorrect.value = false
  showModal.value = false
  guess.value = ''
  stopConfetti()
  const id = Math.floor(Math.random() * 898) + 1
  const res = await fetch(`https://pokeapi.co/api/v2/pokemon/${id}`)
  const data = await res.json()
  const image = data.sprites.other['official-artwork'].front_default
  pokemon.value = { name: data.name, image }
  loading.value = false
  setTimeout(() => {
    showModal.value = true
  }, 3000)
}

function submitGuess() {
  guessed.value = true
  isCorrect.value = guess.value.trim().toLowerCase() === pokemon.value?.name.toLowerCase()
  if (isCorrect.value) {
    showConfetti()
  }
}

function playAgain() {
  fetchRandomPokemon()
}

function showConfetti() {
  nextTick(async () => {
    if (!ConfettiGenerator) {
      ConfettiGenerator = (await import('confetti-js')).default
    }
    if (confettiInstance) {
      confettiInstance.clear()
      confettiInstance = null
    }
    if (confettiRef.value) {
      confettiInstance = new ConfettiGenerator({
        target: confettiRef.value,
        max: 120,
        size: 1.2,
        animate: true,
        props: ['circle', 'square', 'triangle', 'line'],
        clock: 25,
        width: window.innerWidth,
        height: window.innerHeight,
        start_from_edge: true,
        respawn: false,
      })
      confettiInstance.render()
    }
  })
}

function stopConfetti() {
  if (confettiInstance) {
    confettiInstance.clear()
    confettiInstance = null
  }
}

onMounted(() => {
  fetchRandomPokemon()
})

watch(showModal, (val) => {
  if (dialogRef.value) {
    if (val) {
      dialogRef.value.showModal()
      document.body.classList.add('modal-open')
    } else {
      dialogRef.value.close()
      document.body.classList.remove('modal-open')
    }
  }
})
</script>

<template>
  <div class="container">
    <canvas
      ref="confettiRef"
      v-show="isCorrect && guessed"
      style="
        position: fixed;
        left: 0;
        top: 0;
        width: 100vw;
        height: 100vh;
        z-index: 99999;
        pointer-events: none;
      "
    ></canvas>
    <div
      class="pokemon-container"
      style="
        position: relative;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
      "
    >
      <div
        style="
          position: relative;
          width: 300px;
          height: 300px;
          display: flex;
          align-items: center;
          justify-content: center;
        "
      >
        <img
          src="/src/assets/whos-that-pokemon.png"
          class="whos-that-pokemon"
          alt="Who's that Pokémon background"
          style="
            position: absolute;
            left: 50%;
            top: 50%;
            width: 900px;
            height: 900px;
            transform: translate(-50%, -50%);
            z-index: 1;
            pointer-events: none;
          "
        />
        <img
          v-if="pokemon && pokemon.image"
          :src="pokemon.image"
          alt="Pokemon silhouette"
          class="pokemon-image"
          :style="guessed ? 'filter:brightness(1)' : 'filter:brightness(0)'"
        />
      </div>
    </div>
    <div v-if="loading" style="color: white; text-align: center; margin-top: 2rem">Loading...</div>
    <dialog ref="dialogRef" id="modal-1">
      <div class="modal-body">
        <h1 v-if="!guessed">Who's that Pokemon?</h1>
        <h1 v-else>
          {{
            isCorrect
              ? 'Bonne réponse!'
              : `Non, c'était ${pokemon?.name.charAt(0).toUpperCase() + pokemon?.name.slice(1)}`
          }}
        </h1>
        <div class="guess-form" v-if="!guessed">
          <input
            class="guess-value"
            v-model="guess"
            type="text"
            @keyup.enter="submitGuess"
            placeholder="Enter Pokémon name"
            autofocus
          />
          <button class="guess-submit" @click="submitGuess">Envoyer</button>
        </div>
        <div v-else>
          <button class="guess-submit" @click="playAgain">Rejouer</button>
        </div>
      </div>
    </dialog>
  </div>
</template>

<style scoped>
.container {
  font-family: 'Helvetica';
  max-width: 700px;
  margin: auto;
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
}

.pokemon-container {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.pokemon-image {
  transition: 0.3s;
  width: 300px;
  height: 300px;
  position: absolute;
  left: -15%;
  top: -30%;
  z-index: 2;
}

dialog {
  border: none;
  border-radius: 10px;
  padding: 0;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.3);
  background: transparent;
  z-index: 10000;
  margin: 10vh auto;
}
.modal-body {
  background: white;
  border-radius: 10px;
  padding: 2rem;
  text-align: center;
  min-width: 300px;
}
</style>
