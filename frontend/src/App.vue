<template>
  <router-view v-slot="{ Component }">
    <transition name="page" mode="out-in">
      <component :is="Component" />
    </transition>
  </router-view>
</template>

<script setup>
// Global router + design system loaded via main.js
</script>

<style>
@import './assets/design-tokens.css';

/* ── Page Transitions ── */
.page-enter-active,
.page-leave-active {
  transition: opacity 200ms ease, transform 200ms ease;
}
.page-enter-from {
  opacity: 0;
  transform: translateY(8px);
}
.page-leave-to {
  opacity: 0;
  transform: translateY(-4px);
}

/* ── Selection ── */
::selection {
  background: rgba(255, 181, 158, 0.25);
  color: var(--primary);
}
</style>
