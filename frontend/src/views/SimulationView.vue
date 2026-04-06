<template>
  <div class="sim-view">
    <TopNav show-links :simulation-id="simulationId" active-step="Simulation" />
    <SideNav current-step="simulation" :completed-steps="['graph', 'env']" @navigate="handleNav" />

    <div class="sim-view__content">
      <header class="sim-view__header">
        <div style="display:flex;align-items:center;gap:.75rem;margin-bottom:.75rem">
          <span class="step-tag">Step 03</span>
          <span class="sim-view__sep">—</span>
          <span class="label-sm" style="color:var(--text-muted)">Live Simulation Run</span>
        </div>
        <h1 class="display-lg" style="color:var(--text-primary)">Simulation</h1>
        <p style="color:var(--text-muted);font-size:.9375rem;max-width:560px;line-height:1.7;margin-top:.5rem">
          Dual-platform autonomous agent simulation running across Info Plaza and Topic Community environments.
        </p>
      </header>

      <Step3Simulation
        :simulation-id="simulationId"
        @completed="onSimulationComplete"
      />
    </div>
  </div>
</template>

<script setup>
import { useRouter } from 'vue-router'
import TopNav from '../components/TopNav.vue'
import SideNav from '../components/SideNav.vue'
import Step3Simulation from '../components/Step3Simulation.vue'

const props = defineProps({ simulationId: String })
const router = useRouter()

function onSimulationComplete() {
  router.push(`/report/${props.simulationId}-report`)
}

function handleNav(step) {
  if (step === 'report') router.push(`/report/${props.simulationId}-report`)
}
</script>

<style scoped>
.sim-view { min-height: 100vh; background: var(--surface); }
.sim-view__content {
  margin-left: 240px;
  padding: 100px 2.5rem 4rem;
  min-height: 100vh;
}
.sim-view__header { margin-bottom: 2rem; }
.sim-view__sep { color: var(--text-muted); font-size: .75rem; }
</style>
