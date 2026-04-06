<template>
  <aside class="sidenav">
    <!-- Engine Info -->
    <div class="sidenav__engine">
      <div class="sidenav__engine-icon">
        <span class="material-symbols-outlined icon-filled" style="font-size:18px;color:var(--secondary)">account_tree</span>
      </div>
      <div>
        <div class="sidenav__engine-name font-headline">Simulation Engine</div>
        <div class="sidenav__engine-ver label-sm">v4.2 Obsidian</div>
      </div>
    </div>

    <!-- Navigation Items -->
    <nav class="sidenav__nav">
      <a
        v-for="item in navItems"
        :key="item.step"
        class="sidenav__item"
        :class="{
          'sidenav__item--active': currentStep === item.step,
          'sidenav__item--completed': completedSteps.includes(item.step),
          'sidenav__item--locked': !completedSteps.includes(item.step) && currentStep !== item.step
        }"
        href="#"
        @click.prevent="$emit('navigate', item.step)"
      >
        <span class="sidenav__item-icon material-symbols-outlined">{{ item.icon }}</span>
        <div class="sidenav__item-content">
          <span class="sidenav__item-step label-sm">{{ item.label }}</span>
        </div>
        <span v-if="completedSteps.includes(item.step)" class="sidenav__item-check material-symbols-outlined">check_circle</span>
      </a>
    </nav>

    <!-- Footer Actions -->
    <div class="sidenav__footer">
      <button class="sidenav__new-btn btn-secondary" style="width:100%;justify-content:center;">
        <span class="material-symbols-outlined" style="font-size:16px">add</span>
        New Project
      </button>
      <div class="sidenav__links">
        <a href="#" class="sidenav__link label-sm">
          <span class="material-symbols-outlined" style="font-size:14px">help</span>
          Documentation
        </a>
        <a href="#" class="sidenav__link label-sm">
          <span class="material-symbols-outlined" style="font-size:14px">contact_support</span>
          Support
        </a>
      </div>
    </div>
  </aside>
</template>

<script setup>
defineProps({
  currentStep: { type: String, default: 'landing' },
  completedSteps: { type: Array, default: () => [] },
})
defineEmits(['navigate'])

const navItems = [
  { step: 'landing',    icon: 'home',                   label: 'Landing' },
  { step: 'graph',      icon: 'account_tree',            label: 'Graph Build' },
  { step: 'env',        icon: 'settings_input_component', label: 'Env Setup' },
  { step: 'simulation', icon: 'play_circle',             label: 'Simulation' },
  { step: 'report',     icon: 'analytics',               label: 'Report' },
]
</script>

<style scoped>
.sidenav {
  position: fixed;
  top: 60px;
  left: 0;
  bottom: 0;
  width: 240px;
  background: var(--surface-low);
  display: flex;
  flex-direction: column;
  padding: 1.5rem 0;
  z-index: 90;
  overflow-y: auto;
}

.sidenav__engine {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0 1.25rem 1.5rem;
  border-bottom: 1px solid var(--ghost-border);
  margin-bottom: 0.75rem;
}

.sidenav__engine-icon {
  width: 36px;
  height: 36px;
  border-radius: var(--radius-md);
  background: rgba(14, 63, 174, 0.25);
  display: flex;
  align-items: center;
  justify-content: center;
  border: 1px solid rgba(182, 196, 255, 0.15);
}

.sidenav__engine-name {
  font-size: 0.875rem;
  font-weight: 700;
  color: var(--primary);
  line-height: 1.2;
}

.sidenav__engine-ver {
  color: var(--text-muted);
  margin-top: 2px;
}

.sidenav__nav {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 2px;
  padding: 0 0.5rem;
}

.sidenav__item {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.75rem 0.875rem;
  border-radius: var(--radius-md);
  text-decoration: none;
  color: var(--text-secondary);
  transition: background var(--duration-fast), color var(--duration-fast);
  position: relative;
}
.sidenav__item:hover {
  background: var(--surface-container);
  color: var(--text-primary);
}

.sidenav__item--active {
  background: linear-gradient(90deg, rgba(255, 181, 158, 0.1) 0%, transparent 100%);
  color: var(--primary);
  border-right: 3px solid var(--primary-container);
}
.sidenav__item--active .sidenav__item-icon { color: var(--primary); }

.sidenav__item--completed { color: var(--text-secondary); }
.sidenav__item--completed:hover { color: var(--text-primary); }

.sidenav__item--locked { opacity: 0.4; cursor: not-allowed; }
.sidenav__item--locked:hover { background: transparent; color: var(--text-secondary); }

.sidenav__item-icon {
  font-size: 20px;
  flex-shrink: 0;
  transition: color var(--duration-fast);
}

.sidenav__item-content { flex: 1; }

.sidenav__item-step {
  font-size: 0.6875rem;
  letter-spacing: 0.08em;
}

.sidenav__item-check {
  font-size: 14px;
  color: #86EFAC;
  flex-shrink: 0;
}

.sidenav__footer {
  padding: 1.25rem 0.75rem 0;
  border-top: 1px solid var(--ghost-border);
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.sidenav__links {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  padding: 0 0.25rem;
}

.sidenav__link {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  color: var(--text-muted);
  text-decoration: none;
  font-size: 0.6875rem;
  letter-spacing: 0.06em;
  transition: color var(--duration-fast);
  padding: 0.375rem 0.5rem;
  border-radius: var(--radius-sm);
}
.sidenav__link:hover { color: var(--primary); }
</style>
