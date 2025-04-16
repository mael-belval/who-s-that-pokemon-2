<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'

const pokemon = ref<{ name: string; image: string } | null>(null)
const guess = ref('')
const showModal = ref(false)
const guessed = ref(false)
const isCorrect = ref(false)
const loading = ref(true)
const dialogRef = ref<HTMLDialogElement | null>(null)

async function fetchRandomPokemon() {
  loading.value = true
  guessed.value = false
  isCorrect.value = false
  showModal.value = false
  guess.value = ''
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
}

function playAgain() {
  fetchRandomPokemon()
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
        <h1 v-if="!guessed">Name that Pokemon!</h1>
        <h1 v-else>
          {{
            isCorrect
              ? 'Correct!'
              : `No, it was ${pokemon?.name.charAt(0).toUpperCase() + pokemon?.name.slice(1)}`
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
          <button class="guess-submit" @click="submitGuess">Submit</button>
        </div>
        <div v-else>
          <button class="guess-submit" @click="playAgain">Play Again</button>
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
