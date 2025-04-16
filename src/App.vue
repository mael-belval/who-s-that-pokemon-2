<script setup lang="ts">
import { ref, onMounted } from 'vue'
import PokemonShadow from './components/PokemonShadow.vue'
import GuessModal from './components/GuessModal.vue'
import ConfettiCanvas from './components/ConfettiCanvas.vue'

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
  // 494 parce que je maitrise mal après diamant et perle
  const id = Math.floor(Math.random() * 494) + 1
  const res = await fetch(`https://pokeapi.co/api/v2/pokemon/${id}`)
  const data = await res.json()
  const image = data.sprites.other['official-artwork'].front_default
  pokemon.value = { name: data.name, image }
  loading.value = false
  setTimeout(() => {
    showModal.value = true
  }, 3000)
}

function handleGuess(submittedGuess: string) {
  guessed.value = true
  guess.value = submittedGuess
  isCorrect.value = submittedGuess.trim().toLowerCase() === pokemon.value?.name.toLowerCase()
}

function playAgain() {
  fetchRandomPokemon()
}

function handleDialogMounted(ref: HTMLDialogElement) {
  dialogRef.value = ref
}

onMounted(() => {
  fetchRandomPokemon()
})
</script>

<template>
  <div class="container">
    <ConfettiCanvas :show="isCorrect && guessed" />
    <PokemonShadow :image="pokemon?.image || ''" :guessed="guessed" />
    <div v-if="loading" style="color: white; text-align: center; margin-top: 2rem">Loading...</div>
    <GuessModal
      :show="showModal"
      :guessed="guessed"
      :isCorrect="isCorrect"
      :pokemonName="pokemon ? pokemon.name.charAt(0).toUpperCase() + pokemon.name.slice(1) : ''"
      @submit="handleGuess"
      @play-again="playAgain"
      @dialog-mounted="handleDialogMounted"
    />
  </div>
</template>

<style>
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
