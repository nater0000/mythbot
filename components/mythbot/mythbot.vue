<template>
  <div class="flex flex-col md:flex-row min-h-screen bg-main text-main font-lora">
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

    <main class="flex-1 p-4 md:p-10 overflow-y-auto">
      <div v-if="activeStage" class="stage-content active">
        <div class="content-card" v-html="stageContent(activeStage)"></div>
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

// Reactive state for stages, data, and active stage
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
let compassChart = null;

const compassPoints = ['Purpose', 'Vision', 'Truth', 'Creativity', 'Community', 'Power', 'Economy-Ecology', 'System Integration'];

// Group stages by artifact for navigation
const groupedStages = computed(() => {
  return stages.value.reduce((acc, stage) => {
    (acc[stage.artifact] = acc[stage.artifact] || []).push(stage);
    return acc;
  }, {});
});

const activeStageIndex = computed(() => {
    if (!activeStage.value) return -1;
    return stages.value.findIndex(s => s.id === activeStage.value.id);
});

const showNavigationButtons = computed(() => {
    if (!activeStage.value) return false;
    const nonNavIds = ['intro', 'epilogue'];
    return !nonNavIds.includes(activeStage.value.id);
});

// Function to set the active stage
const showStage = (stage) => {
  activeStage.value = stage;
  window.scrollTo(0, 0);
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

// Computed property to generate stage HTML
const stageContent = (stage) => {
  if (!stage) return '';

  const getDesc = (id) => ({/*...descriptions from original file...*/}[id] || '');
  const getPrompts = (id) => ({/*...prompts from original file...*/}[id] || []);
  
  // (For brevity, the large description and prompt objects are omitted here)
  // You would paste the 'descriptions' and 'prompts' objects from the original JS here.
  const descriptions = {
      dragon: "Look around with fresh eyes...",
      // ...all other descriptions
  };
  const prompts = {
      dragon: [ { label: "What civilizational madness...", key: "madness", rows: 4 }, /*...*/ ],
      // ...all other prompts
  };

  const stageContentHTMLs = {
    intro: `<div class="text-center"><h2 class="text-5xl font-bold mb-4">Awaken.</h2>...</div>`,
    howto: `<div class="flex items-center mb-4">...</div>`,
    origin: `<div class="flex items-center mb-4">...</div>`,
    //... Paste ALL stage-specific HTML content here from the original file
  };
  
  if (stageContentHTMLs[stage.id]) return stageContentHTMLs[stage.id];

  // Default template for stages with prompts
  const stagePrompts = prompts[stage.id];
  if (stagePrompts) {
      const promptsHtml = stagePrompts.map(p => `<div><label class="font-semibold text-lg text-white block mb-2">${p.label}</label><textarea data-stage="${stage.id}" data-field="${p.key}" class="w-full" rows="${p.rows || 3}">${metamythData.value[stage.id]?.[p.key] || ''}</textarea></div>`).join('');
      return `<div class="flex items-center mb-4"><span class="artifiction-icon ${stage.iconClass || ''}">${stage.icon}</span><h2 class="text-4xl font-bold">${stage.title}</h2></div><div class="prose max-w-none mb-8"><p>${descriptions[stage.id]}</p></div><div class="ornamental-divider"></div><div class="space-y-8">${promptsHtml}</div>`;
  }
  
  return `<h2 class="text-4xl font-bold">${stage.title}</h2>`;
};


const initializeCompassData = () => {
    const compassData = {};
    compassPoints.forEach(point => {
        const key = point.toLowerCase().replace(/[^a-z]/g, '');
        compassData[key] = 5;
    });
    metamythData.value.compass = compassData;
};

const renderCompassChart = () => {
  nextTick(() => {
    const ctx = document.getElementById('compassChart')?.getContext('2d');
    if (!ctx) return;
    
    const labels = compassPoints.map(p => p.replace('Economy-Ecology', 'Economy').replace('System Integration', 'Integration'));
    const data = compassPoints.map(p => metamythData.value.compass[p.toLowerCase().replace(/[^a-z]/g, '')] || 5);

    if (compassChart) {
      compassChart.destroy();
    }

    compassChart = new Chart(ctx, {
      type: 'radar',
      data: {
        labels: labels,
        datasets: [{
          label: 'Your Alignment',
          data: data,
          backgroundColor: 'rgba(95, 226, 223, 0.2)',
          borderColor: 'rgba(95, 226, 223, 1)',
          borderWidth: 2,
          pointBackgroundColor: 'rgba(95, 226, 223, 1)'
        }]
      },
      options: { /* ... Chart options from original file ... */ }
    });
  });
};

// Update chart when compass data changes
watch(() => metamythData.value.compass, () => {
    if (compassChart && activeStage.value?.id === 'compass') {
        const labels = compassChart.data.labels;
        compassChart.data.datasets[0].data = labels.map(l => metamythData.value.compass[l.toLowerCase().replace(/[^a-z]/g, '')] || 0);
        compassChart.update();
    }
}, { deep: true });

// Watch for active stage changes to render the chart
watch(activeStage, (newStage) => {
    if (newStage && newStage.id === 'compass') {
        renderCompassChart();
    }
});


onMounted(() => {
  // Initialize and show the first stage
  if (stages.value.length > 0) {
    showStage(stages.value[0]);
  }
  initializeCompassData();

  // Event delegation for textarea inputs
  document.querySelector('.flex-1')?.addEventListener('input', (e) => {
      if (e.target.tagName === 'TEXTAREA' && e.target.dataset.stage) {
          const { stage, field } = e.target.dataset;
          if (!metamythData.value[stage]) {
              metamythData.value[stage] = {};
          }
          metamythData.value[stage][field] = e.target.value;
      }
  });
});

</script>

<style>
/* Paste the entire <style> block from mythbot.html here */
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
body {
    font-family: 'Lora', serif;
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
.stage-content { display: none; }
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
/* Custom Slider Styles */
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