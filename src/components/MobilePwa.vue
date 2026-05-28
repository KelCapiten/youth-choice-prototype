<script setup>
import { ref, computed, watch, onMounted } from 'vue';

const props = defineProps({
  isOnline: {
    type: Boolean,
    required: true
  },
  outboxQueue: {
    type: Array,
    required: true
  },
  syncTrigger: {
    type: Number,
    default: 0
  }
});

const emit = defineEmits(['add-to-outbox', 'add-barrier', 'add-forum', 'pwa-click']);

// App Internal States
const onboardingTier = ref(null); // null, 1 (anonymous), 2 (registered)
const registeredPhone = ref('');
const registeredDistrict = ref('');
const currentUser = ref('');

const activeTab = ref('home'); // home, map, chat, report
const discreetMode = ref(false);
const quickExitActive = ref(false);

const selectedDistrict = ref('balaka');
const selectedTa = ref('kalembo');
const selectedFacility = ref(null);

const chatInput = ref('');
const activeAudioText = ref('');
const audioToastActive = ref(false);

const barrierIssue = ref('');
const barrierDistrict = ref('');
const barrierLandmark = ref('');
const barrierDetails = ref('');

// Speech Synthesis
let speechSynth = typeof window !== 'undefined' ? window.speechSynthesis : null;
let activeUtterance = null;

// Local Data Models (Re-populated and reacted to)
const forumPosts = ref([
  { id: 101, username: 'BalakaStar77', message: 'Hello, is the youth friendly clinic open at Phalula today?', timestamp: '10:14 AM', district: 'Balaka', status: 'synced' },
  { id: 102, username: 'ChamboRunner', message: 'Yes, I was there yesterday. They have condoms and implants in stock!', timestamp: '10:25 AM', district: 'Balaka', status: 'synced' },
  { id: 103, username: 'ZombaGuide', message: 'Remember you can also dial 811 for the CCPF hotline for direct clinical advice.', timestamp: '10:30 AM', district: 'Lilongwe', status: 'synced' }
]);

// Map Facility Data
const mapFacilities = {
  balaka: [
    { id: 1, name: 'Phalula Youth Unit', x: 25, y: 35, ta: 'kalembo', implants: 'IN STOCK', pills: 'IN STOCK', condoms: 'STOCKOUT', type: 'Youth Friendly Support' },
    { id: 2, name: 'Nkaya Clinic Health post', x: 65, y: 55, ta: 'msamala', implants: 'STOCKOUT', pills: 'IN STOCK', condoms: 'IN STOCK', type: 'Family Planning Hub' },
    { id: 3, name: 'Toleza Clinic Support', x: 45, y: 75, ta: 'amidu', implants: 'IN STOCK', pills: 'STOCKOUT', condoms: 'IN STOCK', type: 'Zero Stigma Zone' }
  ],
  lilongwe: [
    { id: 4, name: 'Kabudula Health Centre', x: 30, y: 40, ta: 'kalembo', implants: 'IN STOCK', pills: 'IN STOCK', condoms: 'IN STOCK', type: 'Youth Friendly Support' },
    { id: 5, name: 'Area 25 Youth Clinic', x: 70, y: 30, ta: 'msamala', implants: 'STOCKOUT', pills: 'STOCKOUT', condoms: 'IN STOCK', type: 'Family Planning Hub' },
    { id: 6, name: 'Mitundu Health Outpost', x: 50, y: 70, ta: 'amidu', implants: 'IN STOCK', pills: 'IN STOCK', condoms: 'STOCKOUT', type: 'Zero Stigma Zone' }
  ]
};

// Filtered Map Pins based on District and Traditional Authority
const activePins = computed(() => {
  const list = mapFacilities[selectedDistrict.value] || [];
  return list.filter(facility => facility.ta === selectedTa.value);
});

// Watch sync trigger from parent to resolve pending outbox logs locally
watch(() => props.syncTrigger, () => {
  forumPosts.value.forEach(post => {
    if (post.status === 'pending') {
      post.status = 'synced';
    }
  });
});

// Dynamic clock simulation
const phoneTime = ref('10:45 AM');
onMounted(() => {
  setInterval(() => {
    const now = new Date();
    phoneTime.value = now.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
  }, 1000);
});

// Trigger dynamic onboarding pseudonyms
const handleOnboarding = (tier) => {
  emit('pwa-click');
  onboardingTier.value = tier;
  if (tier === 1) {
    const prefixes = ["BalakaStar", "ChamboRunner", "PhalulaChampion", "LilongweYouth", "YaoProtector"];
    const suffix = Math.floor(Math.random() * 90) + 10;
    currentUser.value = prefixes[Math.floor(Math.random() * prefixes.length)] + suffix;
    triggerAlert("Anonymous Privacy On", `Child consent gate bypassed under Section 17 of Malawian DPA. Welcome anonymous explorer: @${currentUser.value}`, 'success');
  } else {
    currentUser.value = "YouthLeaderPlus";
  }
};

const completeRegisteredOnboarding = () => {
  if (!registeredPhone.value || !registeredDistrict.value) {
    triggerAlert("Invalid Fields", "Please specify a phone number and region to secure active SMS access.", "error");
    return;
  }
  currentUser.value = "User_" + registeredPhone.value.slice(-4);
  onboardingTier.value = 2;
  triggerAlert("Verified Portal Enabled", `Tier 2 access granted. Fully authorized under Section 16 written consent guidelines. Welcome: @${currentUser.value}`, 'success');
};

// Toggle Discreet Mode
const triggerDiscreetMode = () => {
  emit('pwa-click');
  discreetMode.value = !discreetMode.value;
  if (discreetMode.value) {
    triggerAlert("Discreet Mode ON", "All sensitive titles replaced with standard secondary-school Cellular Biology chapters.", "info");
  } else {
    triggerAlert("Discreet Mode OFF", "Standard Sexual & Reproductive Health articles restored.", "success");
  }
};

// Quick Exit Function
const triggerQuickExit = () => {
  emit('pwa-click');
  quickExitActive.value = true;
  stopAudio();
  triggerAlert("Safety Shield Active", "Quick exit triggered. All active panels hidden instantly with academic masks.", "success");
};

// Play audio articles using HTML5 SpeechSynthesis
const speakArticle = (textToRead) => {
  emit('pwa-click');
  stopAudio();
  audioToastActive.value = true;
  activeAudioText.value = textToRead;

  if (speechSynth) {
    activeUtterance = new SpeechSynthesisUtterance(textToRead);
    activeUtterance.rate = 0.95; // Friendly
    activeUtterance.pitch = 1.1; // Youthful
    activeUtterance.onend = () => {
      audioToastActive.value = false;
    };
    speechSynth.speak(activeUtterance);
  }
};

const stopAudio = () => {
  if (speechSynth) {
    speechSynth.cancel();
  }
  audioToastActive.value = false;
};

// Submit message inside moderated forums
const postMessage = () => {
  emit('pwa-click');
  if (!chatInput.value.trim()) return;

  const msg = {
    id: Date.now(),
    username: currentUser.value || 'AnonymousYouth',
    message: chatInput.value.trim(),
    timestamp: 'Just Now',
    district: selectedDistrict.value.charAt(0).toUpperCase() + selectedDistrict.value.slice(1),
    status: props.isOnline ? 'synced' : 'pending'
  };

  forumPosts.value.push(msg);

  if (!props.isOnline) {
    emit('add-to-outbox', {
      id: msg.id,
      type: 'forum_post',
      payload: msg,
      label: `Chat Post: "${msg.message.substring(0, 18)}..."`
    });
    triggerAlert("Offline Action Queued", "Simulated message stored locally in IndexedDB. Will sync when server connection returns.", "info");
  } else {
    emit('add-forum', msg);
    triggerAlert("Live Message Posted", "Comment parsed and sent to BI dashboard Metabase aggregates.", "success");
  }

  chatInput.value = '';
};

// Submit Access Barrier form
const selectBarrier = (type) => {
  emit('pwa-click');
  barrierIssue.value = type;
};

const postBarrierReport = () => {
  emit('pwa-click');
  if (!barrierIssue.value || !barrierDistrict.value || !barrierLandmark.value) {
    triggerAlert("Incomplete Form", "Please select a barrier type and specify geographical landmark contexts.", "error");
    return;
  }

  const report = {
    id: Date.now(),
    district: barrierDistrict.value,
    location: barrierLandmark.value,
    issue: barrierIssue.value,
    details: barrierDetails.value || "Anonymous access refusal details.",
    status: props.isOnline ? 'Investigating' : 'pending_outbox'
  };

  if (!props.isOnline) {
    emit('add-to-outbox', {
      id: report.id,
      type: 'barrier_report',
      payload: report,
      label: `Barrier Alert: [${report.issue}] at ${report.location}`
    });
    triggerAlert("Secure Local Queue Active", "Network offline. Barrier logged securely using background service worker queues.", "info");
  } else {
    emit('add-barrier', report);
    triggerAlert("Access Barrier Received", "Incident successfully logged! Monitoring units and regional advisers alerted.", "success");
  }

  // Clear
  barrierIssue.value = '';
  barrierDistrict.value = '';
  barrierLandmark.value = '';
  barrierDetails.value = '';
};

// Global Simulation Alert Box (Standardized)
const showModal = ref(false);
const modalTitle = ref('');
const modalMsg = ref('');
const modalType = ref('success');

const triggerAlert = (title, message, type) => {
  modalTitle.value = title;
  modalMsg.value = message;
  modalType.value = type;
  showModal.value = true;
};

const handleNavClick = (tab) => {
  emit('pwa-click');
  activeTab.value = tab;
};
</script>

<template>
  <div class="phone-shell" :class="{ 'discreet-active': discreetMode }">
    <!-- Phone Notch Speaker element -->
    <div class="phone-notch">
      <div class="phone-speaker"></div>
    </div>

    <!-- Screen Viewport -->
    <div class="phone-screen">
      
      <!-- Top phone stats status bar -->
      <div class="phone-status-bar">
        <span>{{ phoneTime }}</span>
        <div class="phone-status-icons">
          <!-- Network Signal Icon -->
          <svg v-if="props.isOnline" style="width:14px; height:14px;" viewBox="0 0 24 24" fill="currentColor" class="text-oxfam">
            <path d="M12 3c-4.97 0-9 4.03-9 9 0 2.12.74 4.07 1.97 5.61L4.35 19.4c-.39.39-.39 1.02 0 1.41.39.39 1.02.39 1.41 0l1.79-1.79C9.09 19.61 10.48 20 12 20c4.97 0 9-4.03 9-9s-4.03-9-9-9zm0 15c-3.31 0-6-2.69-6-6s2.69-6 6-6 6 2.69 6 6-2.69 6-6 6z"/>
          </svg>
          <svg v-else style="width:14px; height:14px;" viewBox="0 0 24 24" fill="currentColor" class="text-charcoal-600">
            <path d="M23.64 20.38l-2.73-2.73C21.82 16.14 22 14.12 22 12c0-5.52-4.48-10-10-10-2.12 0-4.14.18-5.65.91L3.62 1.18 1.18 3.62l19.82 19.82 2.64-3.06zM12 4c4.41 0 8 3.59 8 8 0 1.57-.46 3.03-1.24 4.26L5.74 5.24C6.97 4.46 8.43 4 12 4z"/>
          </svg>
          <span style="font-size: 8px;">{{ props.isOnline ? '4G' : 'OFFLINE' }}</span>
        </div>
      </div>

      <!-- App Header Bar -->
      <header class="pwa-app-header" v-if="!quickExitActive">
        <div class="pwa-logo-title">
          <div class="pwa-app-icon">
            <svg v-if="discreetMode" style="width:12px; height:12px;" fill="none" stroke="currentColor" stroke-width="2.5" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" d="M12 6.253v13m0-13C10.832 5.477 9.246 5 7.5 5S4.168 5.477 3 6.253v13C4.168 18.477 5.754 18 7.5 18s3.332.477 4.5 1.253m0-13C13.168 5.477 14.754 5 16.5 5c1.747 0 3.332.477 4.5 1.253v13C19.832 18.477 18.247 18 16.5 18c-1.746 0-3.332.477-4.5 1.253"/>
            </svg>
            <svg v-else style="width:12px; height:12px;" fill="none" stroke="currentColor" stroke-width="2.5" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" d="M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z"/>
            </svg>
          </div>
          <div class="pwa-title-area">
            <h2 v-if="discreetMode">Youth Learn</h2>
            <h2 v-else>Youth Choice</h2>
            <p v-if="discreetMode">Secondary Biology Hub</p>
            <p v-else>SRHR Safe Space • Malawi</p>
          </div>
        </div>

        <div class="pwa-header-controls" v-if="onboardingTier !== null">
          <!-- Discreet mode toggle -->
          <button @click="triggerDiscreetMode" class="pwa-icon-btn" title="Toggle Discreet Mode">
            <svg style="width:16px; height:16px;" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"/>
              <path stroke-linecap="round" stroke-linejoin="round" d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z"/>
            </svg>
          </button>
          
          <!-- Quick Exit Button -->
          <button @click="triggerQuickExit" class="quick-exit-btn">Exit</button>
        </div>
      </header>

      <!-- Scrollable app screen panels -->
      <div class="pwa-viewport" v-if="!quickExitActive">
        
        <!-- ==========================================
             SCREEN: ONBOARDING FLOW
             ========================================== -->
        <div v-if="onboardingTier === null" class="onboarding-wrap">
          <div>
            <h3 style="font-size: 1.4rem; margin-bottom: 0.25rem;">Tikulandirani!</h3>
            <p style="font-size: 0.75rem; color: var(--charcoal-600);">Welcome to your secure digital safe space for adolescent wellness and guidance.</p>
          </div>

          <div class="legal-badge">
            <svg style="width:20px; height:20px; flex-shrink:0;" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" d="M9 12l2 2 4-4m5.618-4.016A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016z"/>
            </svg>
            <div>
              <strong style="font-size: 0.7rem; display:block;">Malawi DPA 2024 Legal Shield</strong>
              <span style="font-size: 0.6rem; line-height: 1.2;">Your privacy is legally protected. Tier 1 requires zero identification or guardian phone verification inputs.</span>
            </div>
          </div>

          <div style="width: 100%; display:flex; flex-direction:column; gap: 0.75rem;">
            <!-- Tier 1 Select -->
            <button @click="handleOnboarding(1)" class="onboarding-btn">
              <h4>Tier 1: 100% Anonymous <span class="badge-tag">Under 18</span></h4>
              <p>Instant access. No real names or contact tracking logs. Autogenerates secure local handles.</p>
            </button>

            <!-- Tier 2 Select -->
            <button @click="handleOnboarding(22)" class="onboarding-btn">
              <h4>Tier 2: Registered Access <span class="badge-tag blue">Over 18</span></h4>
              <p>Opt-in SMS health notifications, interactive postings, and customized tracking dashboards.</p>
            </button>
          </div>

          <!-- Tier 2 Form fields if active -->
          <div v-if="onboardingTier === 22" style="width:100%; text-align: left;" class="pwa-card">
            <h4 style="font-size:0.75rem; margin-bottom: 0.5rem; text-transform: uppercase;">Register Verified Account</h4>
            <div style="display:flex; flex-direction:column; gap:0.5rem;">
              <input v-model="registeredPhone" type="tel" placeholder="Phone (+265...)" class="chat-field" />
              <select v-model="registeredDistrict" class="map-control-select">
                <option value="">Select District</option>
                <option value="Balaka">Balaka</option>
                <option value="Lilongwe">Lilongwe</option>
              </select>
              <button @click="completeRegisteredOnboarding" class="barrier-submit-btn" style="background-color: var(--oxfam-green);">
                Complete Registration
              </button>
            </div>
          </div>
        </div>

        <!-- ==========================================
             SCREEN: WORKSPACE TABS
             ========================================== -->
        <div v-else style="display:flex; flex-direction:column; gap:1rem; width:100%;">
          
          <!-- TAB 1: LEARN (SRHR Articles) -->
          <div v-if="activeTab === 'home'" style="display:flex; flex-direction:column; gap:1rem;">
            
            <!-- Welcome Gradient Card -->
            <div class="pwa-card pwa-card-gradient">
              <span style="font-size:0.55rem; font-weight:800; background-color: rgba(255,255,255,0.15); padding: 2px 6px; border-radius:10px;">
                AUDIO PLAYBACK ENABLED
              </span>
              <h3 style="font-size: 1rem; margin-top: 0.35rem;">Let's learn together without stigma.</h3>
              <p style="font-size: 0.6rem; opacity:0.85; margin-top: 2px;">Logged in pseudonymously: @{{ currentUser }}</p>
            </div>

            <!-- Articles Deck -->
            <!-- Article 1 -->
            <div class="pwa-card">
              <div style="display:flex; justify-content:space-between; align-items:flex-start; margin-bottom:0.25rem;">
                <span style="font-size: 0.6rem; font-weight:800; color: var(--oxfam-green);" v-if="!discreetMode">
                  CONTRACEPTION HEALTH
                </span>
                <span style="font-size: 0.6rem; font-weight:800; color: var(--oxfam-green);" v-else>
                  CELLULAR SCIENCE
                </span>
                <span style="font-size:0.55rem; color: var(--charcoal-600); background-color: var(--charcoal-100); padding: 1px 4px; border-radius:3px;">
                  3 min
                </span>
              </div>
              
              <h4 style="font-size: 0.8rem; margin-bottom: 0.25rem;">
                {{ discreetMode ? 'Human Reproduction & Mitosis Studies' : 'Understanding Modern Contraception Options' }}
              </h4>
              <p style="font-size: 0.65rem; color: var(--charcoal-600); line-height: 1.4;">
                {{ discreetMode 
                  ? 'A deep look at cell divisions, mitosis phases, chromosome replication patterns, and foundational secondary biology concepts.'
                  : 'Get accurate facts about long-acting implants, pills, and barrier protection methods available safely and confidentially at local units.' 
                }}
              </p>

              <div class="audio-action-row">
                <button 
                  @click="speakArticle(discreetMode ? 'Cell division guide. Mitosis is a cellular process that duplicates chromosomes.' : 'Understanding Modern Contraception. Implants and pills are distributed for free at youth-friendly health units in Lilongwe.')"
                  class="play-audio-btn"
                >
                  <svg style="width:12px; height:12px;" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M19.114 5.636a9 9 0 010 12.728M16.463 8.288a5.25 5.25 0 010 7.424M6.75 8.25l4.72-4.72a.75.75 0 011.28.53v15.88a.75.75 0 01-1.28.53l-4.72-4.72H4.51c-.88 0-1.704-.507-1.938-1.354A9.01 9.01 0 012.25 12c0-.83.112-1.633.322-2.396C2.806 8.756 3.63 8.25 4.51 8.25H6.75z"/>
                  </svg>
                  <span>Mvetserani (Listen Audio)</span>
                </button>
                <span style="font-size: 0.55rem; color: var(--charcoal-600);">Yao & Chichewa Supported</span>
              </div>
            </div>

            <!-- Article 2 -->
            <div class="pwa-card">
              <div style="display:flex; justify-content:space-between; align-items:flex-start; margin-bottom:0.25rem;">
                <span style="font-size: 0.6rem; font-weight:800; color: var(--oxfam-green);" v-if="!discreetMode">
                  BODY RIGHTS & RELATIONSHIPS
                </span>
                <span style="font-size: 0.6rem; font-weight:800; color: var(--oxfam-green);" v-else>
                  SOCIAL ECOLOGY
                </span>
                <span style="font-size:0.55rem; color: var(--charcoal-600); background-color: var(--charcoal-100); padding: 1px 4px; border-radius:3px;">
                  2 min
                </span>
              </div>
              
              <h4 style="font-size: 0.8rem; margin-bottom: 0.25rem;">
                {{ discreetMode ? 'Demographics & Citizen Social Ethics' : 'Healthy Consent, Boundaries, and Peer Choices' }}
              </h4>
              <p style="font-size: 0.65rem; color: var(--charcoal-600); line-height: 1.4;">
                {{ discreetMode 
                  ? 'A structural study mapping community systems, regional boundaries, local trading indicators, and secondary civic studies.'
                  : 'How to communicate comfort zones, handle social pressure safely, and secure confidential advice about rights and protection.' 
                }}
              </p>

              <div class="audio-action-row">
                <button 
                  @click="speakArticle(discreetMode ? 'Social ecology module. Learning about community structures and district boundaries.' : 'Consent and bodies. Every youth has the right to declare boundaries and make confidential choices.')"
                  class="play-audio-btn"
                >
                  <svg style="width:12px; height:12px;" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M19.114 5.636a9 9 0 010 12.728M16.463 8.288a5.25 5.25 0 010 7.424M6.75 8.25l4.72-4.72a.75.75 0 011.28.53v15.88a.75.75 0 01-1.28.53l-4.72-4.72H4.51c-.88 0-1.704-.507-1.938-1.354A9.01 9.01 0 012.25 12c0-.83.112-1.633.322-2.396C2.806 8.756 3.63 8.25 4.51 8.25H6.75z"/>
                  </svg>
                  <span>Mvetserani (Listen Audio)</span>
                </button>
                <span style="font-size: 0.55rem; color: var(--charcoal-600);">Yao & Chichewa Supported</span>
              </div>
            </div>

          </div>

          <!-- TAB 2: MAP (Services Near Me) -->
          <div v-else-if="activeTab === 'map'" style="display:flex; flex-direction:column; gap:0.75rem;">
            
            <div class="map-district-selects">
              <div>
                <label style="font-size: 0.55rem; font-weight:800; color: var(--charcoal-600); display:block; margin-bottom: 2px;">DISTRICT</label>
                <select v-model="selectedDistrict" class="map-control-select">
                  <option value="balaka">Balaka</option>
                  <option value="lilongwe">Lilongwe</option>
                </select>
              </div>
              <div>
                <label style="font-size: 0.55rem; font-weight:800; color: var(--charcoal-600); display:block; margin-bottom: 2px;">TRADITIONAL AUTHORITY</label>
                <select v-model="selectedTa" class="map-control-select">
                  <option value="kalembo">T.A. Kalembo</option>
                  <option value="msamala">T.A. Msamala</option>
                  <option value="amidu">T.A. Amidu</option>
                </select>
              </div>
            </div>

            <!-- Vector Map Frame (SVG boundary rendering) -->
            <div class="vector-map-frame">
              <svg style="width:100%; height:100%;" viewBox="0 0 100 100" fill="none">
                <!-- Grid dots -->
                <defs>
                  <pattern id="grid" width="10" height="10" patternUnits="userSpaceOnUse">
                    <circle cx="2" cy="2" r="1" fill="#cbd5e1" opacity="0.5"/>
                  </pattern>
                </defs>
                <rect width="100%" height="100%" fill="url(#grid)" />
                
                <!-- Dynamic SVG district polygon -->
                <path 
                  v-if="selectedDistrict === 'balaka'"
                  d="M 20 25 Q 40 10 75 20 T 80 80 T 35 85 Z" 
                  fill="hsl(120, 40%, 93%)" 
                  stroke="hsl(120, 45%, 70%)" 
                  stroke-width="1.5" 
                  stroke-dasharray="2,2"
                />
                <path 
                  v-else
                  d="M 15 45 Q 45 15 80 30 T 70 85 T 25 70 Z" 
                  fill="hsl(120, 40%, 93%)" 
                  stroke="hsl(120, 45%, 70%)" 
                  stroke-width="1.5" 
                  stroke-dasharray="2,2"
                />
              </svg>

              <!-- Map Markers pins placed -->
              <button 
                v-for="pin in activePins" 
                :key="pin.id"
                @click="selectedFacility = pin; emit('pwa-click')"
                class="map-marker-pin"
                :style="{ left: pin.x + '%', top: pin.y + '%' }"
              >
                <span class="map-pin-pulse"></span>
                <span class="map-pin-core"></span>
              </button>

              <div style="position:absolute; top: 6px; right: 6px; background-color: rgba(15,23,42,0.85); color:#fff; font-size:0.5rem; font-weight:800; padding:2px 6px; border-radius:3px; text-transform:uppercase; letter-spacing:0.5px;">
                OFFLINE MAP ENABLED
              </div>
            </div>

            <!-- Clicked Facility Card HUD -->
            <div class="pwa-card" style="border-color: var(--charcoal-200);">
              <div v-if="selectedFacility === null" style="text-align:center; padding: 0.5rem 0;">
                <p style="font-size:0.65rem; color:var(--charcoal-600);">
                  Tap a clinic marker on the pre-compiled vector map to view product supply volumes and support options.
                </p>
              </div>
              <div v-else class="facility-card-info">
                <div style="display:flex; justify-content:space-between; align-items:flex-start; margin-bottom: 0.35rem;">
                  <div>
                    <h4>{{ selectedFacility.name }}</h4>
                    <p style="font-size:0.55rem; color: var(--charcoal-600);">TA {{ selectedFacility.ta.toUpperCase() }} • Mapped Center</p>
                  </div>
                  <span class="facility-tag">{{ selectedFacility.type }}</span>
                </div>

                <div class="stock-grid">
                  <div class="stock-box">
                    <strong>Implants</strong>
                    <span :class="selectedFacility.implants === 'IN STOCK' ? 'stock-ok' : 'stock-none'">
                      {{ selectedFacility.implants }}
                    </span>
                  </div>
                  <div class="stock-box">
                    <strong>Oral Pills</strong>
                    <span :class="selectedFacility.pills === 'IN STOCK' ? 'stock-ok' : 'stock-none'">
                      {{ selectedFacility.pills }}
                    </span>
                  </div>
                  <div class="stock-box">
                    <strong>Condoms</strong>
                    <span :class="selectedFacility.condoms === 'IN STOCK' ? 'stock-ok' : 'stock-none'">
                      {{ selectedFacility.condoms }}
                    </span>
                  </div>
                </div>

                <div style="display:flex; justify-content:space-between; align-items:center; font-size: 0.55rem; color:var(--charcoal-600); border-t: 1px solid var(--charcoal-100); padding-top:0.4rem; margin-top:0.4rem;">
                  <span>Distance: ~2.8km from you</span>
                  <span style="font-weight:700; color:var(--oxfam-green);">✓ Zero-Rated Hotline Active</span>
                </div>
              </div>
            </div>

          </div>

          <!-- TAB 3: CHAT (Moderated Forums) -->
          <div v-else-if="activeTab === 'chat'" style="display:flex; flex-direction:column; gap:0.5rem;">
            
            <div style="display:flex; justify-content:space-between; align-items:center;">
              <div>
                <h3 style="font-size: 0.75rem; text-transform:uppercase; color: var(--charcoal-600);">Moderated Chat Forums</h3>
                <p style="font-size:0.55rem; color: var(--oxfam-green-dark); font-weight:700;">Community Lead: Oxfam Moderator</p>
              </div>
              <span style="font-size:0.55rem; background-color: var(--charcoal-200); padding:1px 5px; border-radius:4px; font-weight:700;">
                PSEUDONYM ACTIVE
              </span>
            </div>

            <!-- Chat Threads View -->
            <div class="forum-thread">
              <div 
                v-for="post in forumPosts" 
                :key="post.id" 
                class="forum-bubble" 
                :class="{ 'forum-bubble-pending': post.status === 'pending' }"
              >
                <div class="forum-bubble-header">
                  <span>@{{ post.username }}</span>
                  <span>{{ post.timestamp }}</span>
                </div>
                <p>{{ post.message }}</p>
                <div class="forum-bubble-footer">
                  <span style="background-color: var(--charcoal-200); padding:0 3px; border-radius:2px; font-weight:700;">
                    {{ post.district }}
                  </span>
                  
                  <span v-if="post.status === 'pending'" class="sync-status wait">
                    <svg style="width:8px; height:8px;" fill="none" stroke="currentColor" stroke-width="3" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" d="M12 6v6h4.5m4.5 0a9 9 0 11-18 0 9 9 0 0118 0z"/>
                    </svg>
                    IndexedDB Outbox Queue
                  </span>
                  <span v-else class="sync-status ok">
                    <svg style="width:8px; height:8px;" fill="none" stroke="currentColor" stroke-width="3" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7"/>
                    </svg>
                    Synced to Metabase
                  </span>
                </div>
              </div>
            </div>

            <!-- Message Input console -->
            <div style="display:flex; flex-direction:column; gap:2px;">
              <div class="chat-input-row">
                <input 
                  v-model="chatInput" 
                  type="text" 
                  placeholder="Ask or share anonymously..." 
                  class="chat-field"
                  @keyup.enter="postMessage"
                />
                <button @click="postMessage" class="chat-send-btn">
                  <svg style="width:14px; height:14px;" fill="none" stroke="currentColor" stroke-width="2.5" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M6 12L3.269 3.126A59.768 59.768 0 0121.485 12 59.77 59.77 0 013.27 20.876L5.999 12zm0 0h7.5"/>
                  </svg>
                </button>
              </div>
              <span style="font-size:0.55rem; color:var(--charcoal-600);">
                Posts are securely filtered asynchronous to support safe environments.
              </span>
            </div>

          </div>

          <!-- TAB 4: REPORT (Access Barrier Submission) -->
          <div v-else-if="activeTab === 'report'" style="display:flex; flex-direction:column; gap:0.75rem;">
            
            <div>
              <h3 style="font-size:0.75rem; text-transform:uppercase; color: var(--charcoal-600);">Access Barrier Reporter</h3>
              <p style="font-size:0.6rem; color: var(--charcoal-600); margin-top:2px;">
                Encountered clinic issues? Anonymously signal product stock-outs, bribe demands, or stigma.
              </p>
            </div>

            <div class="pwa-card" style="display:flex; flex-direction:column; gap: 0.75rem;">
              <!-- Issue Category Grid -->
              <div>
                <label style="font-size:0.6rem; font-weight:800; color:var(--charcoal-600); display:block; margin-bottom: 4px;">
                  1. SELECT ENCOUNTERED ISSUE
                </label>
                <div class="barrier-grid">
                  <button 
                    @click="selectBarrier('Product Stock-out')" 
                    class="barrier-grid-btn"
                    :class="{ active: barrierIssue === 'Product Stock-out' }"
                  >
                    📦 Stock-out Alert
                  </button>
                  <button 
                    @click="selectBarrier('Service Denial')" 
                    class="barrier-grid-btn"
                    :class="{ active: barrierIssue === 'Service Denial' }"
                  >
                    ❌ Service Refused
                  </button>
                  <button 
                    @click="selectBarrier('Stigma/Harassment')" 
                    class="barrier-grid-btn"
                    :class="{ active: barrierIssue === 'Stigma/Harassment' }"
                  >
                    🗣️ Staff Stigma
                  </button>
                  <button 
                    @click="selectBarrier('Bribe Demand')" 
                    class="barrier-grid-btn"
                    :class="{ active: barrierIssue === 'Bribe Demand' }"
                  >
                    💰 Bribe Requested
                  </button>
                </div>
              </div>

              <!-- Location Landmarking -->
              <div>
                <label style="font-size:0.6rem; font-weight:800; color:var(--charcoal-600); display:block; margin-bottom: 2px;">
                  2. SPECIFY GEOGRAPHIC LOCATION
                </label>
                <div style="display:flex; flex-direction:column; gap:4px;">
                  <select v-model="barrierDistrict" class="map-control-select">
                    <option value="">Select District</option>
                    <option value="Balaka">Balaka</option>
                    <option value="Lilongwe">Lilongwe</option>
                  </select>
                  <input 
                    v-model="barrierLandmark" 
                    type="text" 
                    placeholder="Clinic name / Nearest trading post TA landmark" 
                    class="chat-field"
                  />
                </div>
              </div>

              <!-- Optional Details -->
              <div>
                <label style="font-size:0.6rem; font-weight:800; color:var(--charcoal-600); display:block; margin-bottom: 2px;">
                  3. ANONYMOUS DETAILS (OPTIONAL)
                </label>
                <textarea 
                  v-model="barrierDetails" 
                  rows="2" 
                  placeholder="Share details confidentially. Identifiers are never sent." 
                  class="chat-field"
                  style="resize:none;"
                ></textarea>
              </div>

              <button @click="postBarrierReport" class="barrier-submit-btn">
                <svg style="width:12px; height:12px;" fill="none" stroke="currentColor" stroke-width="2.5" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z"/>
                </svg>
                <span>Submit Secure Encrypted Report</span>
              </button>

            </div>

          </div>

        </div>

      </div>

      <!-- Quick Exit cover search mask overlay -->
      <div v-else class="quick-exit-overlay">
        <div style="text-align:left; width:100%;">
          <strong style="color: hsl(217, 89%, 61%); font-size:1.4rem;">Google</strong>
          <div class="fake-search-box">
            <svg style="width:14px; height:14px;" fill="none" stroke="currentColor" stroke-width="2.5" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"/>
            </svg>
            <span>Malawi national education curriculum secondary modules</span>
          </div>

          <div class="fake-results-box">
            <h4 style="font-size:0.75rem; margin-bottom:0.25rem;">Biology & Social Sciences Syllabus Recommendations</h4>
            <p style="font-size:0.6rem; color:var(--charcoal-600);">Guides for MSCE national exams covering plant photosynthesis models, chemical atomic bonds, and historic regional demographic boundaries.</p>
            
            <ul class="fake-links">
              <li>• secondary-chemistry-syllabus-2025.pdf</li>
              <li>• historical-malawi-traditional-authorities.docx</li>
            </ul>
          </div>
          
          <button @click="quickExitActive = false" class="restore-sandbox-btn">
            Resume Testing Sandbox
          </button>
        </div>
      </div>

      <!-- PWA Navigation bottom bar -->
      <nav class="pwa-bottom-nav" v-if="onboardingTier !== null && !quickExitActive">
        <button 
          @click="handleNavClick('home')" 
          class="pwa-nav-item"
          :class="{ active: activeTab === 'home' }"
        >
          <svg fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" d="M12 6.253v13m0-13C10.832 5.477 9.246 5 7.5 5S4.168 5.477 3 6.253v13C4.168 18.477 5.754 18 7.5 18s3.332.477 4.5 1.253m0-13C13.168 5.477 14.754 5 16.5 5c1.747 0 3.332.477 4.5 1.253v13C19.832 18.477 18.247 18 16.5 18c-1.746 0-3.332.477-4.5 1.253"/>
          </svg>
          <span>Learn</span>
        </button>

        <button 
          @click="handleNavClick('map')" 
          class="pwa-nav-item"
          :class="{ active: activeTab === 'map' }"
        >
          <svg fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" d="M9 6.75V15m6-6v8m-9-3.5a3.375 3.375 0 110-6.75M21 12a9 9 0 11-18 0 9 9 0 0118 0z"/>
          </svg>
          <span>Map</span>
        </button>

        <button 
          @click="handleNavClick('chat')" 
          class="pwa-nav-item"
          :class="{ active: activeTab === 'chat' }"
        >
          <svg fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" d="M8.625 9.75a.625.625 0 11-1.25 0 .625.625 0 011.25 0zm4.5 0a.625.625 0 11-1.25 0 .625.625 0 011.25 0zm4.5 0a.625.625 0 11-1.25 0 .625.625 0 011.25 0zM21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z"/>
          </svg>
          <span>Forums</span>
        </button>

        <button 
          @click="handleNavClick('report')" 
          class="pwa-nav-item"
          :class="{ active: activeTab === 'report' }"
        >
          <svg fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z"/>
          </svg>
          <span>Report</span>
        </button>
      </nav>

      <!-- Audio playback visualizer toast -->
      <div v-if="audioToastActive" class="audio-toast">
        <div style="display:flex; align-items:center; gap:0.5rem;">
          <button @click="stopAudio" style="background-color: var(--oxfam-green); color:#fff; border:none; width:20px; height:20px; border-radius:50%; display:flex; justify-content:center; align-items:center; cursor:pointer;">
            ■
          </button>
          <div>
            <p style="font-size:0.55rem; color:var(--charcoal-600); text-transform:uppercase; font-weight:800; letter-spacing:0.5px;">AUDIO READING</p>
            <p style="font-size:0.6rem; font-weight:700; width:160px; overflow:hidden; text-overflow:ellipsis; white-space:nowrap;">{{ activeAudioText }}</p>
          </div>
        </div>
        <!-- Simulated animated wave bar graphs -->
        <div class="wave-bars-box">
          <span class="wave-bar" style="animation-delay: 0.1s"></span>
          <span class="wave-bar" style="animation-delay: 0.3s"></span>
          <span class="wave-bar" style="animation-delay: 0.5s"></span>
          <span class="wave-bar" style="animation-delay: 0.2s"></span>
        </div>
      </div>

    </div>
  </div>

  <!-- Global Modal/Toast Popup simulation -->
  <div v-if="showModal" class="modal-backdrop" @click.self="showModal = false">
    <div class="modal-content">
      <div class="modal-header" :class="modalType">
        <svg v-if="modalType === 'success'" style="width:20px; height:20px;" fill="none" stroke="currentColor" stroke-width="2.5" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"/>
        </svg>
        <svg v-else-if="modalType === 'error'" style="width:20px; height:20px;" fill="none" stroke="currentColor" stroke-width="2.5" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z"/>
        </svg>
        <svg v-else style="width:20px; height:20px;" fill="none" stroke="currentColor" stroke-width="2.5" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" d="M11.25 11.25l.041-.02a.75.75 0 111.063.852l-.708 2.836a.75.75 0 001.063.852l.041-.021M21 12a9 9 0 11-18 0 9 9 0 0118 0zm-9-3.75h.008v.008H12V8.25z"/>
        </svg>
        <span>{{ modalTitle }}</span>
      </div>
      <div class="modal-body">{{ modalMsg }}</div>
      <button @click="showModal = false" class="modal-close-btn">Confirm & Continue</button>
    </div>
  </div>
</template>
