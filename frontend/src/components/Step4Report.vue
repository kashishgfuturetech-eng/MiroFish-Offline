<template>
  <div class="report">
    <!-- Top action bar -->
    <div class="report__topbar">
      <div>
        <div class="step-tag">Step 04 — Report Generation</div>
        <h1 class="display-lg report__title" style="margin-top:.5rem">Prediction Report</h1>
      </div>
      <div class="report__topbar-actions">
        <button class="btn-secondary" @click="downloadReport" :disabled="!isComplete">
          <span class="material-symbols-outlined" style="font-size:16px">download</span>
          Export PDF
        </button>
        <button v-if="isComplete" class="btn-primary" @click="$emit('completed')">
          Open Interaction
          <span class="material-symbols-outlined" style="font-size:18px">chat</span>
        </button>
      </div>
    </div>

    <div class="report__split">
      <!-- ── LEFT: Report Document ── -->
      <div class="report__doc card" ref="docRef">
        <!-- Waiting State -->
        <div v-if="!reportOutline && !isGenerating" class="report__waiting">
          <div class="report__waiting-rings">
            <div class="report__ring" style="animation-delay:0ms"></div>
            <div class="report__ring" style="animation-delay:200ms"></div>
            <div class="report__ring" style="animation-delay:400ms"></div>
          </div>
          <span class="report__waiting-text">Waiting for Report Agent…</span>
          <button class="btn-primary" @click="beginReport">
            <span class="material-symbols-outlined" style="font-size:18px">auto_awesome</span>
            Generate Report
          </button>
        </div>

        <!-- Generating state (outline building) -->
        <div v-else-if="isGenerating && !reportOutline" class="report__waiting">
          <div class="ai-chip">
            <span class="ai-chip-dot"></span>
            <span class="label-sm" style="color:var(--secondary)">Building report outline…</span>
          </div>
        </div>

        <!-- Report Content -->
        <div v-else-if="reportOutline" class="report__content">
          <!-- Report Header Block -->
          <div class="report__header-block">
            <div class="report__meta-row">
              <span class="chip chip-orange">Prediction Report</span>
              <span class="report__id font-mono label-sm">ID: {{ reportId }}</span>
              <span class="report__date label-sm">{{ reportDate }}</span>
            </div>
            <h2 class="report__main-title font-headline">{{ reportOutline.title }}</h2>
            <p class="report__summary">{{ reportOutline.summary }}</p>
            <div class="section-divider"></div>
          </div>

          <!-- Sections -->
          <div class="report__sections">
            <div
              v-for="(section, idx) in reportOutline.sections"
              :key="idx"
              class="report__section"
              :class="{
                'report__section--active':    currentSectionIdx === idx + 1,
                'report__section--completed': isSectionDone(idx + 1),
                'report__section--pending':   !isSectionDone(idx + 1) && currentSectionIdx !== idx + 1,
              }"
            >
              <!-- Section Header -->
              <div
                class="report__section-header"
                :class="{'report__section-header--clickable': isSectionDone(idx + 1)}"
                @click="isSectionDone(idx + 1) && toggleCollapse(idx)"
              >
                <span class="report__section-num font-mono">{{ String(idx + 1).padStart(2, '0') }}</span>
                <h3 class="report__section-title headline-md">{{ section.title }}</h3>
                <div class="report__section-header-right">
                  <span v-if="isSectionDone(idx + 1)" class="chip chip-green" style="font-size:.6rem">
                    <span class="material-symbols-outlined" style="font-size:11px">check</span>
                  </span>
                  <div v-else-if="currentSectionIdx === idx + 1" class="ai-chip" style="padding:.2rem .625rem">
                    <span class="ai-chip-dot"></span>
                    <span class="label-sm" style="color:var(--secondary)">Writing…</span>
                  </div>
                  <span v-else class="chip chip-muted" style="font-size:.6rem">Pending</span>
                  <span
                    v-if="isSectionDone(idx + 1)"
                    class="material-symbols-outlined report__collapse-icon"
                    :class="{'report__collapse-icon--open': !collapsedSet.has(idx)}"
                  >expand_more</span>
                </div>
              </div>

              <!-- Section Body -->
              <div class="report__section-body" v-show="!collapsedSet.has(idx)">
                <!-- Generated Content -->
                <div
                  v-if="generatedSections[idx + 1]"
                  class="report__generated"
                  v-html="renderMarkdown(generatedSections[idx + 1])"
                ></div>
                <!-- Writing State -->
                <div v-else-if="currentSectionIdx === idx + 1" class="report__writing">
                  <div class="report__writing-cursor font-mono">{{ writingPreview }}<span class="report__cursor-blink">|</span></div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- ── RIGHT: Workflow Timeline ── -->
      <div class="report__timeline card">
        <span class="label-sm" style="color:var(--text-muted);margin-bottom:1.25rem;display:block">Report Workflow</span>

        <div class="report__timeline-list">
          <div
            v-for="(step, i) in timelineSteps"
            :key="i"
            class="report__timeline-item"
            :class="{
              'report__timeline-item--done':    step.status === 'done',
              'report__timeline-item--active':  step.status === 'active',
              'report__timeline-item--pending': step.status === 'pending',
            }"
          >
            <div class="report__timeline-connector" v-if="i > 0"></div>
            <div class="report__timeline-dot">
              <span v-if="step.status === 'done'" class="material-symbols-outlined" style="font-size:14px">check</span>
              <div v-else-if="step.status === 'active'" class="spinner" style="width:12px;height:12px;border-width:1.5px"></div>
              <span v-else class="report__timeline-pending-dot"></span>
            </div>
            <div class="report__timeline-content">
              <div class="report__timeline-title">{{ step.title }}</div>
              <div class="report__timeline-desc label-sm">{{ step.desc }}</div>
              <div v-if="step.status !== 'pending'" class="report__timeline-time label-sm">{{ step.time }}</div>
            </div>
          </div>
        </div>

        <!-- Section Progress -->
        <div v-if="reportOutline" class="report__section-progress">
          <div class="section-divider"></div>
          <span class="label-sm" style="color:var(--text-muted)">Section Progress</span>
          <div class="report__progress-track">
            <div class="report__progress-fill" :style="`width:${sectionProgressPct}%`"></div>
          </div>
          <span class="label-sm" style="color:var(--text-muted)">
            {{ Object.keys(generatedSections).length }} / {{ reportOutline.sections.length }} sections
          </span>
        </div>

        <!-- Tokens used -->
        <div v-if="tokensUsed" class="report__tokens card-nested" style="margin-top:1rem">
          <span class="label-sm" style="color:var(--text-muted)">Tokens Used</span>
          <div class="font-mono" style="color:var(--primary);font-size:1.125rem;font-weight:700">{{ tokensUsed.toLocaleString() }}</div>
          <div class="label-sm" style="color:var(--text-muted)">claude-3.5-sonnet</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({ reportId: { type: String, default: 'REF-2024-X92' } })
const emit = defineEmits(['completed'])

const docRef = ref(null)
const isGenerating = ref(false)
const isComplete = ref(false)
const reportOutline = ref(null)
const currentSectionIdx = ref(0)
const generatedSections = ref({})
const collapsedSet = ref(new Set())
const writingPreview = ref('')
const tokensUsed = ref(null)
const reportDate = new Date().toLocaleDateString('en-US', { year: 'numeric', month: 'long', day: 'numeric' })

const demoOutline = {
  title: 'Emergent Social Dynamics: Predictive Intelligence Report',
  summary: 'Analysis of autonomous agent behavior across Info Plaza and Topic Community reveals significant clustering around key opinion leaders, with 73% of information cascades originating from fewer than 8% of agents.',
  sections: [
    { title: 'Executive Summary', desc: 'High-level findings overview' },
    { title: 'Information Cascade Patterns', desc: 'How information spreads' },
    { title: 'Key Opinion Leader Identification', desc: 'Influence mapping' },
    { title: 'Sentiment Trajectory Analysis', desc: 'Emotional arc over time' },
    { title: 'Cross-Platform Dynamics', desc: 'Info Plaza vs Topic Community' },
    { title: 'Predictive Scenarios', desc: 'Future state projections' },
    { title: 'Recommendations', desc: 'Actionable intelligence' },
  ],
}

const demoContent = [
  'The simulation reveals a **highly centralized information ecosystem** where content from a small cluster of high-degree nodes rapidly propagates across the entire network. Key findings include:\n\n- **73%** of viral cascades originate from ≤8% of agents\n- Average propagation time from source to saturation: **4.2 rounds**\n- Cross-platform amplification factor: **2.3×**\n\nThese dynamics suggest systemic vulnerability to coordinated narrative injection.',
  'Information cascades follow a **power-law distribution** characteristic of scale-free networks. Three distinct cascade archetypes were observed:\n\n1. **Flash cascades** — rapid, high-intensity, short-lived (avg. 2.1 rounds)\n2. **Slow burns** — gradual adoption over 15–30 rounds with sustained engagement\n3. **Boomerang patterns** — initial rejection followed by delayed mass adoption\n\nThe boomerang pattern is strongly correlated with counter-narrative injection at round 3–5.',
  'Network centrality analysis identified **12 Key Opinion Leaders (KOLs)** with disproportionate influence:\n\n| Agent | Degree | Betweenness | Platform |\n|-------|--------|-------------|----------|\n| Agent_007 | 0.89 | 0.73 | Both |\n| Agent_023 | 0.81 | 0.61 | Info Plaza |\n| Agent_041 | 0.77 | 0.58 | Topic |\n\nKOLs exhibit **adaptive resonance** — they shift messaging style within 3 rounds of detecting audience drift.',
  'Sentiment analysis across 1,200 simulation rounds reveals a **three-phase emotional arc**:\n\n- **Phase 1 (Rounds 1–300):** Cautious optimism (+0.23 valence)\n- **Phase 2 (Rounds 301–800):** Polarization spike (σ = 0.67)\n- **Phase 3 (Rounds 801–1200):** Synthetic consensus emergence (+0.41 valence)\n\nNegative sentiment events cluster around **round 420–480**, correlating with information scarcity signals.',
  'Cross-platform comparison reveals **asymmetric influence dynamics**:\n\n- Info Plaza drives **initial narrative framing** (first-mover advantage)\n- Topic Community provides **validation and depth** (second-mover amplification)\n- Cross-platform agents (active on both) serve as critical **bridge nodes**\n\nDisconnecting bridge nodes reduces cascade velocity by **61%** but increases narrative polarization by **34%**.',
  'Based on emergent patterns, three predictive scenarios are modeled:\n\n**Scenario A — Status Quo (+18% probability):** Gradual consensus formation with KOL-led narrative stabilization by round 1,400.\n\n**Scenario B — Disruption (+45% probability):** New high-degree node injection triggers cascade reset and re-polarization.\n\n**Scenario C — Fragmentation (+37% probability):** Echo chamber formation leads to parallel, non-intersecting information ecosystems.',
  'Strategic recommendations based on simulation intelligence:\n\n1. **Engage KOLs early** — Intervening before round 50 reduces cascade probability by 58%\n2. **Deploy bridge agents** — Cross-platform presence amplifies reach without additional nodes\n3. **Monitor Phase 2 triggers** — Sentiment dip at rounds 420–480 represents highest intervention ROI\n4. **Scenario B preparedness** — Maintain 15% agent reserve for rapid counter-narrative deployment',
]

function isSectionDone(idx) { return !!generatedSections.value[idx] }

const sectionProgressPct = computed(() => {
  if (!reportOutline.value) return 0
  return (Object.keys(generatedSections.value).length / reportOutline.value.sections.length) * 100
})

const timelineSteps = computed(() => {
  const steps = [
    { title: 'Collect Simulation Data', desc: 'Aggregate action logs', time: '0.8s', status: 'done' },
    { title: 'Build Report Outline', desc: 'Structure 7 sections', time: '1.2s', status: isGenerating.value || reportOutline.value ? 'done' : 'pending' },
    { title: 'Generate Section Content', desc: 'AI writes each section', time: isComplete.value ? '14.3s' : '…', status: isComplete.value ? 'done' : (reportOutline.value ? 'active' : 'pending') },
    { title: 'Render & Format', desc: 'Apply markdown + styling', time: isComplete.value ? '0.4s' : '—', status: isComplete.value ? 'done' : 'pending' },
  ]
  return steps
})

async function beginReport() {
  isGenerating.value = true
  await new Promise(r => setTimeout(r, 1500))
  reportOutline.value = demoOutline
  isGenerating.value = false
  await generateSections()
}

async function generateSections() {
  for (let i = 0; i < demoOutline.sections.length; i++) {
    currentSectionIdx.value = i + 1
    // Simulate writing preview
    const content = demoContent[i]
    writingPreview.value = ''
    for (let c = 0; c < Math.min(60, content.length); c++) {
      writingPreview.value += content[c]
      await new Promise(r => setTimeout(r, 18))
    }
    await new Promise(r => setTimeout(r, 600))
    generatedSections.value[i + 1] = content
    writingPreview.value = ''
    await new Promise(r => setTimeout(r, 200))
  }
  currentSectionIdx.value = 0
  isComplete.value = true
  tokensUsed.value = 8432
}

function toggleCollapse(idx) {
  const next = new Set(collapsedSet.value)
  if (next.has(idx)) next.delete(idx)
  else next.add(idx)
  collapsedSet.value = next
}

function renderMarkdown(text) {
  if (!text) return ''
  return text
    .replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')
    .replace(/\*(.*?)\*/g, '<em>$1</em>')
    .replace(/`(.*?)`/g, '<code>$1</code>')
    .replace(/^#{1,3} (.+)$/gm, '<h4 class="report-md-h">$1</h4>')
    .replace(/\n\n/g, '</p><p>')
    .replace(/^\| (.+) \|$/gm, (m) => {
      const cells = m.split('|').filter(c => c.trim()).map(c => `<td>${c.trim()}</td>`)
      return `<tr>${cells.join('')}</tr>`
    })
    .replace(/(<tr>.*<\/tr>)/gs, '<table class="report-md-table">$1</table>')
    .replace(/^- (.+)$/gm, '<li>$1</li>')
    .replace(/(<li>.*<\/li>)/gs, '<ul>$1</ul>')
    .replace(/^\d+\. (.+)$/gm, '<li>$1</li>')
    .replace(/\n/g, ' ')
}

function downloadReport() {
  // Placeholder — would trigger PDF export
  alert('PDF export would be triggered here via backend.')
}
</script>

<style scoped>
.report { display: flex; flex-direction: column; gap: 1.5rem; }

.report__topbar {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 1rem;
}

.report__title {
  color: var(--text-primary);
  margin-top: 0.5rem;
}

.report__topbar-actions {
  display: flex;
  gap: 0.75rem;
  align-items: center;
  flex-shrink: 0;
}

/* Split */
.report__split {
  display: grid;
  grid-template-columns: 1fr 280px;
  gap: 1.25rem;
  align-items: start;
}

/* Doc Panel */
.report__doc {
  min-height: 500px;
  overflow: hidden;
}

.report__waiting {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 400px;
  gap: 1.5rem;
  padding: 3rem;
}

.report__waiting-rings {
  position: relative;
  width: 60px;
  height: 60px;
}

.report__ring {
  position: absolute;
  inset: 0;
  border: 1.5px solid rgba(182, 196, 255, 0.3);
  border-top-color: var(--secondary);
  border-radius: 50%;
  animation: spin 1.4s linear infinite;
}
.report__ring:nth-child(2) { inset: 8px; border-width: 1px; opacity: .7; }
.report__ring:nth-child(3) { inset: 16px; border-width: 1px; opacity: .4; }

.report__waiting-text {
  color: var(--text-muted);
  font-size: 0.875rem;
}

/* Content */
.report__content { padding: 2rem; }

.report__header-block { margin-bottom: 1.5rem; }

.report__meta-row {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  margin-bottom: 1rem;
  flex-wrap: wrap;
}

.report__id { color: var(--text-muted); }
.report__date { color: var(--text-muted); margin-left: auto; }

.report__main-title {
  font-size: clamp(1.5rem, 3vw, 2rem);
  color: var(--text-primary);
  margin-bottom: 0.875rem;
  line-height: 1.2;
}

.report__summary {
  color: var(--text-secondary);
  font-size: 0.9375rem;
  line-height: 1.75;
}

/* Sections */
.report__sections { display: flex; flex-direction: column; gap: 0; }

.report__section {
  border-bottom: 1px solid var(--ghost-border);
  transition: background var(--duration-fast);
}
.report__section:last-child { border-bottom: none; }

.report__section--active { background: rgba(255,181,158,.03); }
.report__section--pending { opacity: 0.55; }

.report__section-header {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 1.125rem 0;
}
.report__section-header--clickable { cursor: pointer; }
.report__section-header--clickable:hover .report__section-title { color: var(--primary); }

.report__section-num {
  font-size: 1.25rem;
  font-weight: 700;
  color: var(--text-muted);
  opacity: 0.4;
  min-width: 32px;
}

.report__section-title {
  flex: 1;
  color: var(--text-primary);
  font-size: 1rem;
  transition: color var(--duration-fast);
}

.report__section-header-right {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  flex-shrink: 0;
}

.report__collapse-icon {
  font-size: 20px;
  color: var(--text-muted);
  transition: transform var(--duration-normal) var(--ease-out);
  transform: rotate(-90deg);
}
.report__collapse-icon--open { transform: rotate(0deg); }

/* Section Body */
.report__section-body { padding-bottom: 1.25rem; }

.report__generated {
  color: var(--text-secondary);
  font-size: 0.875rem;
  line-height: 1.8;
  padding-left: 2.5rem;
}

.report__writing {
  padding-left: 2.5rem;
  padding-top: 0.5rem;
}

.report__writing-cursor {
  color: var(--text-secondary);
  font-size: 0.8125rem;
  line-height: 1.7;
  white-space: pre-wrap;
}

.report__cursor-blink {
  color: var(--secondary);
  animation: blink 700ms step-end infinite;
}
@keyframes blink { 0%,100%{opacity:1} 50%{opacity:0} }

/* Timeline */
.report__timeline {
  padding: 1.5rem;
  position: sticky;
  top: 80px;
}

.report__timeline-list {
  display: flex;
  flex-direction: column;
  gap: 0;
}

.report__timeline-item {
  display: flex;
  gap: 0.875rem;
  position: relative;
  padding-bottom: 1.25rem;
}

.report__timeline-connector {
  position: absolute;
  left: 11px;
  top: -1.25rem;
  height: 1.25rem;
  width: 1px;
  background: var(--ghost-border);
}

.report__timeline-dot {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 1px solid var(--ghost-border);
  background: var(--surface-container);
  margin-top: 1px;
}
.report__timeline-item--done .report__timeline-dot {
  background: rgba(134, 239, 172, 0.15);
  border-color: rgba(134, 239, 172, 0.3);
  color: #86EFAC;
}
.report__timeline-item--active .report__timeline-dot {
  border-color: rgba(182,196,255,.3);
  box-shadow: var(--shadow-glow-ai);
}

.report__timeline-pending-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: var(--surface-container-highest);
}

.report__timeline-content { flex: 1; min-width: 0; }

.report__timeline-title {
  font-size: 0.8125rem;
  font-weight: 600;
  color: var(--text-primary);
  margin-bottom: 0.125rem;
}
.report__timeline-item--pending .report__timeline-title { color: var(--text-muted); }

.report__timeline-desc { color: var(--text-muted); }

.report__timeline-time { color: var(--primary); margin-top: 0.25rem; }

/* Progress */
.report__section-progress {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.report__progress-track {
  height: 4px;
  background: var(--surface-container-high);
  border-radius: var(--radius-full);
  overflow: hidden;
}

.report__progress-fill {
  height: 100%;
  background: var(--gradient-primary);
  border-radius: var(--radius-full);
  transition: width 400ms var(--ease-out);
}

.report__tokens { text-align: center; }

/* Markdown rendered content */
:deep(.report__generated strong) { color: var(--text-primary); font-weight: 700; }
:deep(.report__generated code) {
  font-family: var(--font-mono);
  font-size: 0.8em;
  background: var(--surface-container-high);
  padding: 0 4px;
  border-radius: 3px;
  color: var(--secondary);
}
:deep(.report__generated ul) { padding-left: 1.25rem; margin: 0.5rem 0; }
:deep(.report__generated li) { margin-bottom: 0.25rem; }
:deep(.report__generated table) { width: 100%; border-collapse: collapse; margin: 0.75rem 0; font-size: .8125rem; }
:deep(.report__generated td) {
  padding: 0.375rem 0.625rem;
  border: 1px solid var(--ghost-border);
  color: var(--text-secondary);
}
:deep(.report__generated tr:first-child td) {
  background: var(--surface-container-high);
  color: var(--text-primary);
  font-weight: 600;
}
:deep(.report-md-h) {
  font-family: var(--font-headline);
  color: var(--text-primary);
  font-size: .9375rem;
  margin: .75rem 0 .375rem;
}

@media (max-width: 1100px) {
  .report__split { grid-template-columns: 1fr; }
}
</style>
