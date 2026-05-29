<script setup>
import { ref } from 'vue';

const props = defineProps({
  isOnline: {
    type: Boolean,
    required: true
  },
  outboxQueue: {
    type: Array,
    required: true
  }
});

const emit = defineEmits(['toggle-network', 'increment-stat', 'trigger-sync']);

const isSyncing = ref(false);

const handleSyncClick = () => {
  if (!props.isOnline || props.outboxQueue.length === 0 || isSyncing.value) return;
  
  isSyncing.value = true;
  
  // Progress loader duration 1.8s matching CSS rules
  setTimeout(() => {
    emit('trigger-sync');
    isSyncing.value = false;
  }, 1800);
};

const handleInjectClick = (channel) => {
  emit('increment-stat', channel);
};
</script>

<template>
  <div class="system-controller-card">
    <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom: 0.75rem; border-bottom: 1px solid var(--charcoal-100); padding-bottom: 0.5rem;">
      <h3 style="font-size:0.95rem; font-weight:800; margin:0;">Ecosystem Simulation Controller</h3>
      <span style="font-size:0.6rem; font-weight:800; background-color: var(--oxfam-green-light); color:var(--oxfam-green-dark); padding:2px 6px; border-radius:4px;">
        TEST DECK ACTIVE
      </span>
    </div>

    <div class="controller-panels-grid">
      
      <!-- Network control & Outbox queue inspector -->
      <div class="ctrl-box">
        <h4 style="font-size:0.75rem; font-weight:800; color:var(--charcoal-900);">1. SIMULATE NETWORK CONNECTIVITY</h4>
        
        <div style="display:flex; justify-content:space-between; align-items:center; margin: 0.25rem 0;">
          <span style="font-size:0.75rem; font-weight:700; color:var(--charcoal-700);">Network status:</span>
          <div class="network-switch">
            <span style="font-size:0.75rem; font-weight:800;" :style="{ color: props.isOnline ? 'var(--oxfam-green-dark)' : 'var(--gac-red)' }">
              {{ props.isOnline ? 'ONLINE' : 'OFFLINE' }}
            </span>
            <div 
              class="toggle-pill" 
              :class="{ online: props.isOnline }" 
              @click="emit('toggle-network')"
            >
              <span class="toggle-dot"></span>
            </div>
          </div>
        </div>

        <!-- Sync outbox bar indicator -->
        <div class="sync-bar-wrapper">
          <div class="sync-bar-label" style="font-size:0.68rem; margin-bottom:4px;">
            <span>IndexedDB Sync Outbox</span>
            <span style="font-weight:800;" :style="{ color: props.isOnline ? 'var(--oxfam-green-dark)' : 'var(--charcoal-600)' }">
              {{ isSyncing ? 'Synchronizing...' : props.isOnline ? 'Sync Active' : 'Suspended' }}
            </span>
          </div>
          <div class="sync-bar-outer" style="height:8px; border-radius:4px;">
            <div class="sync-bar-inner" :class="{ syncing: isSyncing }"></div>
          </div>
        </div>

        <!-- Trigger Outbox sync manually if connection is restored -->
        <button 
          v-if="props.outboxQueue.length > 0" 
          @click="handleSyncClick"
          class="barrier-submit-btn" 
          style="background-color: var(--oxfam-green); font-size:0.7rem; padding: 0.4rem 0.6rem; margin-top:0.25rem; border-radius:6px; cursor:pointer;"
          :disabled="!props.isOnline || isSyncing"
        >
          {{ !props.isOnline ? '🔌 Reconnect network to Sync' : isSyncing ? 'Syncing outbox...' : 'Process Local Outbox Sync' }}
        </button>
      </div>

      <!-- In-memory outbox logger list -->
      <div class="ctrl-box">
        <h4 style="font-size:0.75rem; font-weight:800; color:var(--charcoal-900);">2. LOCAL BROWSER OUTBOX QUEUE</h4>
        <div class="outbox-list" style="height: 90px;">
          <div v-if="props.outboxQueue.length === 0" class="outbox-empty" style="font-size:0.68rem; line-height:1.35; padding: 1rem 0.5rem;">
            Queue empty. Toggle network **Offline** to queue posts/reports locally in IndexedDB outbox.
          </div>
          <div 
            v-else 
            v-for="job in props.outboxQueue" 
            :key="job.id" 
            class="outbox-item"
            style="padding: 0.35rem 0.4rem; margin-bottom: 3px;"
          >
            <div style="display:flex; align-items:center; gap:4px; max-width:80%; overflow:hidden; text-overflow:ellipsis;">
              <span style="width:6px; height:6px; border-radius:50%; background-color: hsl(38, 92%, 50%); display:inline-block; flex-shrink:0;"></span>
              <span class="truncate" style="white-space:nowrap; font-size:0.65rem;">{{ job.label }}</span>
            </div>
            <span style="font-size:0.5rem; background-color: hsl(38, 92%, 92%); color: hsl(38, 92%, 35%); padding:2px 4px; border-radius:3px; font-weight:800;">
              QUEUED
            </span>
          </div>
        </div>
      </div>

      <!-- Secondary ecosystem channels simulator injectors -->
      <div class="ctrl-box" style="grid-column: span 2;">
        <h4 style="font-size:0.75rem; font-weight:800; color:var(--charcoal-900);">3. INJECT SECONDARY CHANNEL ENGAGEMENTS (INTEGRATED PIPELINE)</h4>
        <p style="font-size:0.68rem; color:var(--charcoal-600); line-height:1.35; margin-bottom: 0.4rem;">
          The Metabase Dashboard consolidates multiple streams. Tap buttons below to simulate automated metadata loads from third-party networks:
        </p>

        <div style="display:grid; grid-template-columns: repeat(3, 1fr); gap: 0.5rem;">
          <button @click="handleInjectClick('whatsapp')" class="inject-btn" style="font-size:0.68rem; padding: 0.5rem 0.25rem; font-weight:700; border-radius:8px;">
            💬 WhatsApp API
          </button>
          <button @click="handleInjectClick('facebook')" class="inject-btn" style="font-size:0.68rem; padding: 0.5rem 0.25rem; font-weight:700; border-radius:8px;">
            👍 Facebook Reach
          </button>
          <button @click="handleInjectClick('hotline')" class="inject-btn" style="font-size:0.68rem; padding: 0.5rem 0.25rem; font-weight:700; border-radius:8px;">
            📞 CCPF Helpline Call
          </button>
        </div>
      </div>

    </div>
  </div>
</template>
