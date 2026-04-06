<template>
  <div class="interact">
    <div class="interact__split">
      <!-- LEFT: Report Reference Panel -->
      <div class="interact__ref card">
        <div class="interact__ref-header">
          <span class="material-symbols-outlined" style="font-size:16px;color:var(--secondary)">analytics</span>
          <span class="label-sm" style="color:var(--text-muted)">Simulation Report</span>
        </div>

        <div v-if="reportSummary" class="interact__ref-content">
          <div class="interact__ref-meta">
            <span class="chip chip-orange">Prediction Report</span>
            <span class="font-mono label-sm" style="color:var(--text-muted)">{{ reportId }}</span>
          </div>
          <h3 class="interact__ref-title font-headline">{{ reportSummary.title }}</h3>
          <p class="interact__ref-desc">{{ reportSummary.summary }}</p>

          <div class="section-divider"></div>

          <div class="interact__ref-sections">
            <div
              v-for="(s, i) in reportSummary.sections"
              :key="i"
              class="interact__ref-section"
              :class="{'interact__ref-section--active': activeSectionRef === i}"
              @click="activeSectionRef = i; askAboutSection(s)"
            >
              <span class="interact__ref-section-num font-mono">{{ String(i + 1).padStart(2, '0') }}</span>
              <span class="interact__ref-section-title">{{ s.title }}</span>
              <span class="material-symbols-outlined" style="font-size:14px;color:var(--text-muted)">chat_bubble</span>
            </div>
          </div>
        </div>

        <div v-else class="interact__ref-empty">
          <span class="label-sm" style="color:var(--text-muted)">No report loaded</span>
        </div>
      </div>

      <!-- RIGHT: Chat Interface -->
      <div class="interact__chat card">
        <!-- Chat Header -->
        <div class="interact__chat-header">
          <div class="interact__chat-title-row">
            <div class="ai-chip">
              <span class="ai-chip-dot"></span>
              <span class="label-sm" style="color:var(--secondary)">Simulation Oracle · Online</span>
            </div>
            <button class="btn-ghost" @click="clearChat" title="Clear conversation">
              <span class="material-symbols-outlined" style="font-size:18px">delete_sweep</span>
            </button>
          </div>
          <p class="interact__chat-subtitle">
            Ask questions about the simulated world. Query agent behaviors, emergent patterns, or generate hypothetical scenarios.
          </p>
        </div>

        <!-- Suggested Prompts (shown when empty) -->
        <div v-if="!messages.length" class="interact__suggestions">
          <span class="label-sm" style="color:var(--text-muted)">Suggested queries</span>
          <div class="interact__suggestion-grid">
            <button
              v-for="s in suggestions"
              :key="s"
              class="interact__suggestion"
              @click="sendMessage(s)"
            >
              <span class="material-symbols-outlined" style="font-size:14px;color:var(--primary);flex-shrink:0">auto_awesome</span>
              {{ s }}
            </button>
          </div>
        </div>

        <!-- Messages -->
        <div class="interact__messages" ref="messagesRef">
          <div
            v-for="(msg, i) in messages"
            :key="i"
            class="interact__message"
            :class="`interact__message--${msg.role}`"
          >
            <div class="interact__message-avatar">
              <span v-if="msg.role === 'user'" class="material-symbols-outlined" style="font-size:16px">person</span>
              <span v-else class="material-symbols-outlined icon-filled" style="font-size:16px">smart_toy</span>
            </div>
            <div class="interact__message-body">
              <div class="interact__message-role label-sm">{{ msg.role === 'user' ? 'You' : 'Simulation Oracle' }}</div>
              <div class="interact__message-content" v-html="renderContent(msg.content)"></div>
              <div class="interact__message-time label-sm">{{ msg.time }}</div>
            </div>
          </div>

          <!-- Typing indicator -->
          <div v-if="isTyping" class="interact__message interact__message--assistant">
            <div class="interact__message-avatar">
              <span class="material-symbols-outlined icon-filled" style="font-size:16px">smart_toy</span>
            </div>
            <div class="interact__message-body">
              <div class="interact__message-role label-sm">Simulation Oracle</div>
              <div class="interact__typing">
                <span class="interact__typing-dot" style="animation-delay:0ms"></span>
                <span class="interact__typing-dot" style="animation-delay:160ms"></span>
                <span class="interact__typing-dot" style="animation-delay:320ms"></span>
              </div>
            </div>
          </div>
        </div>

        <!-- Input Area -->
        <div class="interact__input-area">
          <!-- Context chips -->
          <div v-if="contextChips.length" class="interact__context-chips">
            <span
              v-for="(chip, i) in contextChips"
              :key="i"
              class="chip chip-blue"
              style="cursor:pointer"
              @click="removeChip(i)"
            >
              <span class="material-symbols-outlined" style="font-size:11px">label</span>
              {{ chip }}
              <span class="material-symbols-outlined" style="font-size:11px">close</span>
            </span>
          </div>

          <div class="interact__input-row">
            <textarea
              v-model="inputText"
              class="input-field interact__textarea"
              rows="2"
              placeholder="Ask about agent behavior, emergent patterns, or simulate a hypothetical…"
              @keydown.enter.exact.prevent="sendCurrentMessage"
              @keydown.enter.shift.exact="inputText += '\n'"
            ></textarea>
            <div class="interact__input-actions">
              <button class="btn-ghost interact__attach-btn" @click="addContextChip" title="Add context tag">
                <span class="material-symbols-outlined" style="font-size:20px">add_circle</span>
              </button>
              <button
                class="btn-primary interact__send-btn"
                :disabled="!inputText.trim() || isTyping"
                @click="sendCurrentMessage"
              >
                <span class="material-symbols-outlined" style="font-size:20px">send</span>
              </button>
            </div>
          </div>
          <div class="interact__input-hint label-sm">
            Press <kbd class="interact__kbd">Enter</kbd> to send · <kbd class="interact__kbd">Shift+Enter</kbd> for new line
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, nextTick, onMounted } from 'vue'

const props = defineProps({
  reportId: { type: String, default: 'REF-2024-X92' },
})

const messagesRef = ref(null)
const inputText = ref('')
const messages = ref([])
const isTyping = ref(false)
const contextChips = ref([])
const activeSectionRef = ref(null)

const reportSummary = ref({
  title: 'Emergent Social Dynamics: Predictive Intelligence Report',
  summary: 'Analysis of autonomous agent behavior reveals significant clustering around key opinion leaders, with 73% of information cascades originating from fewer than 8% of agents.',
  sections: [
    { title: 'Executive Summary' },
    { title: 'Information Cascade Patterns' },
    { title: 'Key Opinion Leader Identification' },
    { title: 'Sentiment Trajectory Analysis' },
    { title: 'Cross-Platform Dynamics' },
    { title: 'Predictive Scenarios' },
    { title: 'Recommendations' },
  ],
})

const suggestions = [
  'Which agent had the most influence on information cascades?',
  'What would happen if Agent_007 was removed from the simulation?',
  'Summarize the cross-platform dynamics in simple terms',
  'What were the main sentiment turning points?',
  'Generate a hypothetical scenario where 3 KOLs coordinate messaging',
  'Which agents showed the most adaptive behavior over time?',
]

const demoResponses = [
  'Based on the simulation data, **Agent_007** emerged as the highest-influence node with a betweenness centrality score of **0.73**. This agent was active across both Info Plaza and Topic Community, acting as a critical bridge node. Approximately **31% of all cascades** passed through this agent at some point in their propagation chain.\n\nKey behavioral traits observed:\n- Consistently responded to low-engagement content within 2 rounds\n- Maintained positive sentiment framing even during Phase 2 polarization\n- Cross-posted 67% of content across both platforms',
  'Running a counterfactual model with Agent_007 removed from the simulation:\n\n**Predicted outcomes:**\n- Overall cascade velocity decreases by **41%**\n- Cross-platform amplification drops from 2.3× to 1.4×\n- Information ecosystem fragments into 2–3 isolated clusters by round 600\n- Phase 2 polarization extends from rounds 300–800 to potentially 300–1,100\n\nThe void left by Agent_007 would likely be partially filled by Agents 023 and 041, but their combined reach only covers **58%** of Agent_007\'s network position.',
  'Cross-platform dynamics in simple terms:\n\n**Info Plaza** (Twitter-like) is where ideas are **born** — rapid, short-form content spreads quickly but dies fast. Think of it as the spark.\n\n**Topic Community** (Reddit-like) is where ideas are **refined** — longer discussions add depth and credibility. Think of it as the fuel.\n\n**Bridge agents** (active on both) are the **match** — they carry validated ideas from Topic Community back to Info Plaza, triggering second-wave amplification that\'s typically 2.3× stronger than the original cascade.',
  'The simulation shows three distinct **sentiment turning points**:\n\n1. **Round 78** — Initial optimism peaks (+0.41 valence) as early content resonates broadly\n2. **Round 423** — Sharp negativity spike (valence -0.28) triggered by information scarcity and KOL conflict\n3. **Round 847** — Synthetic consensus emerges (+0.38 valence) as echo chambers stabilize around shared narratives\n\nThe Phase 2 dip (Round 300–800) represents the highest opportunity window for narrative intervention with minimum resource cost.',
]

let responseIndex = 0

function getTime() {
  const d = new Date()
  return `${String(d.getHours()).padStart(2,'0')}:${String(d.getMinutes()).padStart(2,'0')}`
}

async function sendMessage(text) {
  if (!text.trim() || isTyping.value) return

  messages.value.push({ role: 'user', content: text, time: getTime() })
  inputText.value = ''
  await scrollToBottom()

  isTyping.value = true
  await new Promise(r => setTimeout(r, 1200 + Math.random() * 800))

  const response = demoResponses[responseIndex % demoResponses.length]
  responseIndex++

  isTyping.value = false
  messages.value.push({ role: 'assistant', content: response, time: getTime() })
  await scrollToBottom()
}

function sendCurrentMessage() { sendMessage(inputText.value) }

function askAboutSection(section) {
  sendMessage(`Tell me more about the "${section.title}" section of the report.`)
}

async function scrollToBottom() {
  await nextTick()
  if (messagesRef.value) {
    messagesRef.value.scrollTop = messagesRef.value.scrollHeight
  }
}

function clearChat() { messages.value = []; responseIndex = 0 }

function addContextChip() {
  const tags = ['KOL Analysis', 'Phase 2', 'Info Plaza', 'Agent_007', 'Cascades', 'Sentiment']
  const available = tags.filter(t => !contextChips.value.includes(t))
  if (available.length) contextChips.value.push(available[0])
}

function removeChip(i) { contextChips.value.splice(i, 1) }

function renderContent(text) {
  return text
    .replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')
    .replace(/\*(.*?)\*/g, '<em>$1</em>')
    .replace(/`(.*?)`/g, '<code class="interact-code">$1</code>')
    .replace(/\n\n/g, '</p><p>')
    .replace(/^- (.+)$/gm, '<li>$1</li>')
    .replace(/(<li>.*<\/li>)/gs, '<ul>$1</ul>')
    .replace(/\n/g, '<br>')
}
</script>

<style scoped>
.interact { height: 100%; }

.interact__split {
  display: grid;
  grid-template-columns: 300px 1fr;
  gap: 1.25rem;
  align-items: start;
  height: 100%;
}

/* Reference Panel */
.interact__ref {
  position: sticky;
  top: 80px;
  max-height: calc(100vh - 120px);
  overflow-y: auto;
}

.interact__ref-header {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 1.25rem 1.25rem 0.75rem;
  border-bottom: 1px solid var(--ghost-border);
}

.interact__ref-content { padding: 1.25rem; }

.interact__ref-meta {
  display: flex;
  align-items: center;
  gap: 0.625rem;
  margin-bottom: 0.875rem;
  flex-wrap: wrap;
}

.interact__ref-title {
  font-size: 0.9375rem;
  color: var(--text-primary);
  margin-bottom: 0.625rem;
  line-height: 1.4;
}

.interact__ref-desc {
  font-size: 0.75rem;
  color: var(--text-muted);
  line-height: 1.65;
}

.interact__ref-sections {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.interact__ref-section {
  display: flex;
  align-items: center;
  gap: 0.625rem;
  padding: 0.625rem 0.75rem;
  border-radius: var(--radius-md);
  cursor: pointer;
  transition: background var(--duration-fast);
}
.interact__ref-section:hover { background: var(--surface-container-high); }
.interact__ref-section--active {
  background: rgba(255, 90, 31, 0.08);
  border: 1px solid rgba(255, 90, 31, 0.15);
}

.interact__ref-section-num {
  font-size: 0.6875rem;
  color: var(--text-muted);
  min-width: 20px;
}

.interact__ref-section-title {
  flex: 1;
  font-size: 0.75rem;
  font-weight: 600;
  color: var(--text-secondary);
}
.interact__ref-section--active .interact__ref-section-title { color: var(--primary); }

.interact__ref-empty {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 3rem;
}

/* Chat */
.interact__chat {
  display: flex;
  flex-direction: column;
  height: calc(100vh - 120px);
  overflow: hidden;
}

.interact__chat-header {
  padding: 1.25rem 1.5rem 1rem;
  border-bottom: 1px solid var(--ghost-border);
  flex-shrink: 0;
}

.interact__chat-title-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 0.5rem;
}

.interact__chat-subtitle {
  font-size: 0.75rem;
  color: var(--text-muted);
  line-height: 1.6;
}

/* Suggestions */
.interact__suggestions {
  padding: 1.25rem 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
  border-bottom: 1px solid var(--ghost-border);
  flex-shrink: 0;
}

.interact__suggestion-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 0.5rem;
}

.interact__suggestion {
  display: flex;
  align-items: flex-start;
  gap: 0.5rem;
  padding: 0.75rem;
  background: var(--surface-low);
  border: 1px solid var(--ghost-border);
  border-radius: var(--radius-md);
  text-align: left;
  font-size: 0.75rem;
  color: var(--text-secondary);
  cursor: pointer;
  font-family: var(--font-body);
  line-height: 1.5;
  transition: all var(--duration-fast);
}
.interact__suggestion:hover {
  background: var(--surface-container-high);
  border-color: rgba(255, 90, 31, 0.25);
  color: var(--text-primary);
}

/* Messages */
.interact__messages {
  flex: 1;
  overflow-y: auto;
  padding: 1.25rem 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
}

.interact__message {
  display: flex;
  gap: 0.875rem;
  animation: fade-up 250ms var(--ease-out) both;
}

.interact__message--user { flex-direction: row-reverse; }

.interact__message-avatar {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  margin-top: 2px;
}
.interact__message--user .interact__message-avatar {
  background: var(--gradient-primary);
  color: var(--on-primary);
}
.interact__message--assistant .interact__message-avatar {
  background: rgba(14, 63, 174, 0.25);
  border: 1px solid rgba(182, 196, 255, 0.2);
  color: var(--secondary);
}

.interact__message-body {
  max-width: 72%;
  display: flex;
  flex-direction: column;
  gap: 0.375rem;
}
.interact__message--user .interact__message-body { align-items: flex-end; }

.interact__message-role {
  color: var(--text-muted);
  letter-spacing: 0.04em;
}

.interact__message-content {
  padding: 0.875rem 1.125rem;
  border-radius: var(--radius-xl);
  font-size: 0.875rem;
  line-height: 1.75;
}
.interact__message--user .interact__message-content {
  background: var(--gradient-primary);
  color: var(--on-primary-container);
  border-bottom-right-radius: var(--radius-sm);
}
.interact__message--assistant .interact__message-content {
  background: var(--surface-container);
  color: var(--text-secondary);
  border: 1px solid var(--ghost-border);
  border-bottom-left-radius: var(--radius-sm);
}

.interact__message-time { color: var(--text-muted); }

/* Typing */
.interact__typing {
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 0.75rem 1rem;
  background: var(--surface-container);
  border: 1px solid var(--ghost-border);
  border-radius: var(--radius-xl);
  border-bottom-left-radius: var(--radius-sm);
}

.interact__typing-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: var(--secondary);
  animation: typing-bounce 1s ease-in-out infinite;
}
@keyframes typing-bounce {
  0%,100% { transform: translateY(0); opacity: .5; }
  50%      { transform: translateY(-4px); opacity: 1; }
}

/* Input Area */
.interact__input-area {
  padding: 1rem 1.5rem 1.25rem;
  border-top: 1px solid var(--ghost-border);
  display: flex;
  flex-direction: column;
  gap: 0.625rem;
  flex-shrink: 0;
}

.interact__context-chips {
  display: flex;
  flex-wrap: wrap;
  gap: 0.375rem;
}

.interact__input-row {
  display: flex;
  gap: 0.625rem;
  align-items: flex-end;
}

.interact__textarea {
  flex: 1;
  resize: none;
  font-size: 0.875rem;
  min-height: 52px;
  max-height: 140px;
  line-height: 1.6;
}

.interact__input-actions {
  display: flex;
  flex-direction: column;
  gap: 0.375rem;
}

.interact__attach-btn { padding: 0.375rem; }

.interact__send-btn {
  padding: 0.5rem;
  border-radius: var(--radius-md);
}
.interact__send-btn:disabled {
  background: var(--surface-container-high);
  color: var(--text-muted);
  box-shadow: none;
  filter: none;
  transform: none;
  cursor: not-allowed;
}

.interact__input-hint {
  color: var(--text-muted);
  font-size: 0.625rem;
  letter-spacing: 0.04em;
}

.interact__kbd {
  display: inline-flex;
  padding: 0.0625rem 0.25rem;
  background: var(--surface-container-high);
  border: 1px solid var(--ghost-border);
  border-radius: 3px;
  font-size: 0.625rem;
  font-family: var(--font-mono);
  color: var(--text-secondary);
}

:deep(.interact-code) {
  font-family: var(--font-mono);
  font-size: 0.8em;
  background: var(--surface-container-high);
  padding: 1px 4px;
  border-radius: 3px;
  color: var(--secondary);
}
:deep(.interact__message--assistant .interact__message-content ul) {
  padding-left: 1.125rem;
  margin: 0.375rem 0;
}
:deep(.interact__message--assistant .interact__message-content li) { margin-bottom: 0.25rem; }
:deep(.interact__message--assistant .interact__message-content strong) {
  color: var(--text-primary);
  font-weight: 700;
}

@media (max-width: 1100px) {
  .interact__split { grid-template-columns: 1fr; }
  .interact__ref { position: static; max-height: 300px; }
}
</style>
