<template>
  <div class="main-view">
    <TopNav show-links :project-id="projectId" active-step="Graph Build" />
    <SideNav :current-step="currentStep" :completed-steps="completedSteps" @navigate="navigateTo" />

    <div class="main-view__content">
      <!-- Page Header -->
      <header class="main-view__header">
        <div class="main-view__breadcrumb">
          <span class="step-tag">Step 0{{ stepIndex + 1 }}</span>
          <span class="main-view__breadcrumb-sep">—</span>
          <span class="label-sm" style="color:var(--text-muted)">{{ stepTitles[currentStep] }}</span>
        </div>
        <h1 class="display-lg main-view__title">{{ stepHeadings[currentStep] }}</h1>
        <p class="main-view__subtitle">{{ stepSubtitles[currentStep] }}</p>
      </header>

      <!-- Step Components -->
      <transition name="step" mode="out-in">
        <Step1GraphBuild
          v-if="currentStep === 'graph'"
          :project-data="projectData"
          @completed="onStepCompleted('graph', $event)"
        />
        <Step2EnvSetup
          v-else-if="currentStep === 'env'"
          :project-data="projectData"
          @completed="onStepCompleted('env', $event)"
        />
        <div v-else class="main-view__placeholder">
          <span class="material-symbols-outlined" style="font-size:48px;color:var(--text-muted)">account_tree</span>
          <p class="main-view__placeholder-text">Select a step from the sidebar to begin</p>
          <button class="btn-primary" @click="navigateTo('graph')">
            Begin Graph Build
            <span class="material-symbols-outlined" style="font-size:18px">arrow_forward</span>
          </button>
        </div>
      </transition>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import TopNav from '../components/TopNav.vue'
import SideNav from '../components/SideNav.vue'
import Step1GraphBuild from '../components/Step1GraphBuild.vue'
import Step2EnvSetup from '../components/Step2EnvSetup.vue'

const props = defineProps({ projectId: String })
const router = useRouter()

const currentStep = ref('graph')
const completedSteps = ref([])
const projectData = ref({ project_id: props.projectId, graph_id: null, ontology: null })

const stepOrder = ['graph', 'env', 'simulation', 'report']
const stepIndex = computed(() => stepOrder.indexOf(currentStep.value))

const stepTitles = {
  graph: 'Graph Build & Ontology',
  env: 'Environment Configuration',
  simulation: 'Simulation Run',
  report: 'Report Generation',
}
const stepHeadings = {
  graph: 'Graph Build',
  env: 'Env Setup',
  simulation: 'Simulation',
  report: 'Report',
}
const stepSubtitles = {
  graph: 'Extract ontology structures, entity types, and relationships from your reality seeds.',
  env: 'Define simulation parameters, agent personas, and recommendation logic.',
  simulation: 'Launch parallel autonomous agents across Info Plaza and Topic Community.',
  report: 'Generate AI-powered predictive intelligence from simulation data.',
}

function navigateTo(step) {
  // Allow navigation to completed steps or current step
  if (step === currentStep.value || completedSteps.value.includes(step) || stepOrder.indexOf(step) <= stepOrder.indexOf(currentStep.value) + 1) {
    currentStep.value = step
  }
}

function onStepCompleted(step, data) {
  if (!completedSteps.value.includes(step)) completedSteps.value.push(step)
  if (data) Object.assign(projectData.value, data)

  // Auto-advance
  const next = stepOrder[stepOrder.indexOf(step) + 1]
  if (next) {
    if (next === 'simulation') {
      router.push(`/simulation/${projectData.value.simulation_id || 'demo-sim-001'}`)
    } else {
      currentStep.value = next
    }
  }
}

onMounted(() => {
  // Initialize with graph step
  currentStep.value = 'graph'
})
</script>

<style scoped>
.main-view {
  min-height: 100vh;
  background: var(--surface);
}

.main-view__content {
  margin-left: 240px;
  padding-top: 60px;
  min-height: 100vh;
  padding: 100px 2.5rem 4rem;
}

.main-view__header {
  margin-bottom: 2.5rem;
}

.main-view__breadcrumb {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  margin-bottom: 0.875rem;
}

.main-view__breadcrumb-sep {
  color: var(--text-muted);
  font-size: 0.75rem;
}

.main-view__title {
  color: var(--text-primary);
  margin-bottom: 0.75rem;
}

.main-view__subtitle {
  color: var(--text-muted);
  font-size: 0.9375rem;
  max-width: 640px;
  line-height: 1.7;
}

.main-view__placeholder {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 400px;
  gap: 1.25rem;
  background: var(--surface-container);
  border-radius: var(--radius-xl);
  border: 1px solid var(--ghost-border);
}

.main-view__placeholder-text {
  color: var(--text-muted);
  font-size: 0.9375rem;
}

/* Step transitions */
.step-enter-active,
.step-leave-active {
  transition: opacity 200ms ease, transform 200ms ease;
}
.step-enter-from { opacity: 0; transform: translateX(12px); }
.step-leave-to   { opacity: 0; transform: translateX(-8px); }
</style>
