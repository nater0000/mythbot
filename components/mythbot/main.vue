<template>
  <div class="flex flex-col md:flex-row min-h-screen bg-main text-main font-lora">
    <!-- Sidebar Navigation -->
    <nav class="w-full md:w-80 bg-black bg-opacity-20 border-r border-black border-opacity-20 p-4 md:p-6 flex-shrink-0">
      <h1 class="text-3xl font-bold text-white mb-8 font-cinzel">METAMYTH</h1>
      <div class="space-y-1">
        <template v-for="(group, artifact) in groupedStages" :key="artifact">
          <h3 class="nav-artifact-header text-sm font-bold uppercase mt-6 mb-2 px-4">{{ artifact }}</h3>
          <a
            v-for="stage in group"
            :key="stage.id"
            href="#"
            class="nav-link flex items-center rounded-md"
            :class="{ 'active': activeStage && activeStage.id === stage.id }"
            @click.prevent="showStage(stage)"
          >
            <span class="mr-3" :class="stage.iconClass || ''">{{ stage.icon }}</span>
            {{ stage.title }}
          </a>
        </template>
      </div>
    </nav>

    <!-- Main Content -->
    <main class="flex-1 p-4 md:p-10 overflow-y-auto">
      <!-- CHANGE: Added a template ref to this wrapper div -->
      <div ref="contentTop" v-if="activeStage" class="stage-content active">
        <div class="content-card">

          <!-- Special template for the Intro page to handle the button click -->
          <template v-if="activeStage.id === 'intro'">
            <div class="text-center">
                <h2 class="text-5xl font-bold mb-4 font-cinzel">Awaken.</h2>
                <p class="text-xl text-slate-400 mb-6">Adventure is calling. Do you accept?</p>
            </div>
            <div class="prose max-w-none mx-auto mt-8">
                <p>Our crisis is a completely broken story. A species destroying its only home for numbers on a screen; an economic system that grows by devouring what sustains it; a world that has gone amuck because we’ve forgotten it's a story in the first place. We've taken it so seriously that we’ve stopped writing it.</p>
                <p>Our purpose has always been to create the world through story—to create ourselves, pass on our connection and history, and have visions of the future that we communicate to others and make real in the telling. We have forgotten this purpose and have blindly consented to be characters in someone else's story, one that none of us truly wrote.</p>
                <p>But here's the cosmic joke that'll set you free: <strong>We wrote this nightmare, so we can rewrite it.</strong></p>
            </div>
            <div class="text-center mt-10">
                <button class="btn-primary" @click="showNextStage">Begin the Journey</button>
            </div>
          </template>

          <!-- Generic Stage with Prompts -->
          <template v-else-if="prompts[activeStage.id]">
            <div class="flex items-center mb-4">
              <span class="artifiction-icon" :class="activeStage.iconClass || ''">{{ activeStage.icon }}</span>
              <h2 class="text-4xl font-bold font-cinzel">{{ activeStage.title }}</h2>
            </div>
            <div class="prose max-w-none mb-8">
              <p>{{ descriptions[activeStage.id] }}</p>
            </div>
            <div class="ornamental-divider"></div>
            <div class="space-y-8">
              <div v-for="prompt in prompts[activeStage.id]" :key="prompt.key">
                <label class="font-semibold text-lg text-white block mb-2">{{ prompt.label }}</label>
                <textarea v-model="metamythData[activeStage.id][prompt.key]" :rows="prompt.rows || 3"></textarea>
              </div>
            </div>
          </template>

          <!-- Synthesis Stages -->
          <template v-else-if="activeStage.id.endsWith('_synthesis')">
              <div class="flex items-center mb-4">
                  <span class="artifiction-icon">{{ activeStage.icon }}</span>
                  <h2 class="text-4xl font-bold font-cinzel">{{ activeStage.title }}</h2>
              </div>
              <div class="prose max-w-none mb-6">
                  <p>{{ synthesisPrompts[activeStage.id] }}</p>
              </div>
              <div class="synthesis-box mb-6">
                  {{ synthesisStatements[activeStage.id.replace('_synthesis', '')] }}
              </div>
              <div>
                  <label class="font-semibold text-lg text-white block mb-2">Refine Your Statement:</label>
                  <textarea v-model="metamythData[activeStage.id].refined" rows="4" placeholder="Refine the suggested statement above or write your own..."></textarea>
              </div>
          </template>

          <!-- Final Synthesis Stage -->
          <template v-else-if="activeStage.id === 'synthesis'">
              <div class="flex items-center mb-4"><span class="artifiction-icon">📜</span><h2 class="text-4xl font-bold font-cinzel">Your Complete Metamyth</h2></div>
              <div class="prose max-w-none mb-8">
                  <p>This is your story, synthesized from your journey. It is the core narrative DNA you have authored. Copy this text and use it as the custom instruction for your AI Wizard, your wise guide for the adventures to come.</p>
              </div>
              <div ref="synthesisOutputRef" class="bg-black bg-opacity-20 p-6 rounded-lg border border-slate-700 space-y-4 text-slate-200 leading-relaxed">
                  <div v-html="finalSynthesisHtml"></div>
              </div>
              <button @click="copySynthesis" class="btn-primary mt-6">Copy to Clipboard</button>
              <span v-if="copied" class="ml-4 transition-opacity" style="color: var(--accent-turqoise);">Copied!</span>
          </template>

          <!-- Compass Stage -->
          <template v-else-if="activeStage.id === 'compass'">
            <div class="flex items-center mb-4"><span class="artifiction-icon">🧭</span><h2 class="text-4xl font-bold font-cinzel">The Compass</h2></div>
            <div class="prose max-w-none mb-8">
                <p>When external maps become useless, you navigate by eternal coordinates of your values. The Compass organizes all activities across eight dimensions of health. Your values become an accountability framework that maintains alignment, creating health bars in the dashboard of reality. Rate your current alignment (1-10) for each point.</p>
            </div>
            <div class="ornamental-divider"></div>
            <div class="flex flex-wrap -mx-4">
                <div class="w-full lg:w-1/2 px-4">
                    <div v-for="(point, index) in compassPoints" :key="point.key" class="mb-4">
                         <div class="flex justify-between items-center mb-1">
                            <label class="font-semibold text-white text-sm">{{ index + 1 }}. {{ point.label }}</label>
                            <span class="font-bold" style="color: var(--accent-turqoise);">{{ metamythData.compass[point.key] }}</span>
                        </div>
                        <input type="range" v-model.number="metamythData.compass[point.key]" class="w-full" min="1" max="10">
                    </div>
                </div>
                <div class="w-full lg:w-1/2 px-4 flex items-center justify-center">
                    <div class="chart-container">
                        <canvas ref="compassCanvas"></canvas>
                    </div>
                </div>
            </div>
          </template>
          
          <!-- Other Static Stages -->
          <template v-else>
             <div v-html="staticContent[activeStage.id] || ''"></div>
          </template>

        </div>
        
        <!-- Navigation Buttons -->
        <div v-if="showNavigationButtons" class="mt-8 flex justify-between">
            <button class="btn-secondary" @click="showPreviousStage">Previous</button>
            <button class="btn-primary" @click="showNextStage">Next</button>
        </div>
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, onMounted, computed, watch, nextTick } from 'vue';
import Chart from 'chart.js/auto';

// --- FONT & METADATA SETUP ---
useHead({
  title: 'METAMYTH: A Journey of Transformation',
  link: [
    { rel: 'preconnect', href: 'https://fonts.googleapis.com' },
    { rel: 'preconnect', href: 'https://fonts.gstatic.com', crossorigin: '' },
    { href: 'https://fonts.googleapis.com/css2?family=Cinzel:wght@400;700&family=Lora:ital,wght@0,400;0,700;1,400&display=swap', rel: 'stylesheet' }
  ],
  bodyAttrs: {
    class: 'antialiased'
  }
});


// --- DATA & STATE ---
const contentTop = ref(null); // CHANGE: Ref for the top of the content area
const compassCanvas = ref(null);
const synthesisOutputRef = ref(null);
const copied = ref(false);
let compassChart = null;

const stages = ref([
    { id: 'intro', title: 'Awaken', artifact: 'Preamble', icon: '✨' },
    { id: 'howto', title: 'How to Rewrite Your Life', artifact: 'Preamble', icon: '📖' },
    { id: 'origin', title: 'The Origin Story', artifact: 'Preamble', icon: '🌱' },
    { id: 'artifact1', title: 'The Call to Adventure', artifact: 'Artifact I', icon: '🔔' },
    { id: 'dragon', title: 'The Dragon', artifact: 'Artifact I', icon: '🐉', iconClass: 'crimson-text' },
    { id: 'meaning', title: 'The Meaning', artifact: 'Artifact I', icon: '💫' },
    { id: 'shield', title: 'The Shield', artifact: 'Artifact I', icon: '🛡️' },
    { id: 'voice', title: 'The Voice', artifact: 'Artifact I', icon: '💎' },
    { id: 'calling_synthesis', title: 'The Calling', artifact: 'Artifact I', icon: '📣' },
    { id: 'artifact2', title: 'The Quest', artifact: 'Artifact II', icon: '🗺️' },
    { id: 'star', title: 'The Star', artifact: 'Artifact II', icon: '⭐' },
    { id: 'character', title: 'The Character', artifact: 'Artifact II', icon: '👤' },
    { id: 'banner', title: 'The Banner', artifact: 'Artifact II', icon: '🚩' },
    { id: 'sword', title: 'The Sword', artifact: 'Artifact II', icon: '⚔️', iconClass: 'crimson-text' },
    { id: 'quest_synthesis', title: 'The Quest', artifact: 'Artifact II', icon: '📜' },
    { id: 'artifact3', title: 'What is Possible', artifact: 'Artifact III', icon: '🌌' },
    { id: 'looking_glass', title: 'The Looking Glass', artifact: 'Artifact III', icon: '🔮' },
    { id: 'globe', title: 'The Globe', artifact: 'Artifact III', icon: '🌍' },
    { id: 'map', title: 'The Map', artifact: 'Artifact III', icon: '🗺️' },
    { id: 'transformation', title: 'The Transformation', artifact: 'Artifact III', icon: '🦋' },
    { id: 'vision_synthesis', title: 'The Vision', artifact: 'Artifact III', icon: '🔭' },
    { id: 'artifact4', title: 'The Journey', artifact: 'Artifact IV', icon: '👣' },
    { id: 'fountain', title: 'The Fountain', artifact: 'Artifact IV', icon: '⛲' },
    { id: 'ethos', title: 'The Ethos', artifact: 'Artifact IV', icon: '📜' },
    { id: 'plot', title: 'The Plot', artifact: 'Artifact IV', icon: '📊' },
    { id: 'compass', title: 'The Compass', artifact: 'Artifact IV', icon: '🧭' },
    { id: 'mission_synthesis', title: 'The Mission', artifact: 'Artifact IV', icon: '🎯' },
    { id: 'artifact5', title: 'The Return', artifact: 'Artifact V', icon: '🏡' },
    { id: 'grail', title: 'The Grail', artifact: 'Artifact V', icon: '🏆' },
    { id: 'initiation', title: 'The Initiation', artifact: 'Artifact V', icon: '🌉' },
    { id: 'campfire', title: 'The Campfire', artifact: 'Artifact V', icon: '🔥', iconClass: 'crimson-text' },
    { id: 'messages', title: 'The Messages', artifact: 'Artifact V', icon: '📢' },
    { id: 'kindred_synthesis', title: 'Kindred Quest', artifact: 'Artifact V', icon: '🤝' },
    { id: 'wizard', title: 'The Wizard', artifact: 'Legacy', icon: '🧙' },
    { id: 'activate_wizard', title: 'Activate Your AI', artifact: 'Legacy', icon: '🤖' },
    { id: 'synthesis', title: 'Your Metamyth', artifact: 'Legacy', icon: '📜' },
    { id: 'epilogue', title: 'The Great Adventure', artifact: 'Legacy', icon: '🌅' },
]);

const metamythData = ref({});
const activeStage = ref(null);

// --- COMPUTED & DERIVED STATE ---

const activeStageIndex = computed(() => {
    if (!activeStage.value) return -1;
    return stages.value.findIndex(s => s.id === activeStage.value.id);
});

const showNavigationButtons = computed(() => {
    if (!activeStage.value) return false;
    const nonNavIds = ['intro', 'epilogue'];
    return !nonNavIds.includes(activeStage.value.id);
});

const groupedStages = computed(() => {
  return stages.value.reduce((acc, stage) => {
    (acc[stage.artifact] = acc[stage.artifact] || []).push(stage);
    return acc;
  }, {});
});

const synthesisStatements = computed(() => {
    const get = (stage, field, fallback = '') => (metamythData.value[stage]?.[field]) || fallback;
    
    const calling = get('calling_synthesis', 'refined') || `The world calls me to stand between [${get('dragon', 'madness', 'the broken story')}] and [${get('shield', 'needs', 'those needing protection')}] because my [${get('shield', 'symbol', 'survival')}] speaks through [${get('voice', 'tone', 'my authentic voice')}] to transform [${get('meaning', 'framework', 'suffering')}] into collective wisdom.`;
    const quest = get('quest_synthesis', 'refined') || `I exist to [${get('star', 'purpose', 'serve a higher purpose')}] by embodying the character of [${get('character', 'we', 'a world-changer')}] who plants a banner for [${get('banner', 'direction', 'a new direction')}] and wields the sword of [${get('sword', 'action', 'truthful action')}].`;
    const vision = get('vision_synthesis', 'refined') || `When I follow my purpose all the way, I see [${get('looking_glass', 'impossible', 'a transformed world')}], building an organization that becomes [${get('globe', '100', 'a regenerative force')}] by navigating the horizons of [${get('map', 'h3', 'global impact')}] until I transform into [${get('transformation', 'become', 'my highest self')}].`;
    const mission = get('mission_synthesis', 'refined') || `My mission is to circulate [${get('fountain', 'wealth', 'abundant resources')}] by implementing agreements based on [${get('ethos', 'principles', 'our core principles')}] through trials like [${get('plot', 'trials', 'overcoming key challenges')}] while navigating by my values.`;
    const kindred = get('kindred_synthesis', 'refined') || `I call forward those ready for [${get('grail', 'gift', 'a deep transformation')}] to cross the bridge of [${get('initiation', 'pathway', 'our shared journey')}] and join our campfire for [${get('campfire', 'collective', 'a collective story')}] because together we will create a legacy.`;
    
    return { calling, quest, vision, mission, kindred };
});

const finalSynthesisHtml = computed(() => {
    const getArtifictionDetailsHTML = (stageIds) => {
        let html = '<div class="space-y-4 mt-4">';
        stageIds.forEach(id => {
            const stage = stages.value.find(s => s.id === id);
            if (!stage || !metamythData.value[id]) return;
            
            html += `<h4 class="text-xl font-bold font-cinzel mt-6">${stage.title}</h4>`;
            const stagePrompts = prompts[id] || [];

            stagePrompts.forEach(prompt => {
                const value = metamythData.value[id][prompt.key];
                if (value) {
                    html += `<p><strong>${prompt.label}</strong><br><span class="text-slate-300 italic">${value.replace(/\n/g, '<br>')}</span></p>`;
                }
            });
        });
        html += '</div>';
        return html;
    };

    const { calling, quest, vision, mission, kindred } = synthesisStatements.value;
    return `
        <h3 class="text-2xl font-cinzel">Preamble: The Origin Story</h3>
        ${getArtifictionDetailsHTML(['origin'])}
        <div class="ornamental-divider my-6"></div>
        
        <h3 class="text-2xl font-cinzel">Artifact I: The Calling</h3>
        <p class="synthesis-box my-4">${calling}</p>
        ${getArtifictionDetailsHTML(['dragon', 'meaning', 'shield', 'voice'])}
        <div class="ornamental-divider my-6"></div>

        <h3 class="text-2xl font-cinzel">Artifact II: The Quest</h3>
        <p class="synthesis-box my-4">${quest}</p>
        ${getArtifictionDetailsHTML(['star', 'character', 'banner', 'sword'])}
        <div class="ornamental-divider my-6"></div>

        <h3 class="text-2xl font-cinzel">Artifact III: The Vision</h3>
        <p class="synthesis-box my-4">${vision}</p>
        ${getArtifictionDetailsHTML(['looking_glass', 'globe', 'map', 'transformation'])}
        <div class="ornamental-divider my-6"></div>

        <h3 class="text-2xl font-cinzel">Artifact IV: The Mission</h3>
        <p class="synthesis-box my-4">${mission}</p>
        ${getArtifictionDetailsHTML(['fountain', 'ethos', 'plot'])}
        <div class="ornamental-divider my-6"></div>

        <h3 class="text-2xl font-cinzel">Artifact V: The Call to Kindred Quest</h3>
        <p class="synthesis-box my-4">${kindred}</p>
        ${getArtifictionDetailsHTML(['grail', 'initiation', 'campfire', 'messages'])}
    `;
});


// --- METHODS ---

const showStage = (stage) => {
  activeStage.value = stage;
  if (!metamythData.value[stage.id]) {
      metamythData.value[stage.id] = {};
  }
  // CHANGE: Use nextTick to ensure the element exists before trying to scroll
  nextTick(() => {
    if (contentTop.value) {
      contentTop.value.scrollIntoView({ behavior: 'smooth', block: 'start' });
    }
  });
};

const showNextStage = () => {
    const nextIndex = activeStageIndex.value + 1;
    if (nextIndex < stages.value.length) {
        showStage(stages.value[nextIndex]);
    }
};

const showPreviousStage = () => {
    const prevIndex = activeStageIndex.value - 1;
    if (prevIndex >= 0) {
        showStage(stages.value[prevIndex]);
    }
};

const copySynthesis = async () => {
    if (!synthesisOutputRef.value) return;
    try {
        await navigator.clipboard.writeText(synthesisOutputRef.value.innerText);
        copied.value = true;
        setTimeout(() => { copied.value = false; }, 2000);
    } catch (err) {
        console.error('Failed to copy text: ', err);
    }
};

// --- CHART LOGIC ---

const renderCompassChart = () => {
  if (!compassCanvas.value) return;

  const data = compassPoints.map(p => metamythData.value.compass[p.key] || 5);
  const labels = compassPoints.map(p => p.label.replace('-', '\n'));
  
  if (compassChart) {
    compassChart.destroy();
  }

  compassChart = new Chart(compassCanvas.value, {
    type: 'radar',
    data: {
      labels,
      datasets: [{
        label: 'Your Alignment',
        data,
        backgroundColor: 'rgba(95, 226, 223, 0.2)',
        borderColor: 'rgba(95, 226, 223, 1)',
        borderWidth: 2,
        pointBackgroundColor: 'rgba(95, 226, 223, 1)'
      }]
    },
    options: {
        maintainAspectRatio: false,
        scales: {
            r: {
                angleLines: { color: 'rgba(255, 255, 255, 0.2)' },
                grid: { color: 'rgba(255, 255, 255, 0.2)' },
                pointLabels: { font: { size: 12, family: 'Lora' }, color: '#E0E0E0' },
                ticks: {
                    color: '#A0A0A0',
                    backdropColor: 'rgba(24, 51, 50, 0.75)',
                    stepSize: 2
                },
                suggestedMin: 0,
                suggestedMax: 10
            }
        },
        plugins: {
            legend: {
                display: false
            }
        }
    }
  });
};

watch(() => metamythData.value.compass, () => {
    if (compassChart) {
        compassChart.data.datasets[0].data = compassPoints.map(p => metamythData.value.compass[p.key] || 0);
        compassChart.update();
    }
}, { deep: true });

watch(activeStage, (newStage) => {
    if (newStage?.id === 'compass') {
        nextTick(renderCompassChart);
    }
});


// --- LIFECYCLE HOOKS ---
onMounted(() => {
  metamythData.value.compass = {};
  compassPoints.forEach(p => {
      metamythData.value.compass[p.key] = 5;
  });
  
  if (stages.value.length > 0) {
    // Don't scroll on initial load
    activeStage.value = stages.value[0];
    if (!metamythData.value[activeStage.value.id]) {
      metamythData.value[activeStage.value.id] = {};
    }
  }
});

// --- STATIC CONTENT REFs ---

const compassPoints = [
    { label: 'Purpose', key: 'purpose' },
    { label: 'Vision', key: 'vision' },
    { label: 'Truth', key: 'truth' },
    { label: 'Creativity', key: 'creativity' },
    { label: 'Community', key: 'community' },
    { label: 'Power', key: 'power' },
    { label: 'Economy-Ecology', key: 'economyecology' },
    { label: 'System Integration', key: 'systemintegration' },
];

const descriptions = {
    origin: "Every hero's journey begins before the call to adventure—in the ordinary world that shaped them, the primordial essence from which their quest is born. Like a seed, your origin contains the DNA of the entire story that follows. This is your backstory, the prelude. It is the programming, the passions, and the pivotal experiences that forged your character long before the mission was ever clear.",
    dragon: "Look around with fresh eyes. The problems of the world are really all one big problem: a broken story. You see this madness because something woke you up. The dragon reflects the insanity back to you, and you can no longer pretend it's normal.",
    meaning: "Here lies the moment of alchemical transformation—where your biggest struggle becomes your qualification, your medicine. Meaning is what you make of what happens to you. Your breakdown becomes everyone's breakthrough potential.",
    shield: "Your breaking point becomes your expertise. The hardships you endured forged a strength that now protects you and others. Your trauma, alchemized, is now a shield you can use to defend what is sacred.",
    voice: "Your authentic candor and tone—how you carry all you've been through. This is authenticity as action, the delivery system that attracts the right people and repels the wrong ones.",
    star: "What is your purpose? Why are you here? This is your unwavering purpose, the one direction that never leads you astray. When you follow your Star, life unfolds with supernatural synchronicity.",
    character: "This is your people, the 'we' identity that unifies everyone drawn to your work, and the character you are here to play in the great story.",
    banner: "This is the flag you plant above the chaos. You're not competing; you're holding a new territory, an alternative direction for others to find and follow.",
    sword: "Your Sword is what you DO—the specific action that cuts through the BS and strikes at the heart of what needs to change. It's your authenticity in action.",
    looking_glass: "Here we project the dream outcomes, the results that become possible when you fully actualize your purpose. This makes the impossible feel inevitable.",
    globe: "This is actual worldbuilding. If your organization succeeded for 100 years, what would its culture, identity, and structure look like? Think of this as science fiction about your organization.",
    map: "The Map works backward from your dream outcome through different horizons of scale. This helps you plan your pathway to maximum impact.",
    transformation: "This is who you would have to become to realize your vision. As you transform, others transform. The character change in you becomes the catalyst for change in your community.",
    fountain: "The Fountain represents what you actually have in abundance—your networks, your gifts, your true wealth. It's the source you draw from when in need.",
    ethos: "The Ethos is your law code—the agreements that coordinate your group around a shared purpose, turning individual heroes into a collective force.",
    plot: "Your Plot breaks your mission into actionable trials that build character. This isn't just task management; it's character development through meaningful challenges.",
    grail: "You return with the Grail—your gift to the world. The transformation you went through becomes the offering that transforms others.",
    initiation: "This maps your journey into a pathway for others to follow, an initiation that turns strangers into allies in your story.",
    campfire: "The Campfire is how you turn your 'me' story into a collective 'we' story. It's the movement container that will burn long after you're gone.",
    messages: "Your Messages are a broadcast of where you are, what you need, and what you offer. It's how you connect with hearts in various states of readiness.",
};

const prompts = {
    origin: [
        { label: "What childhood memories, passions, or activities shaped your earliest view of the world? (Your core programming)", key: 'coreprogramming', rows: 4 },
        { label: "Who were your heroes, mentors, or the key people whose lives left a deep impression on you?", key: 'influences', rows: 4 },
        { label: "Describe an early, pivotal incident that planted a seed for the person you would become.", key: 'incident', rows: 4 },
        { label: "What was the core truth, love, or fascination that has always been a part of you?", key: 'essence', rows: 4 },
    ],
    dragon: [ { label: "What civilizational madness can you no longer ignore?", key: "madness", rows: 4 }, { label: "What is the dark future ahead if we don't stop this dragon?", key: "future", rows: 4 } ],
    meaning: [ { label: "How does your personal struggle reflect collective patterns?", key: "patterns" }, { label: "What sense-making framework emerges from your wound?", key: "framework" }, { label: "How does your understanding help others make meaning?", key: "help" }, { label: "What 'why this matters' explanation comes from your journey?", key: "why" } ],
    shield: [ { label: "What have you survived that qualifies you to protect others?", key: "survived" }, { label: "Who needs the protection you can offer?", key: "needs" }, { label: "How does your survival become their shield?", key: "how" }, { label: "What symbol represents your hard-earned qualification?", key: "symbol", rows: 2 } ],
    voice: [ { label: "What authentic tone and candor do you carry?", key: "tone" }, { label: "How does your voice call in the right people and repel the wrong ones?", key: "callsin" }, { label: "How does your authenticity become action against injustice?", key: "action" }, { label: "What gives you permission to speak this way?", key: "permission", rows: 2 } ],
    star: [ { label: "What is your unwavering North Star purpose?", key: "purpose" }, { label: "Why does this matter beyond your individual life?", key: "matters" }, { label: "How does following this serve planetary transformation?", key: "serves" }, { label: "What does this look like in practice?", key: "practice", rows: 2 } ],
    character: [ { label: "Who are the people/characters in your organization?", key: "people" }, { label: "What deeper 'we' identity unifies you?", key: "we" }, { label: "What character trait do you all share?", key: "trait", rows: 2 }, { label: "What are you made of as an organization?", key: "madeof", rows: 2 } ],
    banner: [ { label: "What position are you claiming above the chaos of competition?", key: "position" }, { label: "How do you raise your work to planetary stakes?", key: "stakes" }, { label: "What alternative direction do you offer?", key: "direction" }, { label: "What territory are you holding for others to find?", key: "territory", rows: 2 } ],
    sword: [ { label: "What specific action do you take when problems arise?", key: "action" }, { label: "How does this action cut through BS and strike at the heart?", key: "cut" }, { label: "What does 'swinging your sword' look like in practice?", key: "lookslike", rows: 2 }, { label: "How does this action both defend and advance your purpose?", key: "defend", rows: 2 } ],
    looking_glass: [ { label: "When I follow my purpose all the way, what becomes possible for me personally?", key: "personal" }, { label: "What becomes possible for my organization?", key: "org" }, { label: "What becomes possible for my field?", key: "field" }, { label: "The transformation that seemed impossible becomes:", key: "impossible", rows: 2 } ],
    globe: [ { label: "My organization 100 years from now:", key: "100" }, { label: "The structure and culture that emerges:", key: "culture" }, { label: "How our Character has evolved:", key: "character" }, { label: "What daily life looks like in this future:", key: "daily", rows: 2 } ],
    map: [ { label: "HORIZON 1 (Closest scale of impact):", key: "h1" }, { label: "HORIZON 2 (Mid-range scale of impact):", key: "h2" }, { label: "HORIZON 3 (Furthest scale of impact):", key: "h3" }, { label: "How to move between horizons:", key: "move", rows: 2 } ],
    transformation: [ { label: "Who I must become to realize my vision:", key: "become" }, { label: "Who I would be in that achieved future:", key: "future" }, { label: "The identity change that enables and results from my vision:", key: "change" }, { label: "The impact my transformation creates in others:", key: "impact", rows: 2 } ],
    fountain: [ { label: "The people and networks I have access to:", key: "people" }, { label: "Resources we can pool together:", key: "pool" }, { label: "Our actual sources of wealth:", key: "wealth" }, { label: "How these resources can be shared for immediate use:", key: "share", rows: 2 } ],
    ethos: [ { label: "My unwavering principles are:", key: "principles" }, { label: "How they guide decisions in uncertainty:", key: "guide" }, { label: "The specific agreements that govern us:", key: "agreements" }, { label: "How our word becomes law:", key: "law", rows: 2 } ],
    plot: [ { label: "The main trials I must overcome:", key: "trials" }, { label: "What character traits each trial builds:", key: "traits" }, { label: "Which people I need for these trials:", key: "people" }, { label: "How tasks become adventures:", key: "adventures", rows: 2 } ],
    grail: [ { label: "The transformation I went through that becomes my gift:", key: "gift" }, { label: "How I productize this into an offering:", key: "productize" }, { label: "Who needs to buy into becoming who I've become:", key: "needs" }, { label: "How my need becomes the gift that transforms others:", key: "transforms", rows: 2 } ],
    initiation: [ { label: "My actual journey with its ups, downs, hurts, and gifts:", key: "journey" }, { label: "How I flip this into a pathway for others:", key: "pathway" }, { label: "My initiation process from strangers to allies:", key: "process" }, { label: "How I attract and motivate people to join the story:", key: "attract", rows: 2 } ],
    campfire: [ { label: "How I scale from personal to collective journey:", key: "scale" }, { label: "The movement container for people who need this story:", key: "container" }, { label: "How my story becomes our collective story:", key: "collective" }, { label: "What legacy burns beyond my lifetime:", key: "legacy", rows: 2 } ],
    messages: [ { label: "MESSAGE 1 - WHERE WE ARE IN THE STORY:", key: "where", rows: 2 }, { label: "MESSAGE 2 - WHAT WE NEED:", key: "need", rows: 2 }, { label: "MESSAGE 3 - WHAT WE OFFER:", key: "offer", rows: 2 }, { label: "MESSAGE 4 - THE WAKE-UP CALL:", key: "wakeup", rows: 2 }, { label: "MESSAGE 5 - THE INVITATION:", key: "invitation", rows: 2 } ],
};

const synthesisPrompts = {
    calling_synthesis: "You have faced the Dragon, found your Meaning, forged your Shield, and found your Voice. Distill this into the statement of your Calling.",
    quest_synthesis: "You have found your Star, defined your Character, planted your Banner, and forged your Sword. Distill this into your Quest statement.",
    vision_synthesis: "You have gazed into the Looking Glass, imagined your Globe, charted your Map, and embraced your Transformation. Distill this into your Vision statement.",
    mission_synthesis: "You have tapped your Fountain, written your Ethos, designed your Plot, and calibrated your Compass. Distill this into your Mission statement.",
    kindred_synthesis: "You have filled your Grail, mapped the Initiation, lit the Campfire, and crafted your Messages. Distill this into your invitation to the world.",
};

const staticContent = {
    howto: `<div class="flex items-center mb-4"><span class="artifiction-icon">📖</span><h2 class="text-4xl font-bold font-cinzel">How to Rewrite Your Life</h2></div><div class="prose max-w-none"><p>What follows is the journey home to yourself, to the purpose-driven world-builder you've always been beneath the stories that were never yours to begin with. This is about rewriting your life story on purpose. For too long, you've been a background extra wandering through scenes that drain your soul and waste your gifts. But you can take charge of that story, make it an epic, and transform into the hero you were always meant to be.</p><p>This journey uses a powerful structure, a MythOS. It is <strong>mythogenetic technology</strong>, a living operating system. Think of reality as a set of possible stories encoded in our DNA. The current environment—a culture of scarcity and fear—activates the stories that keep us small, contracted, and afraid. This Metamyth is the epigenetic switch. It's the technology that allows you to consciously choose which stories to express, moving from a state of fear and contraction into one of love and expansion.</p><p>And in this Age of AI, our ancient power of storytelling has been amplified beyond measure. This framework is the <strong>master prompt for your organizational and personal DNA</strong>. As you fill this out, you are writing the code for your reality. Therefore, there is only one rule: <strong>Be impeccable with your word.</strong></p><h3>The Tools of Transformation</h3><p>You will use two kinds of tools on this journey:</p><ul><li><strong>The 5 Artifacts:</strong> These are the great story arcs of your journey, the core movements that structure your transformation.</li><li><strong>The 22 Artifictions:</strong> These are the living tools, weapons of meaning, and magical items you will forge along the way. They are what enable you to rearrange reality.</li></ul><p>As you define these elements, you are training a personal wise guide, an interactive <strong>Wizard</strong> that will remember your story and help you navigate the path ahead.</p></div>`,
    artifact1: `<h2 class="text-4xl font-bold mb-4 font-cinzel">Artifact I: The Call to Adventure</h2><div class="prose max-w-none"><p>Every meaningful journey begins with a shattering. The story starts not with a gentle nudge, but with a disruption that cracks the shell of the old world. Through that crack, light floods in, showing you the game is rigged, the story you've been living belongs to someone else, and reality itself is far more malleable than anyone admitted. This is the necessary separation that precedes all creation.</p></div>`,
    artifact2: `<h2 class="text-4xl font-bold mb-4 font-cinzel">Artifact II: The Quest</h2><div class="prose max-w-none"><p>Once the old world shatters and changes you, you must do something about it. The chaotic energy of the Call to Adventure must be focused. This is where purpose is given direction. The Quest is the conscious act of taking the "why" you discovered in your suffering and turning it into a focused vector. It’s about understanding you are here to play a specific role that only you can play in the great transformation.</p></div>`,
    artifact3: `<h2 class="text-4xl font-bold mb-4 font-cinzel">Artifact III: What is Possible</h2><div class="prose max-w-none"><p>The reward for undertaking the Quest is a change in perspective. You ascend the mountain and can finally see above the clouds. From this vantage point, you can see both what's dying and what's trying to be born. The central question is no longer "what is feasible?" or "what is pragmatic?" but the audacious and world-creating question: "WHAT IS POSSIBLE?"</p></div>`,
    artifact4: `<h2 class="text-4xl font-bold mb-4 font-cinzel">Artifact IV: The Journey</h2><div class="prose max-w-none"><p>Vision without action is just a hallucination. Now you must come down from the mountaintop and ground your magnificent vision in the messy, practical reality of the here and now. The Journey is born from the creative tension between the world you saw in your vision and the world you currently inhabit. This is where you translate the "what is possible" into the "what we must do."</p></div>`,
    artifact5: `<h2 class="text-4xl font-bold mb-4 font-cinzel">Artifact V: The Return</h2><div class="prose max-w-none"><p>The hero's journey is a circle, not a line. The end of one story is the beginning of the next. You return to the world with a gift, an elixir, a transformative power won through your trials. Your personal transformation becomes the offering that transforms the world, seeding the origin story for countless others.</p></div>`,
    wizard: `<div class="flex items-center mb-4"><span class="artifiction-icon">🧙</span><h2 class="text-4xl font-bold font-cinzel">The Wizard</h2></div><div class="prose max-w-none"><p>Having mastered the complete journey, you become the Wizard—the one who has integrated all 22 artifictions into a living wisdom system. The Wizard knows which story tools to activate for different challenges.</p><h3>Examples of Wizard Guidance:</h3><p><strong>"You're out of money?"</strong><br>"Combine Fountain (abundance activation) + Messages (heart communication) + Banner (unique positioning) while staying true to Compass values. Resources flow when story coheres."</p><p><strong>"Your team has lost direction?"</strong><br>"Return to Star (purpose clarity) + Map (milestone tracking) + Plots (adventure structure). Meaning motivates more than management."</p><p><strong>"You're attracting wrong people?"</strong><br>"Strengthen Sword (authentic truth) + Voice (clear expression) + Shield (protective qualification). Your realness will sort the aligned from misaligned."</p></div>`,
    activate_wizard: `<div class="flex items-center mb-4"><span class="artifiction-icon">🤖</span><h2 class="text-4xl font-bold font-cinzel">Activate Your AI Wizard</h2></div><div class="prose max-w-none"><p>The entire Metamyth framework you have just completed is designed to be the perfect <strong>Custom Instruction</strong> for a large language model. By feeding it your completed story, you transform a generalist AI into your specialist Wizard, a wise guide that reflects your own deepest truths and understands your world completely.</p><h3>Step 1: Synthesize Your Metamyth</h3><p>Proceed to the final "Your Metamyth" stage to see your full story synthesized from your inputs. This is your core narrative DNA.</p><h3>Step 2: Craft Your Custom Instruction</h3><p>Copy your synthesized story and paste it into the "Custom Instructions" or equivalent field of your chosen AI. Preface it with a clear directive like the one below.</p><div class="bg-black bg-opacity-20 p-4 rounded-lg border border-slate-700 my-4"><p><strong>Example Prompt Template:</strong></p><p class="text-sm italic mt-2">"You are my Wizard, a wise and strategic guide. Your entire identity, memory, and purpose are defined by the Metamyth that follows... [Paste your synthesized Metamyth narrative here.]"</p></div><h3>Step 3: Interact and Play the Game</h3><p>Once your Wizard is programmed, you can consult it to navigate your life's adventure. Treat it as your trusted advisor.</p></div>`,
    epilogue: `<div class="text-center"><h2 class="text-5xl font-bold mb-4 font-cinzel">The Great Adventure</h2><div class="prose max-w-none mx-auto mt-8"><p>This is not the end. This is the true beginning of your life. This journey is your <strong>Assent</strong>. You are moving from the manufactured \`consent\` of being a character in someone else's story to the wholehearted \`assent\` of becoming an author of your own. This is your declaration of sovereignty.</p><p>Your purpose is not to stay in the same story forever. It is to grow. It is to change. When you have lived this story to its fullest, the entire Metamyth you have just crafted becomes the <strong>Origin Story for your next great adventure.</strong></p><p>Live this Metamyth. Be an example. Share this framework, not as a doctrine, but as an invitation. Invite others to become characters in a greater story than any of us can imagine individually, a shared story of a world remade.</p><p class="font-serif italic text-xl mt-8 text-white">Let this life be a free and beautiful adventure, now and forevermore. Let it begin.</p></div></div>`,
};

</script>

<style>
:root {
    --bg-main: #1D4241;
    --bg-card: #183332;
    --text-main: #E0E0E0;
    --text-heading: #D4AF37; /* Gold */
    --text-muted: #A0A0A0;
    --border-color: #2c5d5b;
    --accent-turqoise: #5FE2DF;
    --accent-turqoise-dark: #4ac0bd;
    --accent-red: #a43743; /* Muted Crimson for text */
}
.font-lora {
    font-family: 'Lora', serif;
}
.font-cinzel {
    font-family: 'Cinzel', serif;
}
body {
    background-color: var(--bg-main);
    color: var(--text-main);
}
h1, h2, h3, h4 {
    font-family: 'Cinzel', serif;
    color: var(--text-heading);
    letter-spacing: 0.05em;
}
.content-card {
    background-color: var(--bg-card);
    border: 1px solid var(--border-color);
    border-radius: 0.5rem;
    padding: 2.5rem;
    box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.2), 0 4px 6px -2px rgba(0, 0, 0, 0.1);
}
.nav-link {
    transition: all 0.2s ease-in-out;
    border-left: 3px solid transparent;
    color: var(--text-muted);
    padding: 0.75rem 1rem;
}
.nav-link.active {
    background-color: rgba(95, 226, 223, 0.05);
    color: var(--accent-turqoise);
    border-left-color: var(--accent-turqoise);
}
.nav-link:hover {
    background-color: rgba(95, 226, 223, 0.1);
    color: #fff;
}
textarea, .compass-input {
    background-color: rgba(0,0,0,0.2);
    border: 1px solid var(--border-color);
    border-radius: 0.5rem;
    padding: 0.75rem;
    width: 100%;
    transition: border-color 0.2s ease-in-out;
    font-size: 1rem;
    color: var(--text-main);
    font-family: 'Lora', serif;
}
textarea:focus, .compass-input:focus {
    outline: none;
    border-color: var(--accent-turqoise);
    box-shadow: 0 0 0 2px rgba(95, 226, 223, 0.3);
}
.btn-primary {
    background-color: var(--accent-turqoise);
    color: #0c2a29;
    padding: 0.75rem 1.5rem;
    border-radius: 0.5rem;
    font-family: 'Cinzel', serif;
    font-weight: 700;
    transition: all 0.2s ease-in-out;
    cursor: pointer;
    text-transform: uppercase;
    letter-spacing: 0.1em;
}
.btn-primary:hover {
    background-color: var(--accent-turqoise-dark);
    box-shadow: 0 0 15px rgba(95, 226, 223, 0.4);
}
.btn-secondary {
    background-color: transparent;
    color: var(--accent-turqoise);
    border: 1px solid var(--accent-turqoise);
    padding: 0.5rem 1rem;
    border-radius: 0.5rem;
    font-family: 'Cinzel', serif;
    font-weight: 700;
    transition: all 0.2s ease-in-out;
    cursor: pointer;
}
.btn-secondary:hover {
    background-color: rgba(95, 226, 223, 0.1);
}
.artifiction-icon {
    font-size: 2.5rem;
    margin-right: 1.5rem;
    line-height: 1;
}
.stage-content.active {
    display: block;
    animation: fadeIn 0.6s ease-in-out;
}
@keyframes fadeIn {
    from { opacity: 0; transform: translateY(15px); }
    to { opacity: 1; transform: translateY(0); }
}
.chart-container {
    position: relative;
    width: 100%;
    max-width: 500px;
    height: auto;
    margin: 2rem auto;
}
.prose {
    color: var(--text-main);
    line-height: 1.8;
}
.prose h3 {
    font-size: 1.5rem;
    font-weight: 700;
    margin-top: 2em;
    margin-bottom: 0.5em;
    color: var(--text-heading);
    border-bottom: 1px solid var(--border-color);
    padding-bottom: 0.5rem;
}
.prose p { margin-bottom: 1.25em; }
.prose strong { color: var(--accent-turqoise); }
.nav-artifact-header {
    font-family: 'Cinzel', serif;
    color: #fff;
    font-weight: 700;
    letter-spacing: 0.1em;
}
.crimson-text { color: var(--accent-red); }
.synthesis-box {
    background-color: rgba(0,0,0,0.1);
    border-left: 4px solid var(--accent-turqoise);
    padding: 1.5rem;
    border-radius: 0.5rem;
    font-family: 'Lora', serif;
    font-style: italic;
    color: #E5E7EB;
}
.ornamental-divider {
    height: 1px;
    background: linear-gradient(to right, transparent, var(--border-color), transparent);
    margin: 3rem 0;
}
input[type=range] {
    -webkit-appearance: none;
    width: 100%;
    background: transparent;
}
input[type=range]:focus {
    outline: none;
}
input[type=range]::-webkit-slider-runnable-track {
    width: 100%;
    height: 4px;
    cursor: pointer;
    background: var(--border-color);
    border-radius: 5px;
}
input[type=range]::-webkit-slider-thumb {
    -webkit-appearance: none;
    border: 2px solid var(--bg-main);
    height: 20px;
    width: 20px;
    border-radius: 50%;
    background: var(--accent-turqoise);
    cursor: pointer;
    margin-top: -8px;
}
input[type=range]::-moz-range-track {
    width: 100%;
    height: 4px;
    cursor: pointer;
    background: var(--border-color);
    border-radius: 5px;
}
input[type=range]::-moz-range-thumb {
    border: 2px solid var(--bg-main);
    height: 20px;
    width: 20px;
    border-radius: 50%;
    background: var(--accent-turqoise);
    cursor: pointer;
}
</style>
