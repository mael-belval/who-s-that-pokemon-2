<template>
  <dialog ref="dialogRef">
    <div class="modal-body">
      <h1 v-if="!guessed">Who's that Pokemon?</h1>
      <h1 v-else>
        {{ isCorrect ? 'Bonne réponse!' : `Non, c'était ${pokemonName}` }}
      </h1>
      <div class="guess-form" v-if="!guessed">
        <input
          class="guess-value"
          v-model="guessValue"
          type="text"
          @keyup.enter="$emit('submit', guessValue)"
          placeholder="Enter Pokémon name"
          autofocus
        />
        <button class="guess-submit" @click="$emit('submit', guessValue)">Envoyer</button>
      </div>
      <div v-else>
        <button class="guess-submit" @click="$emit('play-again')">Rejouer</button>
      </div>
    </div>
  </dialog>
</template>

<script setup lang="ts">
import { ref, watch, onMounted } from 'vue'
const props = defineProps<{
  guessed: boolean
  isCorrect: boolean
  show: boolean
  pokemonName: string
}>()
const emit = defineEmits(['submit', 'play-again', 'dialog-mounted'])
const guessValue = ref('')
const dialogRef = ref<HTMLDialogElement | null>(null)

watch(
  () => props.show,
  (val) => {
    if (dialogRef.value) {
      if (val) {
        dialogRef.value.showModal()
        document.body.classList.add('modal-open')
      } else {
        dialogRef.value.close()
        document.body.classList.remove('modal-open')
      }
    }
  },
)

onMounted(() => {
  emit('dialog-mounted', dialogRef.value)
})
</script>
