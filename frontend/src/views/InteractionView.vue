<template>
  <div class="interact-view">
    <TopNav show-links />
    <SideNav current-step="report" :completed-steps="['graph', 'env', 'simulation', 'report']" @navigate="handleNav" />

    <div class="interact-view__content">
      <header class="interact-view__header">
        <div style="display:flex;align-items:center;gap:.75rem;margin-bottom:.75rem">
          <span class="step-tag">Step 05</span>
          <span style="color:var(--text-muted);font-size:.75rem">—</span>
          <span class="label-sm" style="color:var(--text-muted)">Simulation Oracle</span>
        </div>
        <h1 class="display-lg" style="color:var(--text-primary)">Interaction</h1>
        <p style="color:var(--text-muted);font-size:.9375rem;max-width:600px;line-height:1.7;margin-top:.5rem">
          Query the simulated world. Ask the Oracle about agent behaviors, emergent patterns, and predictive scenarios derived from your simulation.
        </p>
      </header>

      <Step5Interaction :report-id="reportId" />
    </div>
  </div>
</template>

<script setup>
import { useRouter } from 'vue-router'
import TopNav from '../components/TopNav.vue'
import SideNav from '../components/SideNav.vue'
import Step5Interaction from '../components/Step5Interaction.vue'

const props = defineProps({ reportId: String })
const router = useRouter()

function handleNav(step) {
  if (step === 'report') router.back()
}
</script>

<style scoped>
.interact-view { min-height: 100vh; background: var(--surface); }
.interact-view__content {
  margin-left: 240px;
  padding: 80px 2.5rem 2rem;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}
.interact-view__header { margin-bottom: 1.5rem; flex-shrink: 0; }
</style>
