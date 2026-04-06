<template>
  <div class="report-view">
    <TopNav show-links active-step="Reports" />
    <SideNav current-step="report" :completed-steps="['graph', 'env', 'simulation']" @navigate="handleNav" />

    <div class="report-view__content">
      <Step4Report
        :report-id="reportId"
        @completed="onReportComplete"
      />
    </div>
  </div>
</template>

<script setup>
import { useRouter } from 'vue-router'
import TopNav from '../components/TopNav.vue'
import SideNav from '../components/SideNav.vue'
import Step4Report from '../components/Step4Report.vue'

const props = defineProps({ reportId: String })
const router = useRouter()

function onReportComplete() {
  router.push(`/interaction/${props.reportId}`)
}

function handleNav(step) {
  if (step === 'simulation') router.back()
}
</script>

<style scoped>
.report-view { min-height: 100vh; background: var(--surface); }
.report-view__content {
  margin-left: 240px;
  padding: 100px 2.5rem 4rem;
  min-height: 100vh;
}
</style>
