<script setup>
import { ref } from 'vue';
import MobilePwa from './components/MobilePwa.vue';
import Dashboard from './components/Dashboard.vue';
import SystemController from './components/SystemController.vue';

// Global Sandbox States
const isOnline = ref(true);
const outboxQueue = ref([]);
const syncTrigger = ref(0);

// Prepopulated analytics databases (VillageReach base)
const barrierReports = ref([
  { id: 201, district: 'Balaka', location: 'Phalula Youth Unit', issue: 'Product Stock-out', details: 'No oral contraceptives available at the dispensary.', status: 'Actioned' },
  { id: 202, district: 'Lilongwe', location: 'Area 25 Clinic', issue: 'Service Denial', details: 'Youth turned away without consultation due to overcrowding.', status: 'Investigating' },
  { id: 203, district: 'Balaka', location: 'Liwonde Clinic', issue: 'Stigma/Harassment', details: 'Provider lectured client about pre-marital sex.', status: 'Investigating' }
]);

const kpiStats = ref({
  pwaUsers: 3421,
  whatsappThreads: 2185,
  fbReach: 14104,
  hotlineReferrals: 1294
});

// Event Handler: Toggle Network Simulator
const handleToggleNetwork = () => {
  isOnline.value = !isOnline.value;
};

// Event Handler: Add item to IndexedDB simulation outbox queue
const handleAddToOutbox = (job) => {
  outboxQueue.value.push(job);
};

// Event Handler: Push live forum post directly from online emulator
const handleAddForum = () => {
  kpiStats.value.pwaUsers += 1; // Increment interactive active installs
};

// Event Handler: Push live barrier report from online emulator
const handleAddBarrier = (report) => {
  barrierReports.value.unshift(report);
  kpiStats.value.pwaUsers += 1;
};

// Event Handler: Increment specific channel stats from control buttons
const handleIncrementStat = (channel) => {
  if (channel === 'whatsapp') {
    kpiStats.value.whatsappThreads += Math.floor(Math.random() * 5) + 1;
  } else if (channel === 'facebook') {
    kpiStats.value.fbReach += Math.floor(Math.random() * 12) + 5;
  } else if (channel === 'hotline') {
    kpiStats.value.hotlineReferrals += Math.floor(Math.random() * 3) + 1;
  }
};

// Inject USSD/SMS stock-out alert from BI controls
const handleInjectAlert = () => {
  const options = [
    { district: 'Balaka', location: 'Nkaya Clinic TA Msamala', issue: 'Product Stock-out', details: 'Automated USSD Stock signal: Zero condoms left.', status: 'Pending Review' },
    { district: 'Lilongwe', location: 'Mitundu Outpost TA Amidu', issue: 'Bribe Demand', details: 'CCPF Referral Alert: Fees asked for contraceptive injection.', status: 'Pending Review' }
  ];
  
  const picked = options[Math.floor(Math.random() * options.length)];
  const alertReport = {
    id: Date.now(),
    ...picked
  };

  barrierReports.value.unshift(alertReport);
};

// Event Handler: Process IndexedDB Outbox background synchronization
const handleTriggerSync = () => {
  if (outboxQueue.value.length === 0) return;

  // Sync queued items to BI database and active records
  outboxQueue.value.forEach(job => {
    if (job.type === 'barrier_report') {
      const report = job.payload;
      report.status = 'Actioned'; // Resolved during flush sync
      barrierReports.value.unshift(report);
    }
  });

  // Empty outbox logs
  outboxQueue.value = [];
  syncTrigger.value += 1; // Signal MobilePwa to sync local list
  
  // Dynamic metrics updates
  kpiStats.value.pwaUsers += 8;
  kpiStats.value.whatsappThreads += 4;
};

// Event Handler: Track casual user emulator clicks for metric feedback
const handlePwaClick = () => {
  kpiStats.value.pwaUsers += 1;
};
</script>

<template>
  <div>
    <!-- Top Global App Header -->
    <header class="top-header">
      <div class="top-logo-area">
        <div class="top-logo-circle">C</div>
        <div class="top-title-wrapper">
          <h1>CHOICE Digital Ecosystem Prototype</h1>
          <p>VillageReach digital innovation and co-design sandbox (2025–2028)</p>
        </div>
      </div>

      <div class="top-controls">
        <div style="font-size: 0.65rem; color:var(--charcoal-600); font-weight:700; text-align:right;">
          <span>Malawi CHOICE Cohort</span><br />
          <span>GAC & Oxfam Canada supported</span>
        </div>
      </div>
    </header>

    <!-- Master Layout Split Pane Grid -->
    <main class="sandbox-container">
      
      <!-- Column 1: Left Phone frame emulator -->
      <section class="emulator-wrapper">
        <MobilePwa 
          :isOnline="isOnline" 
          :outboxQueue="outboxQueue"
          :syncTrigger="syncTrigger"
          @add-to-outbox="handleAddToOutbox"
          @add-forum="handleAddForum"
          @add-barrier="handleAddBarrier"
          @pwa-click="handlePwaClick"
        />
      </section>

      <!-- Column 2: Right BI Dashboard & System Controller console -->
      <section class="dashboard-panel-wrapper">
        
        <!-- BI Metabase Dashboard panel -->
        <Dashboard 
          :barrierReports="barrierReports"
          :kpiStats="kpiStats"
          @inject-alert="handleInjectAlert"
        />

        <!-- Prototyping System Controller panel -->
        <SystemController 
          :isOnline="isOnline"
          :outboxQueue="outboxQueue"
          @toggle-network="handleToggleNetwork"
          @increment-stat="handleIncrementStat"
          @trigger-sync="handleTriggerSync"
        />

      </section>

    </main>

    <!-- Global Footer -->
    <footer class="app-footer">
      <p>© 2026 VillageReach • CHOICE Digital System Prototype • Developed for Oxfam Canada & Global Affairs Canada</p>
      <p style="margin-top:0.25rem; font-size: 0.65rem; color:var(--charcoal-600);">Designed by UI/UX Lead Consultant - Balaka & Lilongwe Field Trial Cohort</p>
    </footer>
  </div>
</template>
