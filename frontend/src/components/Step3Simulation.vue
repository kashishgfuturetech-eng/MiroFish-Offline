<template>
  <div class="sim">
    <!-- Control Bar -->
    <div class="sim__control-bar glass">
      <!-- Platform Statuses -->
      <div class="sim__platforms">
        <!-- Info Plaza (Twitter-like) -->
        <div class="sim__platform" :class="{'sim__platform--active': runStatus.twitter_running, 'sim__platform--done': runStatus.twitter_completed}">
          <div class="sim__platform-header">
            <span class="material-symbols-outlined" style="font-size:16px">public</span>
            <span class="label-sm sim__platform-name">Info Plaza</span>
            <span v-if="runStatus.twitter_completed" class="chip chip-green" style="margin-left:auto">
              <span class="material-symbols-outlined" style="font-size:11px">check</span> Done
            </span>
            <div v-else-if="runStatus.twitter_running" class="ai-chip" style="margin-left:auto;padding:.2rem .5rem">
              <span class="ai-chip-dot"></span>
              <span class="label-sm" style="color:var(--secondary)">Live</span>
            </div>
          </div>
          <div class="sim__platform-stats">
            <div class="sim__pstat">
              <span class="label-sm sim__pstat-label">Round</span>
              <span class="sim__pstat-val font-mono">{{ runStatus.twitter_current_round || 0 }}<span class="sim__pstat-total">/{{ runStatus.total_rounds || maxRounds || '—' }}</span></span>
            </div>
            <div class="sim__pstat">
              <span class="label-sm sim__pstat-label">Elapsed</span>
              <span class="sim__pstat-val font-mono">{{ twitterElapsed }}</span>
            </div>
            <div class="sim__pstat">
              <span class="label-sm sim__pstat-label">Actions</span>
              <span class="sim__pstat-val font-mono">{{ runStatus.twitter_actions_count || 0 }}</span>
            </div>
          </div>
          <!-- Actions Tooltip -->
          <div class="sim__actions-row">
            <span class="sim__action-tag label-sm" v-for="a in ['POST','LIKE','REPOST','QUOTE','FOLLOW','IDLE']" :key="a">{{ a }}</span>
          </div>
        </div>

        <!-- Divider -->
        <div class="sim__platform-divider">
          <span class="material-symbols-outlined" style="color:var(--text-muted);font-size:18px">sync</span>
        </div>

        <!-- Topic Community (Reddit-like) -->
        <div class="sim__platform" :class="{'sim__platform--active': runStatus.reddit_running, 'sim__platform--done': runStatus.reddit_completed}">
          <div class="sim__platform-header">
            <span class="material-symbols-outlined" style="font-size:16px">forum</span>
            <span class="label-sm sim__platform-name">Topic Community</span>
            <span v-if="runStatus.reddit_completed" class="chip chip-green" style="margin-left:auto">
              <span class="material-symbols-outlined" style="font-size:11px">check</span> Done
            </span>
            <div v-else-if="runStatus.reddit_running" class="ai-chip" style="margin-left:auto;padding:.2rem .5rem">
              <span class="ai-chip-dot"></span>
              <span class="label-sm" style="color:var(--secondary)">Live</span>
            </div>
          </div>
          <div class="sim__platform-stats">
            <div class="sim__pstat">
              <span class="label-sm sim__pstat-label">Round</span>
              <span class="sim__pstat-val font-mono">{{ runStatus.reddit_current_round || 0 }}<span class="sim__pstat-total">/{{ runStatus.total_rounds || maxRounds || '—' }}</span></span>
            </div>
            <div class="sim__pstat">
              <span class="label-sm sim__pstat-label">Elapsed</span>
              <span class="sim__pstat-val font-mono">{{ redditElapsed }}</span>
            </div>
            <div class="sim__pstat">
              <span class="label-sm sim__pstat-label">Actions</span>
              <span class="sim__pstat-val font-mono">{{ runStatus.reddit_actions_count || 0 }}</span>
            </div>
          </div>
          <div class="sim__actions-row">
            <span class="sim__action-tag label-sm" v-for="a in ['POST','COMMENT','LIKE','SHARE','IDLE']" :key="a">{{ a }}</span>
          </div>
        </div>
      </div>

      <!-- Right: Controls -->
      <div class="sim__controls">
        <button
          v-if="!isRunning && !isCompleted"
          class="btn-primary"
          @click="startSimulation"
          :disabled="isStarting"
        >
          <span class="material-symbols-outlined" style="font-size:18px">{{ isStarting ? 'hourglass_empty' : 'play_arrow' }}</span>
          {{ isStarting ? 'Launching…' : 'Start Simulation' }}
        </button>
        <button v-else-if="isRunning" class="btn-secondary" @click="stopSimulation">
          <span class="material-symbols-outlined" style="font-size:18px">stop</span>
          Stop
        </button>
        <button v-if="isCompleted" class="btn-primary" @click="$emit('completed')">
          View Report
          <span class="material-symbols-outlined" style="font-size:18px">analytics</span>
        </button>
      </div>
    </div>

    <!-- Main Split Layout -->
    <div class="sim__split">
      <!-- LEFT: Action Log -->
      <div class="sim__log card">
        <div class="sim__log-header">
          <div class="sim__log-title">
            <span class="material-symbols-outlined" style="font-size:16px;color:var(--secondary)">terminal</span>
            <span class="label-sm" style="color:var(--text-muted)">Action Log</span>
          </div>
          <div style="display:flex;gap:.5rem;align-items:center">
            <span class="chip chip-muted label-sm">{{ actionLog.length }} events</span>
            <button class="btn-ghost" style="font-size:.6875rem" @click="actionLog = []">Clear</button>
          </div>
        </div>

        <!-- Filter tabs -->
        <div class="sim__log-filters">
          <button
            v-for="f in logFilters"
            :key="f"
            class="sim__filter-tab"
            :class="{'sim__filter-tab--active': activeFilter === f}"
            @click="activeFilter = f"
          >
            {{ f }}
          </button>
        </div>

        <!-- Log entries -->
        <div class="sim__log-body" ref="logBodyRef">
          <div
            v-for="(entry, i) in filteredLog"
            :key="i"
            class="sim__log-entry"
            :class="`sim__log-entry--${entry.platform}`"
          >
            <div class="sim__log-entry-header">
              <span class="sim__log-entry-time font-mono">{{ entry.time }}</span>
              <span class="chip label-sm" :class="entry.platform === 'twitter' ? 'chip-blue' : 'chip-orange'" style="font-size:.55rem">
                {{ entry.platform === 'twitter' ? 'Info Plaza' : 'Topic' }}
              </span>
              <span class="sim__log-entry-action chip chip-muted label-sm">{{ entry.action }}</span>
            </div>
            <div class="sim__log-entry-content">
              <span class="sim__log-entry-agent font-mono">{{ entry.agent }}</span>
              <span class="sim__log-entry-text">{{ entry.text }}</span>
            </div>
          </div>

          <div v-if="!filteredLog.length" class="sim__log-empty">
            <span class="material-symbols-outlined" style="font-size:32px;color:var(--text-muted)">inbox</span>
            <span class="label-sm" style="color:var(--text-muted)">No events yet. Start the simulation.</span>
          </div>
        </div>
      </div>

      <!-- RIGHT: Live Stats + Activity Graph -->
      <div class="sim__right">
        <!-- Live Metrics -->
        <div class="sim__metrics card">
          <span class="label-sm" style="color:var(--text-muted);margin-bottom:.75rem;display:block">Live Metrics</span>
          <div class="sim__metrics-grid">
            <div class="sim__metric card-nested" v-for="m in liveMetrics" :key="m.label">
              <div class="sim__metric-val font-headline">{{ m.value }}</div>
              <div class="label-sm" style="color:var(--text-muted)">{{ m.label }}</div>
              <div v-if="m.trend" class="sim__metric-trend" :class="m.trend > 0 ? 'sim__metric-trend--up' : 'sim__metric-trend--down'">
                <span class="material-symbols-outlined" style="font-size:12px">{{ m.trend > 0 ? 'trending_up' : 'trending_down' }}</span>
                {{ Math.abs(m.trend) }}%
              </div>
            </div>
          </div>
        </div>

        <!-- Activity Heatmap / Mini Graph -->
        <div class="sim__graph card">
          <div class="sim__graph-header">
            <span class="label-sm" style="color:var(--text-muted)">Activity Distribution</span>
            <div style="display:flex;gap:.375rem">
              <span class="sim__legend-dot" style="background:var(--primary-container)"></span>
              <span class="label-sm" style="color:var(--text-muted)">Info Plaza</span>
              <span class="sim__legend-dot" style="background:var(--secondary-container);margin-left:.5rem"></span>
              <span class="label-sm" style="color:var(--text-muted)">Topic</span>
            </div>
          </div>
          <div class="sim__graph-bars">
            <div
              v-for="(bar, i) in activityBars"
              :key="i"
              class="sim__graph-bar-group"
              :title="`Round ${bar.round}`"
            >
              <div class="sim__graph-bar sim__graph-bar--twitter" :style="`height:${bar.twitter}%`"></div>
              <div class="sim__graph-bar sim__graph-bar--reddit"  :style="`height:${bar.reddit}%`"></div>
            </div>
          </div>
          <div class="sim__graph-axis">
            <span class="label-sm" v-for="l in graphAxisLabels" :key="l" style="color:var(--text-muted)">{{ l }}</span>
          </div>
        </div>

        <!-- Agent Status Grid -->
        <div class="sim__agents card">
          <span class="label-sm" style="color:var(--text-muted);margin-bottom:.75rem;display:block">Agent Status</span>
          <div class="sim__agents-grid">
            <div
              v-for="agent in agentGrid"
              :key="agent.id"
              class="sim__agent-dot"
              :class="`sim__agent-dot--${agent.status}`"
              :title="`${agent.name} · ${agent.status}`"
            ></div>
          </div>
          <div class="sim__agents-legend">
            <div class="sim__agents-legend-item" v-for="l in agentLegend" :key="l.label">
              <span class="sim__legend-dot" :style="`background:${l.color}`"></span>
              <span class="label-sm" style="color:var(--text-muted)">{{ l.label }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onUnmounted, nextTick } from 'vue'

const props = defineProps({ simulationId: String })
const emit = defineEmits(['completed'])

const maxRounds = ref(100)
const isStarting = ref(false)
const isRunning = ref(false)
const isCompleted = ref(false)
const activeFilter = ref('All')
const logBodyRef = ref(null)

const logFilters = ['All', 'Info Plaza', 'Topic', 'POST', 'LIKE', 'COMMENT']

let simInterval = null
let twitterStart = null
let redditStart = null
let twitterTimer = null
let redditTimer = null

const twitterElapsed = ref('00:00')
const redditElapsed  = ref('00:00')

const runStatus = ref({
  twitter_running: false, twitter_completed: false, twitter_current_round: 0, twitter_actions_count: 0,
  reddit_running:  false, reddit_completed:  false, reddit_current_round:  0, reddit_actions_count:  0,
  total_rounds: 100,
})

const actionLog = ref([])
const activityBars = ref(Array.from({ length: 24 }, (_, i) => ({ round: i * 4, twitter: 0, reddit: 0 })))
const graphAxisLabels = ['0', '25', '50', '75', '100']

const agentGrid = ref(Array.from({ length: 50 }, (_, i) => ({ id: i, name: `Agent_${String(i+1).padStart(3,'0')}`, status: 'idle' })))

const agentLegend = [
  { label: 'Active', color: 'var(--primary-container)' },
  { label: 'Thinking', color: 'var(--secondary)' },
  { label: 'Idle', color: 'var(--surface-container-highest)' },
  { label: 'Done', color: '#86EFAC' },
]

const liveMetrics = computed(() => [
  { label: 'Total Actions', value: (runStatus.value.twitter_actions_count + runStatus.value.reddit_actions_count).toLocaleString(), trend: isRunning.value ? 12 : null },
  { label: 'Active Agents', value: agentGrid.value.filter(a => a.status === 'active').length, trend: null },
  { label: 'Posts Created', value: Math.floor(runStatus.value.twitter_actions_count * 0.35), trend: isRunning.value ? 8 : null },
  { label: 'Interactions', value: Math.floor(runStatus.value.twitter_actions_count * 0.65), trend: isRunning.value ? 15 : null },
])

const filteredLog = computed(() => {
  if (activeFilter.value === 'All') return actionLog.value.slice().reverse()
  if (activeFilter.value === 'Info Plaza') return actionLog.value.filter(e => e.platform === 'twitter').slice().reverse()
  if (activeFilter.value === 'Topic') return actionLog.value.filter(e => e.platform === 'reddit').slice().reverse()
  return actionLog.value.filter(e => e.action === activeFilter.value).slice().reverse()
})

function formatElapsed(start) {
  const s = Math.floor((Date.now() - start) / 1000)
  return `${String(Math.floor(s / 60)).padStart(2, '0')}:${String(s % 60).padStart(2, '0')}`
}

function addLogEntry(platform) {
  const actions = platform === 'twitter' ? ['POST', 'LIKE', 'REPOST', 'QUOTE', 'FOLLOW', 'IDLE'] : ['POST', 'COMMENT', 'LIKE', 'SHARE', 'IDLE']
  const action = actions[Math.floor(Math.random() * actions.length)]
  const agentId = Math.floor(Math.random() * 50)
  const texts = ['Shared market analysis thread', 'Liked a trending post', 'Commented on industry news', 'Followed key influencer', 'Reposted viral content', 'Published new insight', 'Responded to community query', 'Voted on discussion poll']
  const now = new Date()
  actionLog.value.push({
    time: `${String(now.getHours()).padStart(2,'0')}:${String(now.getMinutes()).padStart(2,'0')}:${String(now.getSeconds()).padStart(2,'0')}`,
    platform,
    action,
    agent: `Agent_${String(agentId + 1).padStart(3, '0')}`,
    text: texts[Math.floor(Math.random() * texts.length)],
  })
  if (actionLog.value.length > 200) actionLog.value.shift()
  // Update agent status
  agentGrid.value[agentId].status = action === 'IDLE' ? 'idle' : 'active'
  setTimeout(() => { if (agentGrid.value[agentId]) agentGrid.value[agentId].status = Math.random() > 0.3 ? 'idle' : 'thinking' }, 800)
}

async function startSimulation() {
  isStarting.value = true
  await new Promise(r => setTimeout(r, 1200))
  isStarting.value = false
  isRunning.value = true

  twitterStart = Date.now()
  redditStart  = Date.now()

  runStatus.value.twitter_running = true
  runStatus.value.reddit_running  = true

  twitterTimer = setInterval(() => { twitterElapsed.value = formatElapsed(twitterStart) }, 1000)
  redditTimer  = setInterval(() => { redditElapsed.value  = formatElapsed(redditStart) }, 1000)

  simInterval = setInterval(() => {
    // Advance rounds
    if (runStatus.value.twitter_current_round < maxRounds.value) {
      runStatus.value.twitter_current_round++
      runStatus.value.twitter_actions_count += Math.floor(Math.random() * 8) + 2
      addLogEntry('twitter')
    } else {
      runStatus.value.twitter_running = false
      runStatus.value.twitter_completed = true
      clearInterval(twitterTimer)
    }

    if (runStatus.value.reddit_current_round < maxRounds.value) {
      runStatus.value.reddit_current_round++
      runStatus.value.reddit_actions_count += Math.floor(Math.random() * 6) + 1
      addLogEntry('reddit')
    } else {
      runStatus.value.reddit_running = false
      runStatus.value.reddit_completed = true
      clearInterval(redditTimer)
    }

    // Update activity bars
    const barIdx = Math.floor(runStatus.value.twitter_current_round / (maxRounds.value / 24))
    if (barIdx < 24) {
      activityBars.value[barIdx].twitter = Math.min(100, activityBars.value[barIdx].twitter + Math.random() * 15)
      activityBars.value[barIdx].reddit  = Math.min(100, activityBars.value[barIdx].reddit  + Math.random() * 12)
    }

    if (runStatus.value.twitter_completed && runStatus.value.reddit_completed) {
      clearInterval(simInterval)
      isRunning.value = false
      isCompleted.value = true
      agentGrid.value.forEach(a => a.status = 'done')
    }
  }, 180)
}

function stopSimulation() {
  clearInterval(simInterval)
  clearInterval(twitterTimer)
  clearInterval(redditTimer)
  isRunning.value = false
  runStatus.value.twitter_running = false
  runStatus.value.reddit_running = false
}

onUnmounted(() => {
  clearInterval(simInterval)
  clearInterval(twitterTimer)
  clearInterval(redditTimer)
})
</script>

<style scoped>
.sim { display: flex; flex-direction: column; gap: 1.25rem; }

/* Control Bar */
.sim__control-bar {
  display: flex;
  align-items: flex-start;
  gap: 1.5rem;
  padding: 1.25rem 1.5rem;
  border-radius: var(--radius-xl);
  flex-wrap: wrap;
}

.sim__platforms {
  display: flex;
  align-items: stretch;
  gap: 1rem;
  flex: 1;
  min-width: 0;
}

.sim__platform {
  flex: 1;
  background: var(--surface-container);
  border-radius: var(--radius-md);
  padding: 1rem;
  border: 1px solid var(--ghost-border);
  transition: border-color var(--duration-fast), box-shadow var(--duration-fast);
}
.sim__platform--active {
  border-color: rgba(182, 196, 255, 0.3);
  box-shadow: var(--shadow-glow-ai);
}
.sim__platform--done {
  border-color: rgba(134, 239, 172, 0.2);
}

.sim__platform-header {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 0.75rem;
  color: var(--text-primary);
}

.sim__platform-name { color: var(--text-secondary); }

.sim__platform-stats {
  display: flex;
  gap: 1.25rem;
  margin-bottom: 0.75rem;
}

.sim__pstat { display: flex; flex-direction: column; gap: 0.125rem; }

.sim__pstat-label { color: var(--text-muted); font-size: 0.6rem; }

.sim__pstat-val {
  font-size: 1.125rem;
  font-weight: 700;
  color: var(--text-primary);
}
.sim__pstat-total {
  font-size: 0.75rem;
  color: var(--text-muted);
  font-weight: 400;
}

.sim__actions-row {
  display: flex;
  flex-wrap: wrap;
  gap: 0.25rem;
}

.sim__action-tag {
  padding: 0.125rem 0.375rem;
  background: var(--surface-container-high);
  border-radius: var(--radius-sm);
  color: var(--text-muted);
  font-size: 0.55rem;
  letter-spacing: 0.06em;
}

.sim__platform-divider {
  display: flex;
  align-items: center;
  padding: 0 0.25rem;
}

.sim__controls {
  display: flex;
  flex-direction: column;
  justify-content: center;
  gap: 0.5rem;
  flex-shrink: 0;
}

/* Split */
.sim__split {
  display: grid;
  grid-template-columns: 1fr 340px;
  gap: 1.25rem;
  align-items: start;
}

/* Log */
.sim__log {
  display: flex;
  flex-direction: column;
  max-height: 600px;
  overflow: hidden;
}

.sim__log-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 1rem 1.25rem 0.75rem;
}

.sim__log-title {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.sim__log-filters {
  display: flex;
  gap: 0.25rem;
  padding: 0 1.25rem 0.75rem;
  border-bottom: 1px solid var(--ghost-border);
  flex-wrap: wrap;
}

.sim__filter-tab {
  padding: 0.25rem 0.625rem;
  background: transparent;
  border: 1px solid var(--ghost-border);
  border-radius: var(--radius-full);
  color: var(--text-muted);
  font-size: 0.6875rem;
  font-weight: 600;
  cursor: pointer;
  transition: all var(--duration-fast);
  font-family: var(--font-body);
  letter-spacing: 0.04em;
}
.sim__filter-tab:hover { border-color: rgba(171,137,127,.4); color: var(--text-secondary); }
.sim__filter-tab--active {
  background: rgba(255,90,31,.12);
  border-color: rgba(255,90,31,.3);
  color: var(--primary);
}

.sim__log-body {
  flex: 1;
  overflow-y: auto;
  padding: 0.75rem 1.25rem;
  display: flex;
  flex-direction: column;
  gap: 0.375rem;
  max-height: 460px;
}

.sim__log-entry {
  padding: 0.625rem 0.75rem;
  border-radius: var(--radius-md);
  background: var(--surface-low);
  border-left: 2px solid var(--ghost-border);
  animation: fade-in 200ms ease both;
}
.sim__log-entry--twitter { border-left-color: rgba(182,196,255,.4); }
.sim__log-entry--reddit  { border-left-color: rgba(255,181,158,.4); }

.sim__log-entry-header {
  display: flex;
  align-items: center;
  gap: 0.375rem;
  margin-bottom: 0.25rem;
}

.sim__log-entry-time {
  font-size: 0.625rem;
  color: var(--text-muted);
}

.sim__log-entry-content {
  display: flex;
  align-items: baseline;
  gap: 0.5rem;
}

.sim__log-entry-agent {
  font-size: 0.6875rem;
  color: var(--secondary);
  flex-shrink: 0;
}

.sim__log-entry-text {
  font-size: 0.75rem;
  color: var(--text-secondary);
}

.sim__log-empty {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
  padding: 3rem 1rem;
}

/* Right Panel */
.sim__right {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.sim__metrics { padding: 1.25rem; }

.sim__metrics-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 0.625rem;
}

.sim__metric { padding: .75rem; }

.sim__metric-val {
  font-size: 1.375rem;
  font-weight: 700;
  color: var(--primary);
  margin-bottom: 0.125rem;
}

.sim__metric-trend {
  display: flex;
  align-items: center;
  gap: 0.125rem;
  font-size: 0.625rem;
  font-weight: 700;
  margin-top: 0.25rem;
}
.sim__metric-trend--up   { color: #86EFAC; }
.sim__metric-trend--down { color: var(--error); }

/* Graph */
.sim__graph { padding: 1.25rem; }

.sim__graph-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 1rem;
}

.sim__graph-bars {
  display: flex;
  align-items: flex-end;
  gap: 2px;
  height: 80px;
}

.sim__graph-bar-group {
  flex: 1;
  display: flex;
  align-items: flex-end;
  gap: 1px;
  height: 100%;
}

.sim__graph-bar {
  flex: 1;
  border-radius: 2px 2px 0 0;
  min-height: 2px;
  transition: height 300ms var(--ease-out);
}
.sim__graph-bar--twitter { background: var(--primary-container); opacity: 0.85; }
.sim__graph-bar--reddit  { background: var(--secondary-container); opacity: 0.85; }

.sim__graph-axis {
  display: flex;
  justify-content: space-between;
  margin-top: 0.375rem;
}

.sim__legend-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  display: inline-block;
}

/* Agents */
.sim__agents { padding: 1.25rem; }

.sim__agents-grid {
  display: grid;
  grid-template-columns: repeat(10, 1fr);
  gap: 4px;
  margin-bottom: 0.75rem;
}

.sim__agent-dot {
  width: 100%;
  aspect-ratio: 1;
  border-radius: 3px;
  cursor: pointer;
  transition: transform var(--duration-fast);
}
.sim__agent-dot:hover { transform: scale(1.3); }
.sim__agent-dot--idle     { background: var(--surface-container-highest); }
.sim__agent-dot--active   { background: var(--primary-container); box-shadow: 0 0 4px rgba(255,90,31,.4); }
.sim__agent-dot--thinking { background: var(--secondary); box-shadow: 0 0 4px rgba(182,196,255,.4); }
.sim__agent-dot--done     { background: #86EFAC; }

.sim__agents-legend {
  display: flex;
  gap: 0.875rem;
  flex-wrap: wrap;
}

.sim__agents-legend-item {
  display: flex;
  align-items: center;
  gap: 0.375rem;
}

@media (max-width: 1100px) {
  .sim__split { grid-template-columns: 1fr; }
}
</style>
