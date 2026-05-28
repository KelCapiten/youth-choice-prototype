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
    <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom: 0.75rem; border-b: 1px solid var(--charcoal-100); padding-bottom: 0.5rem;">
      <h3 style="font-size:0.9rem; font-weight:800; margin:0;">Ecosystem Simulation Controller</h3>
      <span style="font-size:0.55rem; font-weight:800; background-color: var(--charcoal-100); color:var(--charcoal-700); padding:2px 6px; border-radius:4px;">
        TEST DECK ACTIVE
      </span>
    </div>

    <div class="controller-panels-grid">
      
      <!-- Network control & Outbox queue inspector -->
      <div class="ctrl-box">
        <h4>1. SIMULATE NETWORK CONNECTIVITY</h4>
        
        <div style="display:flex; justify-content:space-between; align-items:center; margin: 0.25rem 0;">
          <span style="font-size:0.7rem; font-weight:700; color:var(--charcoal-700);">Network status:</span>
          <div class="network-switch">
            <span style="font-weight:800;" :style="{ color: props.isOnline ? 'var(--oxfam-green-dark)' : 'var(--gac-red)' }">
              {{ props.isOnline ? 'ONLINE (4G)' : 'OFFLINE' }}
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
          <div class="sync-bar-label">
            <span>IndexedDB Sync Outbox</span>
            <span>{{ isSyncing ? 'Synchronizing...' : props.isOnline ? 'Online Sync Available' : 'Offline Suspended' }}</span>
          </div>
          <div class="sync-bar-outer">
            <div class="sync-bar-inner" :class="{ syncing: isSyncing }"></div>
          </div>
        </div>

        <!-- Trigger Outbox sync manually if connection is restored -->
        <button 
          v-if="props.outboxQueue.length > 0" 
          @click="handleSyncClick"
          class="barrier-submit-btn" 
          style="background-color: var(--oxfam-green); font-size:0.65rem; padding: 0.35rem 0.5rem; margin-top:0.25rem;"
          :disabled="!props.isOnline || isSyncing"
        >
          {{ !props.isOnline ? 'Reconnect Network to Flush Outbox' : isSyncing ? 'Piping data...' : 'Process Local Outbox Sync' }}
        </button>
      </div>

      <!-- In-memory outbox logger list -->
      <div class="ctrl-box">
        <h4>2. LOCAL BROWSER OUTBOX QUEUE</h4>
        <div class="outbox-list">
          <div v-if="props.outboxQueue.length === 0" class="outbox-empty">
            Queue empty. Toggle network **Offline** to queue posts/reports locally.
          </div>
          <div 
            v-else 
            v-for="job in props.outboxQueue" 
            :key="job.id" 
            class="outbox-item"
          >
            <div style="display:flex; align-items:center; gap:4px; max-width:80%; overflow:hidden; text-overflow:ellipsis;">
              <span style="width:5px; height:5px; border-radius:50%; background-color: hsl(38, 92%, 50%); display:inline-block;"></span>
              <span class="truncate" style="white-space:nowrap;">{{ job.label }}</span>
            </div>
            <span style="font-size:0.45rem; background-color: hsl(38, 92%, 92%); color: hsl(38, 92%, 35%); padding:1px 3px; border-radius:2px; font-weight:800;">
              QUEUED
            </span>
          </div>
        </div>
      </div>

      <!-- Secondary ecosystem channels simulator injectors -->
      <div class="ctrl-box" style="grid-column: span 2;">
        <h4>3. INJECT SECONDARY CHANNEL ENGAGEMENTS (INTEGRATED PIPELINE)</h4>
        <p style="font-size:0.6rem; color:var(--charcoal-600); line-height:1.2; margin-bottom: 0.25rem;">
          The Metabase Dashboard consolidates multiple streams. Tap buttons below to simulate automated metadata loads from third-party networks:
        </p>

        <div style="display:grid; grid-template-columns: repeat(3, 1fr); gap: 0.5rem;">
          <button @click="handleInjectClick('whatsapp')" class="inject-btn">
            💬 WhatsApp Group Thread
          </button>
          <button @click="handleInjectClick('facebook')" class="inject-btn">
            👍 Facebook Reaction Post
          </button>
          <button @click="handleInjectClick('hotline')" class="inject-btn">
            📞 CCPF Hotline Call metadata
          </button>
        </div>
      </div>

    </div>
  </div>
</template>
