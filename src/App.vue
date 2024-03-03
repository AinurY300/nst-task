<template>
  <main>
    <div class="switch">
      <p :class="['switch-button', (showMonths ? '' : 'active')]" @click="showMonths = false">Все года</p>
      <p :class="['switch-button', (showMonths ? 'active' : '')]" @click="showMonths = true">Месяца</p>
    </div>
    <NstSlider
      v-model="value"
      :show-months="showMonths" 
      :min-year="2014" 
      :max-year="2021" 
      :selected-start-year="2020"
      :selected-start-month="5"
      :selected-end-year="2017"
      :selected-end-month="9"
    />
  </main>
</template>

<script setup lang="ts">
import NstSlider from './components/NstSlider.vue'
import { ref } from 'vue'

const showMonths = ref(false)
const value = ref()

</script>

<style scoped>
main {
  margin: 0 auto;
  max-width: 980px;
  box-sizing: border-box;
  padding: 40px;
}

.switch {
  color: var(--blue-text);
  margin-bottom: 64px;
  display: inline-block;
}

.switch-button {
  opacity: .6;
  cursor: pointer;
  text-decoration: underline;
}
.switch-button.active {
  opacity: 1;
  font-weight: 700;
  text-decoration: none;
}
</style>