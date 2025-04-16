<template>
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
        v-if="image"
        :src="image"
        alt="Pokemon silhouette"
        class="pokemon-image"
        :style="guessed ? 'filter:brightness(1)' : 'filter:brightness(0)'"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
defineProps<{ image: string | null; guessed: boolean }>()
</script>
