<script setup>
import { onMounted } from 'vue'

onMounted(() => {
  const MDCTopAppBar = window.mdc.topAppBar.MDCTopAppBar

  new MDCTopAppBar(
    document.querySelector('.mdc-top-app-bar')
  )
})
</script>

<template>
  <header class="mdc-top-app-bar">
    <div class="mdc-top-app-bar__row">
      <section class="mdc-top-app-bar__section mdc-top-app-bar__section--align-start">
        <RouterLink to="/home">
          <button
            class="material-icons mdc-top-app-bar__navigation-icon mdc-icon-button"
            aria-label="Open navigation home"
          >
            home
          </button>
        </RouterLink>

        <span class="mdc-top-app-bar__title">
          Pokedex
        </span>
      </section>
    </div>
  </header>

  <router-view />
</template>

<style scoped>
:global(body) {
  margin: 0;
}
</style>