<template>
  <div class="graph-build">
    <!-- Metrics Row -->
    <div class="graph-build__metrics">
      <div class="graph-build__metric card" v-for="m in metrics" :key="m.label">
        <div class="orb orb-blue" style="width:80px;height:80px;top:-20px;right:-20px;opacity:0.3;"></div>
        <span class="label-sm" style="color:var(--text-muted)">{{ m.label }}</span>
        <div class="graph-build__metric-val font-headline">{{ m.value }}</div>
        <div class="graph-build__metric-sub">{{ m.sub }}</div>
      </div>
    </div>

    <!-- ── Phase 01: Ontology Generation ── -->
    <div class="graph-build__phase card" :class="phaseClass(0)">
      <div class="graph-build__phase-header">
        <div class="graph-build__phase-meta">
          <span class="chip chip-blue label-sm">POST /api/graph/ontology/generate</span>
        </div>
        <div class="graph-build__phase-title-row">
          <span class="graph-build__phase-num font-mono">01</span>
          <h2 class="headline-md" style="color:var(--text-primary)">Ontology Generation</h2>
        </div>
        <p class="graph-build__phase-desc">
          LLM analyzes document content and simulation requirements, extracts reality seeds,
          and automatically generates appropriate ontology structures.
        </p>
        <!-- Status -->
        <div class="graph-build__phase-status">
          <span v-if="currentPhase > 0" class="chip chip-green">
            <span class="material-symbols-outlined" style="font-size:12px">check</span> Completed
          </span>
          <div v-else-if="currentPhase === 0 && isProcessing" class="ai-chip">
            <span class="ai-chip-dot"></span>
            <span class="label-sm" style="color:var(--secondary)">Generating ontology…</span>
          </div>
          <span v-else class="chip chip-muted">Waiting</span>
        </div>
      </div>

      <!-- Detail overlay -->
      <transition name="overlay">
        <div v-if="selectedItem" class="graph-build__overlay glass-elevated">
          <div class="graph-build__overlay-header">
            <div style="display:flex;align-items:center;gap:.75rem">
              <span class="chip" :class="selectedItem.itemType === 'entity' ? 'chip-orange' : 'chip-blue'">
                {{ selectedItem.itemType === 'entity' ? 'ENTITY' : 'RELATION' }}
              </span>
              <span class="headline-md" style="color:var(--text-primary)">{{ selectedItem.name }}</span>
            </div>
            <button class="btn-ghost" @click="selectedItem = null">
              <span class="material-symbols-outlined">close</span>
            </button>
          </div>
          <p class="graph-build__overlay-desc">{{ selectedItem.description }}</p>

          <div v-if="selectedItem.attributes?.length" class="graph-build__overlay-section">
            <span class="label-sm" style="color:var(--text-muted)">Attributes</span>
            <div class="graph-build__attr-list">
              <div v-for="attr in selectedItem.attributes" :key="attr.name" class="graph-build__attr-item">
                <span class="graph-build__attr-name font-mono">{{ attr.name }}</span>
                <span class="chip chip-muted" style="font-size:.6rem">{{ attr.type }}</span>
                <span class="graph-build__attr-desc">{{ attr.description }}</span>
              </div>
            </div>
          </div>

          <div v-if="selectedItem.examples?.length" class="graph-build__overlay-section">
            <span class="label-sm" style="color:var(--text-muted)">Examples</span>
            <div style="display:flex;flex-wrap:wrap;gap:.375rem">
              <span v-for="ex in selectedItem.examples" :key="ex" class="chip chip-orange">{{ ex }}</span>
            </div>
          </div>

          <div v-if="selectedItem.source_targets?.length" class="graph-build__overlay-section">
            <span class="label-sm" style="color:var(--text-muted)">Connections</span>
            <div class="graph-build__conn-list">
              <div v-for="(c, i) in selectedItem.source_targets" :key="i" class="graph-build__conn-item">
                <span class="chip chip-muted">{{ c.source }}</span>
                <span class="material-symbols-outlined" style="font-size:16px;color:var(--primary)">arrow_forward</span>
                <span class="chip chip-muted">{{ c.target }}</span>
              </div>
            </div>
          </div>
        </div>
      </transition>

      <!-- Entity Types -->
      <div v-if="projectData?.ontology?.entity_types" class="graph-build__tags-section" :class="{'graph-build__tags-section--dimmed': selectedItem}">
        <span class="label-sm" style="color:var(--text-muted)">Generated Entity Types</span>
        <div class="graph-build__tag-list">
          <button
            v-for="entity in projectData.ontology.entity_types"
            :key="entity.name"
            class="graph-build__tag"
            :class="{'graph-build__tag--selected': selectedItem?.name === entity.name}"
            @click="selectItem(entity, 'entity')"
          >
            <span class="material-symbols-outlined" style="font-size:14px">category</span>
            {{ entity.name }}
          </button>
        </div>
      </div>

      <!-- Relation Types -->
      <div v-if="projectData?.ontology?.relation_types" class="graph-build__tags-section" :class="{'graph-build__tags-section--dimmed': selectedItem}">
        <span class="label-sm" style="color:var(--text-muted)">Generated Relation Types</span>
        <div class="graph-build__tag-list">
          <button
            v-for="rel in projectData.ontology.relation_types"
            :key="rel.name"
            class="graph-build__tag graph-build__tag--relation"
            :class="{'graph-build__tag--selected': selectedItem?.name === rel.name}"
            @click="selectItem(rel, 'relation')"
          >
            <span class="material-symbols-outlined" style="font-size:14px">share</span>
            {{ rel.name }}
          </button>
        </div>
      </div>

      <!-- Demo State / CTA -->
      <div v-if="currentPhase === 0 && !isProcessing" class="graph-build__cta">
        <button class="btn-primary" @click="startOntology">
          <span class="material-symbols-outlined" style="font-size:18px">auto_awesome</span>
          Generate Ontology
        </button>
      </div>
    </div>

    <!-- ── Phase 02: Entity Extraction ── -->
    <div class="graph-build__phase card" :class="phaseClass(1)">
      <div class="graph-build__phase-header">
        <div class="graph-build__phase-meta">
          <span class="chip chip-blue label-sm">POST /api/graph/entities/extract</span>
        </div>
        <div class="graph-build__phase-title-row">
          <span class="graph-build__phase-num font-mono">02</span>
          <h2 class="headline-md" style="color:var(--text-primary)">Entity Extraction</h2>
        </div>
        <p class="graph-build__phase-desc">
          Using the generated ontology as schema, extract all named entities and their
          attributes from the source documents.
        </p>
        <div class="graph-build__phase-status">
          <span v-if="currentPhase > 1" class="chip chip-green">
            <span class="material-symbols-outlined" style="font-size:12px">check</span> Completed
          </span>
          <div v-else-if="currentPhase === 1 && isProcessing" class="ai-chip">
            <span class="ai-chip-dot"></span>
            <span class="label-sm" style="color:var(--secondary)">Extracting entities…</span>
          </div>
          <span v-else class="chip chip-muted">{{ currentPhase < 1 ? 'Waiting' : 'Ready' }}</span>
        </div>
      </div>

      <div v-if="currentPhase >= 1" class="graph-build__stats-grid">
        <div class="card-nested" v-for="s in entityStats" :key="s.label">
          <div class="graph-build__stat-val font-headline">{{ s.value }}</div>
          <div class="label-sm" style="color:var(--text-muted)">{{ s.label }}</div>
        </div>
      </div>

      <div v-if="currentPhase === 1 && !isProcessing" class="graph-build__cta">
        <button class="btn-primary" @click="startExtraction">
          <span class="material-symbols-outlined" style="font-size:18px">search</span>
          Extract Entities
        </button>
      </div>
    </div>

    <!-- ── Phase 03: Graph Storage ── -->
    <div class="graph-build__phase card" :class="phaseClass(2)">
      <div class="graph-build__phase-header">
        <div class="graph-build__phase-meta">
          <span class="chip chip-blue label-sm">POST /api/graph/store</span>
        </div>
        <div class="graph-build__phase-title-row">
          <span class="graph-build__phase-num font-mono">03</span>
          <h2 class="headline-md" style="color:var(--text-primary)">Graph Storage</h2>
        </div>
        <p class="graph-build__phase-desc">
          Store the knowledge graph in Neo4j. Build vector embeddings for semantic
          retrieval during agent reasoning.
        </p>
        <div class="graph-build__phase-status">
          <span v-if="currentPhase > 2" class="chip chip-green">
            <span class="material-symbols-outlined" style="font-size:12px">check</span> Completed
          </span>
          <div v-else-if="currentPhase === 2 && isProcessing" class="ai-chip">
            <span class="ai-chip-dot"></span>
            <span class="label-sm" style="color:var(--secondary)">Writing to Neo4j…</span>
          </div>
          <span v-else class="chip chip-muted">{{ currentPhase < 2 ? 'Waiting' : 'Ready' }}</span>
        </div>
      </div>

      <div v-if="currentPhase >= 2 && graphStorageInfo" class="graph-build__info-card card-nested">
        <div class="graph-build__info-row" v-for="row in graphStorageInfo" :key="row.label">
          <span class="graph-build__info-label label-sm">{{ row.label }}</span>
          <span class="graph-build__info-val font-mono">{{ row.value }}</span>
        </div>
      </div>

      <div v-if="currentPhase === 2 && !isProcessing" class="graph-build__cta">
        <button class="btn-primary" @click="startStorage">
          <span class="material-symbols-outlined" style="font-size:18px">storage</span>
          Store Graph
        </button>
      </div>

      <!-- Final CTA -->
      <div v-if="currentPhase > 2" class="graph-build__complete">
        <div class="graph-build__complete-badge chip chip-green">
          <span class="material-symbols-outlined" style="font-size:14px">check_circle</span>
          Graph Build Complete
        </div>
        <button class="btn-primary" @click="proceedToEnv">
          Proceed to Env Setup
          <span class="material-symbols-outlined" style="font-size:18px">arrow_forward</span>
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps({ projectData: Object })
const emit = defineEmits(['completed'])

const currentPhase = ref(0)
const isProcessing = ref(false)
const selectedItem = ref(null)

const metrics = ref([
  { label: 'Documents', value: '3', sub: 'Reality seeds loaded' },
  { label: 'Tokens Processed', value: '—', sub: 'Context window usage' },
  { label: 'Graph ID', value: 'Pending', sub: 'Neo4j node ID' },
])

const entityStats = ref([
  { label: 'Entities Found', value: '—' },
  { label: 'Relations Found', value: '—' },
  { label: 'Properties', value: '—' },
])

const graphStorageInfo = ref(null)

function phaseClass(phase) {
  if (currentPhase.value > phase) return 'graph-build__phase--completed'
  if (currentPhase.value === phase) return 'graph-build__phase--active'
  return 'graph-build__phase--locked'
}

function selectItem(item, type) {
  selectedItem.value = selectedItem.value?.name === item.name ? null : { ...item, itemType: type }
}

async function simulate(delay) {
  isProcessing.value = true
  await new Promise(r => setTimeout(r, delay))
  isProcessing.value = false
}

async function startOntology() {
  await simulate(2000)
  currentPhase.value = 1
  metrics.value[1].value = '12.4K'
}

async function startExtraction() {
  await simulate(1800)
  currentPhase.value = 2
  entityStats.value = [
    { label: 'Entities Found', value: '247' },
    { label: 'Relations Found', value: '89' },
    { label: 'Properties', value: '1,203' },
  ]
}

async function startStorage() {
  await simulate(1500)
  currentPhase.value = 3
  metrics.value[2].value = 'GRF-8A2F1'
  graphStorageInfo.value = [
    { label: 'Graph ID', value: 'GRF-8A2F1' },
    { label: 'Node Count', value: '247' },
    { label: 'Edge Count', value: '89' },
    { label: 'Vector Dims', value: '1536' },
    { label: 'Storage', value: 'Neo4j (local)' },
  ]
}

function proceedToEnv() {
  emit('completed', { graph_id: 'GRF-8A2F1', ontology: { entity_types: [], relation_types: [] } })
}
</script>

<style scoped>
.graph-build {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.graph-build__metrics {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
}

.graph-build__metric {
  padding: 1.25rem 1.5rem;
  overflow: hidden;
}

.graph-build__metric-val {
  font-size: 1.75rem;
  font-weight: 700;
  color: var(--primary);
  margin: 0.25rem 0 0.125rem;
}

.graph-build__metric-sub {
  font-size: 0.6875rem;
  color: var(--text-muted);
}

/* Phase Card */
.graph-build__phase {
  padding: 1.75rem;
  transition: opacity var(--duration-normal);
}

.graph-build__phase--locked {
  opacity: 0.45;
  pointer-events: none;
}

.graph-build__phase--active {
  border-color: rgba(255, 181, 158, 0.25);
  box-shadow: var(--shadow-glow-primary);
}

.graph-build__phase--completed {
  border-color: rgba(134, 239, 172, 0.15);
}

.graph-build__phase-header {
  display: flex;
  flex-direction: column;
  gap: 0.625rem;
  margin-bottom: 1.25rem;
}

.graph-build__phase-meta {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.graph-build__phase-title-row {
  display: flex;
  align-items: center;
  gap: 0.875rem;
}

.graph-build__phase-num {
  font-size: 2rem;
  font-weight: 700;
  color: var(--text-muted);
  opacity: 0.4;
  line-height: 1;
}

.graph-build__phase-desc {
  color: var(--text-muted);
  font-size: 0.875rem;
  line-height: 1.7;
  max-width: 640px;
}

.graph-build__phase-status {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

/* Overlay */
.graph-build__overlay {
  border-radius: var(--radius-xl);
  padding: 1.5rem;
  margin-bottom: 1.25rem;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.graph-build__overlay-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.graph-build__overlay-desc {
  color: var(--text-secondary);
  font-size: 0.875rem;
  line-height: 1.7;
}

.graph-build__overlay-section {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.graph-build__attr-list {
  display: flex;
  flex-direction: column;
  gap: 0.375rem;
}

.graph-build__attr-item {
  display: flex;
  align-items: center;
  gap: 0.625rem;
  padding: 0.5rem 0.75rem;
  background: var(--surface-container-high);
  border-radius: var(--radius-md);
}

.graph-build__attr-name {
  font-size: 0.8125rem;
  color: var(--primary);
  min-width: 100px;
}

.graph-build__attr-desc {
  font-size: 0.75rem;
  color: var(--text-muted);
  flex: 1;
}

.graph-build__conn-list {
  display: flex;
  flex-direction: column;
  gap: 0.375rem;
}

.graph-build__conn-item {
  display: flex;
  align-items: center;
  gap: 0.625rem;
}

/* Tags */
.graph-build__tags-section {
  display: flex;
  flex-direction: column;
  gap: 0.625rem;
  margin-bottom: 0.75rem;
  transition: opacity var(--duration-normal);
}

.graph-build__tags-section--dimmed { opacity: 0.35; }

.graph-build__tag-list {
  display: flex;
  flex-wrap: wrap;
  gap: 0.375rem;
}

.graph-build__tag {
  display: inline-flex;
  align-items: center;
  gap: 0.375rem;
  padding: 0.375rem 0.75rem;
  background: rgba(255, 90, 31, 0.1);
  color: var(--primary);
  border: 1px solid rgba(255, 90, 31, 0.2);
  border-radius: var(--radius-full);
  font-size: 0.75rem;
  font-weight: 600;
  cursor: pointer;
  transition: all var(--duration-fast);
  font-family: var(--font-body);
}
.graph-build__tag:hover {
  background: rgba(255, 90, 31, 0.2);
  border-color: rgba(255, 90, 31, 0.4);
}
.graph-build__tag--selected {
  background: rgba(255, 90, 31, 0.25);
  border-color: var(--primary-container);
  box-shadow: 0 0 10px rgba(255, 90, 31, 0.2);
}

.graph-build__tag--relation {
  background: rgba(14, 63, 174, 0.15);
  color: var(--secondary);
  border-color: rgba(182, 196, 255, 0.2);
}
.graph-build__tag--relation:hover {
  background: rgba(14, 63, 174, 0.25);
  border-color: rgba(182, 196, 255, 0.4);
}
.graph-build__tag--relation.graph-build__tag--selected {
  background: rgba(14, 63, 174, 0.3);
  border-color: var(--secondary);
  box-shadow: var(--shadow-glow-ai);
}

/* Stats Grid */
.graph-build__stats-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 0.75rem;
  margin-bottom: 1.25rem;
}

.graph-build__stat-val {
  font-size: 1.5rem;
  font-weight: 700;
  color: var(--primary);
  margin-bottom: 0.25rem;
}

/* Info Card */
.graph-build__info-card {
  margin-bottom: 1.25rem;
}

.graph-build__info-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.5rem 0;
  border-bottom: 1px solid var(--ghost-border);
}
.graph-build__info-row:last-child { border-bottom: none; }

.graph-build__info-label { color: var(--text-muted); }

.graph-build__info-val {
  color: var(--text-primary);
  font-size: 0.8125rem;
}

/* CTA */
.graph-build__cta { display: flex; gap: 0.75rem; padding-top: 0.5rem; }

.graph-build__complete {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 1rem 1.25rem;
  background: rgba(134, 239, 172, 0.06);
  border: 1px solid rgba(134, 239, 172, 0.15);
  border-radius: var(--radius-md);
  margin-top: 0.75rem;
}

/* Overlay transition */
.overlay-enter-active, .overlay-leave-active { transition: all 250ms var(--ease-out); }
.overlay-enter-from { opacity: 0; transform: translateY(-8px); }
.overlay-leave-to   { opacity: 0; transform: translateY(4px); }
</style>
