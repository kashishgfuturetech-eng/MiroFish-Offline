<template>
  <div class="home">
    <!-- Background orbs -->
    <div class="orb orb-orange" style="width:600px;height:600px;top:-200px;right:-100px;opacity:0.6;"></div>
    <div class="orb orb-blue" style="width:500px;height:500px;bottom:-150px;left:-100px;opacity:0.5;"></div>

    <!-- Nav -->
    <TopNav />

    <main class="home__main">
      <!-- ── Hero Section ── -->
      <section class="home__hero">
        <div class="home__hero-left">
          <div class="home__tag-row animate-fade-up" style="animation-delay:0ms">
            <span class="chip chip-orange">Offline Multi-Agent Simulation Engine</span>
            <span class="home__version label-sm">/ v0.1-preview</span>
          </div>

          <h1 class="home__headline display-lg animate-fade-up" style="animation-delay:80ms">
            Upload Any Document<br>
            <span class="home__headline-accent">Predict What Happens Next</span>
          </h1>

          <p class="home__desc animate-fade-up" style="animation-delay:160ms">
            From a single document, <strong class="home__strong">MiroFish Offline</strong> extracts
            reality seeds and builds a parallel world of
            <span class="home__accent">autonomous AI agents</span> — running entirely on your machine.
            Inject variables, observe emergent behavior, and find
            <code class="home__code">"local optima"</code> in complex social dynamics.
          </p>

          <div class="home__slogan animate-fade-up" style="animation-delay:200ms">
            Your data never leaves your machine. The future is simulated locally<span class="home__cursor">_</span>
          </div>

          <div class="home__cta-row animate-fade-up" style="animation-delay:240ms">
            <button class="btn-primary home__cta-btn" @click="scrollToConsole">
              <span class="material-symbols-outlined" style="font-size:18px">rocket_launch</span>
              Start Engine
            </button>
            <a
              href="https://github.com/nikmcfly/Predly"
              target="_blank"
              class="btn-secondary"
            >
              <span class="material-symbols-outlined" style="font-size:16px">open_in_new</span>
              View Github
            </a>
          </div>
        </div>

        <div class="home__hero-right animate-fade-up" style="animation-delay:120ms">
          <!-- Stats cluster -->
          <div class="home__stat-cluster">
            <div class="home__stat-card card">
              <div class="home__stat-orb orb-blue" style="width:80px;height:80px;top:-20px;right:-20px;opacity:0.5;"></div>
              <span class="label-sm" style="color:var(--text-muted)">Mode</span>
              <div class="home__stat-value font-headline">Free</div>
              <div class="home__stat-label">Runs on your hardware</div>
            </div>
            <div class="home__stat-card card">
              <div class="home__stat-orb orb-orange" style="width:80px;height:80px;top:-20px;right:-20px;opacity:0.5;"></div>
              <span class="label-sm" style="color:var(--text-muted)">Privacy</span>
              <div class="home__stat-value font-headline">Private</div>
              <div class="home__stat-label">100% offline, no cloud</div>
            </div>
            <div class="home__stat-card home__stat-card--wide card">
              <span class="label-sm" style="color:var(--text-muted)">System Status</span>
              <div style="display:flex;align-items:center;gap:.625rem;margin-top:.375rem">
                <span class="home__status-dot"></span>
                <div class="home__stat-value font-headline" style="font-size:1.25rem">Ready</div>
              </div>
              <div class="home__stat-label">Local prediction engine on standby</div>
            </div>
          </div>

          <!-- Workflow steps -->
          <div class="home__steps card">
            <div class="home__steps-header label-sm" style="color:var(--text-muted)">
              <span style="color:var(--primary)">◇</span> Workflow Sequence
            </div>
            <div class="home__steps-list">
              <div
                v-for="(step, i) in workflowSteps"
                :key="i"
                class="home__step"
                :style="`animation-delay:${(i + 5) * 60}ms`"
              >
                <span class="home__step-num font-mono">{{ step.num }}</span>
                <div class="home__step-body">
                  <div class="home__step-title">{{ step.title }}</div>
                  <div class="home__step-desc">{{ step.desc }}</div>
                </div>
                <span class="home__step-arrow material-symbols-outlined">arrow_forward</span>
              </div>
            </div>
          </div>
        </div>
      </section>

      <!-- ── Console Section ── -->
      <section class="home__console" ref="consoleRef">
        <div class="home__console-header">
          <div class="step-tag">01 / Reality Seeds</div>
          <span class="label-sm" style="color:var(--text-muted)">Supported: PDF, MD, TXT</span>
        </div>

        <div class="home__console-grid">
          <!-- Upload Zone -->
          <div class="home__upload-zone card" @dragover.prevent="dragging=true" @dragleave="dragging=false" @drop.prevent="handleDrop" @click="triggerFileInput" :class="{'home__upload-zone--drag': dragging, 'home__upload-zone--filled': files.length}">
            <input ref="fileInput" type="file" multiple accept=".pdf,.md,.txt" style="display:none" @change="handleFileSelect" />

            <div v-if="!files.length" class="home__upload-placeholder">
              <div class="home__upload-icon">
                <span class="material-symbols-outlined" style="font-size:32px;color:var(--text-muted)">cloud_upload</span>
              </div>
              <div class="home__upload-title">Drop reality seeds here</div>
              <div class="label-sm" style="color:var(--text-muted)">PDF, MD, TXT — your local data stays local</div>
            </div>

            <div v-else class="home__file-list">
              <div v-for="(f, i) in files" :key="i" class="home__file-item">
                <span class="material-symbols-outlined" style="font-size:16px;color:var(--secondary)">description</span>
                <span class="home__file-name font-mono">{{ f.name }}</span>
                <span class="home__file-size label-sm">{{ formatSize(f.size) }}</span>
                <button class="home__file-remove" @click.stop="removeFile(i)">
                  <span class="material-symbols-outlined" style="font-size:16px">close</span>
                </button>
              </div>
            </div>
          </div>

          <!-- Input + CTA -->
          <div class="home__input-side">
            <div class="home__divider-label label-sm">02 / Context Prompt</div>
            <div class="home__textarea-wrap">
              <textarea
                v-model="prompt"
                class="input-field home__textarea"
                rows="5"
                placeholder="Describe your simulation goal, target audience, or specific dynamics to observe…"
              ></textarea>
              <span class="home__model-badge font-mono label-sm">claude-3-5-sonnet · local</span>
            </div>

            <button
              class="home__start-btn btn-primary"
              :disabled="!canStart"
              @click="startEngine"
            >
              <span>START ENGINE</span>
              <span class="material-symbols-outlined" style="font-size:20px">arrow_forward</span>
            </button>
          </div>
        </div>
      </section>
    </main>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useRouter } from 'vue-router'
import TopNav from '../components/TopNav.vue'

const router = useRouter()
const consoleRef = ref(null)
const fileInput = ref(null)
const files = ref([])
const prompt = ref('')
const dragging = ref(false)

const canStart = computed(() => files.value.length > 0 || prompt.value.trim().length > 10)

const workflowSteps = [
  { num: '01', title: 'Graph Build',    desc: 'Extract ontology & knowledge graph from reality seeds' },
  { num: '02', title: 'Env Setup',      desc: 'Configure simulation parameters & agent personas' },
  { num: '03', title: 'Simulation',     desc: 'Run parallel autonomous agent interactions' },
  { num: '04', title: 'Report',         desc: 'AI-generated predictive intelligence report' },
  { num: '05', title: 'Interaction',    desc: 'Query the simulated world with your own prompts' },
]

function scrollToConsole() {
  consoleRef.value?.scrollIntoView({ behavior: 'smooth', block: 'start' })
}

function triggerFileInput() { fileInput.value?.click() }

function handleFileSelect(e) {
  const newFiles = Array.from(e.target.files)
  files.value = [...files.value, ...newFiles]
}

function handleDrop(e) {
  dragging.value = false
  const dropped = Array.from(e.dataTransfer.files).filter(f =>
    ['.pdf','.md','.txt'].some(ext => f.name.toLowerCase().endsWith(ext))
  )
  files.value = [...files.value, ...dropped]
}

function removeFile(i) { files.value.splice(i, 1) }

function formatSize(bytes) {
  if (bytes < 1024) return `${bytes}B`
  if (bytes < 1048576) return `${(bytes/1024).toFixed(1)}KB`
  return `${(bytes/1048576).toFixed(1)}MB`
}

function startEngine() {
  // Navigate to process view with placeholder projectId
  router.push(`/process/demo-project-001`)
}
</script>

<style scoped>
.home {
  min-height: 100vh;
  background: var(--surface);
  position: relative;
  overflow-x: hidden;
}

.home__main {
  max-width: 1360px;
  margin: 0 auto;
  padding: 100px 2.5rem 4rem;
  display: flex;
  flex-direction: column;
  gap: 5rem;
}

/* ── Hero ── */
.home__hero {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 4rem;
  align-items: start;
}

.home__tag-row {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 1.5rem;
}

.home__version {
  color: var(--text-muted);
}

.home__headline {
  color: var(--text-primary);
  margin-bottom: 1.5rem;
}

.home__headline-accent {
  background: var(--gradient-primary);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.home__desc {
  color: var(--text-secondary);
  font-size: 1rem;
  line-height: 1.85;
  max-width: 560px;
  margin-bottom: 1.5rem;
}

.home__strong { color: var(--text-primary); font-weight: 700; }

.home__accent {
  color: var(--primary);
  font-weight: 600;
}

.home__code {
  background: var(--surface-container-high);
  padding: 1px 6px;
  border-radius: 4px;
  font-family: var(--font-mono);
  font-size: 0.875em;
  color: var(--secondary);
  border: 1px solid var(--ghost-border);
}

.home__slogan {
  font-size: 1rem;
  font-weight: 600;
  color: var(--text-secondary);
  border-left: 3px solid var(--primary-container);
  padding-left: 1rem;
  margin-bottom: 2rem;
  line-height: 1.6;
}

.home__cursor {
  color: var(--primary);
  animation: blink 1s step-end infinite;
  font-weight: 700;
}
@keyframes blink { 0%,100%{opacity:1} 50%{opacity:0} }

.home__cta-row {
  display: flex;
  gap: 0.75rem;
  flex-wrap: wrap;
}

.home__cta-btn {
  padding: 0.75rem 1.75rem;
  font-size: 0.875rem;
  box-shadow: var(--shadow-glow-primary);
}

/* ── Hero Right ── */
.home__hero-right {
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
}

.home__stat-cluster {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 0.75rem;
}

.home__stat-card {
  padding: 1.25rem;
  overflow: hidden;
}

.home__stat-card--wide {
  grid-column: span 2;
}

.home__stat-orb {
  position: absolute;
  border-radius: 50%;
  filter: blur(30px);
  pointer-events: none;
}

.home__stat-value {
  font-size: 1.625rem;
  font-weight: 700;
  color: var(--primary);
  line-height: 1.2;
  margin: 0.25rem 0 0.125rem;
}

.home__stat-label {
  font-size: 0.75rem;
  color: var(--text-muted);
}

.home__status-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: #86EFAC;
  box-shadow: 0 0 8px #86EFAC;
  animation: status-pulse 2s ease-in-out infinite;
}
@keyframes status-pulse {
  0%,100% { box-shadow: 0 0 4px #86EFAC; }
  50%      { box-shadow: 0 0 12px #86EFAC; }
}

/* ── Steps Panel ── */
.home__steps {
  padding: 1.25rem;
}

.home__steps-header {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 1rem;
  letter-spacing: 0.08em;
}

.home__steps-list {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.home__step {
  display: flex;
  align-items: center;
  gap: 0.875rem;
  padding: 0.625rem 0.75rem;
  border-radius: var(--radius-md);
  transition: background var(--duration-fast);
  cursor: default;
  animation: slide-right 400ms var(--ease-out) both;
}
.home__step:hover {
  background: var(--surface-container-high);
}
.home__step:hover .home__step-arrow {
  opacity: 1;
  transform: translateX(0);
}

.home__step-num {
  font-size: 0.75rem;
  font-weight: 700;
  color: var(--text-muted);
  min-width: 24px;
}

.home__step-body { flex: 1; }

.home__step-title {
  font-size: 0.8125rem;
  font-weight: 600;
  color: var(--text-primary);
  line-height: 1.3;
}

.home__step-desc {
  font-size: 0.6875rem;
  color: var(--text-muted);
  margin-top: 1px;
}

.home__step-arrow {
  font-size: 14px;
  color: var(--primary);
  opacity: 0;
  transform: translateX(-4px);
  transition: opacity var(--duration-fast), transform var(--duration-fast);
}

/* ── Console Section ── */
.home__console {
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
  padding-top: 3rem;
  border-top: 1px solid var(--ghost-border);
}

.home__console-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.home__console-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.25rem;
  align-items: start;
}

/* Upload Zone */
.home__upload-zone {
  min-height: 220px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  border: 1px dashed rgba(171, 137, 127, 0.2);
  transition: border-color var(--duration-fast), background var(--duration-fast);
  padding: 1.5rem;
}
.home__upload-zone:hover,
.home__upload-zone--drag {
  border-color: var(--primary);
  background: rgba(255, 181, 158, 0.04);
}
.home__upload-zone--filled {
  align-items: flex-start;
  border-style: solid;
}

.home__upload-placeholder {
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.75rem;
}

.home__upload-icon {
  width: 56px;
  height: 56px;
  border-radius: var(--radius-lg);
  background: var(--surface-container-high);
  border: 1px solid var(--ghost-border);
  display: flex;
  align-items: center;
  justify-content: center;
}

.home__upload-title {
  font-weight: 600;
  font-size: 0.9375rem;
  color: var(--text-primary);
}

.home__file-list {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.home__file-item {
  display: flex;
  align-items: center;
  gap: 0.625rem;
  padding: 0.625rem 0.75rem;
  background: var(--surface-container-high);
  border-radius: var(--radius-md);
  border: 1px solid var(--ghost-border);
}

.home__file-name {
  flex: 1;
  font-size: 0.8125rem;
  color: var(--text-primary);
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.home__file-size {
  color: var(--text-muted);
  font-size: 0.6875rem;
  flex-shrink: 0;
}

.home__file-remove {
  background: none;
  border: none;
  cursor: pointer;
  color: var(--text-muted);
  padding: 0.125rem;
  border-radius: var(--radius-sm);
  display: flex;
  align-items: center;
  transition: color var(--duration-fast);
}
.home__file-remove:hover { color: var(--error); }

/* Input Side */
.home__input-side {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.home__divider-label {
  color: var(--text-muted);
  letter-spacing: 0.08em;
  padding-bottom: 0.5rem;
  border-bottom: 1px solid var(--ghost-border);
}

.home__textarea-wrap {
  position: relative;
}

.home__textarea {
  resize: vertical;
  min-height: 140px;
  font-family: var(--font-mono);
  font-size: 0.8125rem;
  line-height: 1.65;
}

.home__model-badge {
  position: absolute;
  bottom: 0.75rem;
  right: 1rem;
  color: var(--text-muted);
  font-size: 0.6875rem;
  pointer-events: none;
}

.home__start-btn {
  width: 100%;
  justify-content: space-between;
  padding: 1rem 1.25rem;
  font-size: 0.875rem;
  letter-spacing: 0.05em;
  box-shadow: var(--shadow-glow-primary);
  transition: all var(--duration-normal) var(--ease-out);
}
.home__start-btn:disabled {
  background: var(--surface-container-high);
  color: var(--text-muted);
  cursor: not-allowed;
  box-shadow: none;
  filter: none;
  transform: none;
}

@media (max-width: 1024px) {
  .home__hero { grid-template-columns: 1fr; }
  .home__console-grid { grid-template-columns: 1fr; }
}
</style>
