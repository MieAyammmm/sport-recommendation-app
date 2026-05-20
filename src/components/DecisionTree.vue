<script setup>
import { ref, computed, onMounted } from "vue";
import axios from "axios";

const API = "http://127.0.0.1:8000/api/decision-tree";

const tree = ref(null);
const currentNode = ref(null);
const phase = ref("start");
const loading = ref(false);
const errorMsg = ref("");
const history = ref([]);
const totalQuestions = ref(0);
const questionCount = ref(0);


const sportImages = {
  renang:
    "https://images.unsplash.com/photo-1530549387789-4c1017266635?w=1600&q=80",
  jalan:
    "https://images.unsplash.com/photo-1476480862126-209bfaa8edc8?w=1600&q=80",
  yoga: "https://images.unsplash.com/photo-1544367567-0f2fcb009e0b?w=1600&q=80",
  bersepeda:
    "https://images.unsplash.com/photo-1541625602330-2277a4c46182?w=1600&q=80",
  sepeda:
    "https://images.unsplash.com/photo-1541625602330-2277a4c46182?w=1600&q=80",
  stretching:
    "https://images.unsplash.com/photo-1571019613454-1cb2f99b2d8b?w=1600&q=80",
  peregangan:
    "https://images.unsplash.com/photo-1571019613454-1cb2f99b2d8b?w=1600&q=80",
  default:
    "https://images.unsplash.com/photo-1517836357463-d25dfeac3438?w=1600&q=80",
};

function getResultImage(recommendation) {
  if (!recommendation) return sportImages.default;
  const lower = recommendation.toLowerCase();
  for (const [key, url] of Object.entries(sportImages)) {
    if (key !== "default" && lower.includes(key)) return url;
  }
  return sportImages.default;
}


function getSportName(recommendation) {
  if (!recommendation) return "";
  const match = recommendation.match(/^[^\.\n]+/);
  return match ? match[0].replace(/^[🏊🚶🧘🚴🎉❤️]+\s*/u, "").trim() : "";
}

function countDepth(node) {
  if (!node || node.recommendation) return 0;
  return 1 + Math.max(countDepth(node.yes), countDepth(node.no));
}

async function fetchTree() {
  loading.value = true;
  errorMsg.value = "";
  try {
    const res = await axios.get(API);
    tree.value = res.data.tree;
    totalQuestions.value = countDepth(tree.value);
  } catch (err) {
    errorMsg.value = "Gagal memuat data. Pastikan server Laravel berjalan.";
  }
  loading.value = false;
}

function start() {
  if (!tree.value) return;
  currentNode.value = tree.value;
  history.value = [];
  questionCount.value = 0;
  phase.value = "question";
}

function answer(choice) {
  history.value.push({ node: currentNode.value });
  questionCount.value++;
  const next = choice === "yes" ? currentNode.value.yes : currentNode.value.no;
  currentNode.value = next;
  if (next.recommendation) phase.value = "result";
}

function goBack() {
  if (history.value.length === 0) return;
  const prev = history.value.pop();
  currentNode.value = prev.node;
  questionCount.value--;
  phase.value = "question";
}

function reset() {
  currentNode.value = null;
  history.value = [];
  questionCount.value = 0;
  phase.value = "start";
}

const progress = computed(() =>
  totalQuestions.value === 0
    ? 0
    : Math.min((questionCount.value / totalQuestions.value) * 100, 95),
);

const resultImage = computed(() =>
  phase.value === "result"
    ? getResultImage(currentNode.value?.recommendation)
    : "",
);

const sportName = computed(() =>
  phase.value === "result"
    ? getSportName(currentNode.value?.recommendation)
    : "",
);


const questionColors = [
  "#0d1117",
  "#0a0f1a",
  "#0f0a1a",
  "#0a1a0f",
  "#1a0a0a",
  "#0a1414",
  "#141400",
];
const questionBg = computed(
  () => questionColors[questionCount.value % questionColors.length],
);

onMounted(fetchTree);
</script>

<template>
  <div class="page">
    <!-- NAV -->
    <nav class="nav">
      <span class="nav-logo">FitPath!</span>
      <span class="nav-label">Olahraga &amp; Kesehatan</span>
    </nav>

    <Transition name="fade" mode="out-in">
    
      <section v-if="phase === 'start'" key="start" class="screen screen--dark">
       
        <div class="bg-layer">
          <svg
            viewBox="0 0 1440 900"
            preserveAspectRatio="xMidYMid slice"
            xmlns="http://www.w3.org/2000/svg"
            class="bg-svg"
          >
            <rect width="1440" height="900" fill="#080c10" />
            <circle
              cx="720"
              cy="380"
              r="320"
              fill="none"
              stroke="#10172a"
              stroke-width="100"
            />
            <circle
              cx="720"
              cy="380"
              r="170"
              fill="none"
              stroke="#0d1220"
              stroke-width="60"
            />
            <circle cx="720" cy="380" r="60" fill="#12192e" />
     
            <g
              transform="translate(680,300)"
              stroke="#1e2d4a"
              stroke-linecap="round"
              fill="none"
            >
              <circle cx="40" cy="0" r="20" fill="#1a2840" stroke="none" />
              <line x1="40" y1="20" x2="40" y2="80" stroke-width="10" />
              <line x1="40" y1="40" x2="10" y2="68" stroke-width="8" />
              <line x1="40" y1="40" x2="68" y2="62" stroke-width="8" />
              <line x1="40" y1="80" x2="16" y2="115" stroke-width="9" />
              <line x1="40" y1="80" x2="62" y2="115" stroke-width="9" />
            </g>
         
            <line
              x1="0"
              y1="560"
              x2="1440"
              y2="560"
              stroke="#0f1620"
              stroke-width="1"
            />
            <line
              x1="0"
              y1="570"
              x2="1440"
              y2="570"
              stroke="#0c1218"
              stroke-width="1"
            />
          </svg>
        </div>

        <div class="content content--left">
          <p class="eyebrow">Pemilihan Olahraga · Jalur Kebugaran Anda</p>
          <h1 class="title--hero">
            Temukan Olahraga<br />yang Tepat untuk Anda
          </h1>
          <p
            class="body-text"
            style="
              color: rgba(255, 255, 255, 0.55);
              max-width: 420px;
              margin-bottom: 40px;
            "
          >
            Jawab beberapa pertanyaan singkat dan dapatkan rekomendasi olahraga
            sesuai kondisi kesehatan Anda.
          </p>
          <div v-if="errorMsg" class="error-bar">{{ errorMsg }}</div>
          <div class="actions">
            <button
              class="btn-primary"
              @click="start"
              :disabled="!tree || loading"
            >
              {{ loading ? "Memuat..." : "Mulai Konsultasi" }}
            </button>
          </div>
          <p class="disclaimer">
            ⚠ Konsultasikan dengan dokter sebelum memulai olahraga baru
          </p>
        </div>
      </section>

      <!-- ======== QUESTION ======== -->
      <section
        v-else-if="phase === 'question'"
        key="question"
        class="screen screen--dark"
        :style="{ '--q-bg': questionBg }"
      >
    
        <div class="bg-layer">
          <svg
            viewBox="0 0 1440 900"
            preserveAspectRatio="xMidYMid slice"
            xmlns="http://www.w3.org/2000/svg"
            class="bg-svg"
          >
            <rect width="1440" height="900" :fill="questionBg" />
   
            <circle
              cx="1100"
              cy="200"
              r="300"
              fill="none"
              stroke-width="80"
              :stroke="questionBg"
              opacity="0.6"
              style="filter: brightness(1.4)"
            />
            <circle
              cx="1100"
              cy="200"
              r="160"
              fill="none"
              stroke-width="50"
              :stroke="questionBg"
              opacity="0.5"
              style="filter: brightness(1.6)"
            />
           
            <line
              v-for="i in 8"
              :key="i"
              x1="0"
              :y1="i * 110"
              x2="1440"
              :y2="i * 110"
              stroke-width="0.5"
              stroke="rgba(255,255,255,0.03)"
            />
          </svg>
        </div>

      
        <div class="progress-top">
          <div class="progress-track">
            <div class="progress-fill" :style="{ width: progress + '%' }"></div>
          </div>
          <span class="progress-label"
            >{{ questionCount + 1 }} / {{ totalQuestions }}</span
          >
        </div>

        
        <div class="content content--center">
          <Transition name="slide" mode="out-in">
            <div :key="currentNode?.question" class="question-wrap">
              <p class="eyebrow" style="color: rgba(255, 255, 255, 0.4)">
                Pertanyaan {{ questionCount + 1 }}
              </p>
              <h2 class="title--question">{{ currentNode?.question }}</h2>
              <div class="choice-row">
                <button class="btn-primary" @click="answer('yes')">Ya</button>
                <button class="btn-outline" @click="answer('no')">Tidak</button>
              </div>
              <button
                v-if="history.length > 0"
                class="btn-back"
                @click="goBack"
              >
                ← Kembali
              </button>
            </div>
          </Transition>
        </div>
      </section>

      <!-- ======== RESULT ======== -->
      <section
        v-else-if="phase === 'result'"
        key="result"
        class="screen screen--dark result-screen"
      >
   
        <div
          class="bg-layer bg-layer--photo"
          :style="{ backgroundImage: `url(${resultImage})` }"
        ></div>
      
        <div class="bg-layer bg-overlay"></div>

        <div class="content content--center result-content">
          <p class="eyebrow" style="color: #3e6ae1; letter-spacing: 0.12em">
            Rekomendasi untuk Anda
          </p>
          <h2 class="title--result">{{ sportName }}</h2>
          <p class="result-detail">{{ currentNode?.recommendation }}</p>
          <div class="result-divider"></div>
          <div class="actions">
            <button class="btn-primary" @click="reset">Konsultasi Ulang</button>
          </div>
          <p class="disclaimer" style="color: rgba(255, 255, 255, 0.3)">
            ⚠ Konsultasikan dengan dokter sebelum memulai olahraga baru
          </p>
        </div>
      </section>
    </Transition>
  </div>
</template>

<style>

html,
body,
#app {
  margin: 0;
  padding: 0;
  width: 100%;
  min-height: 100vh;
  background: #080c10;
  overflow-x: hidden;
}
</style>

<style scoped>

*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

.page {
  width: 100vw;
  min-height: 100vh;
  font-family: -apple-system, Arial, sans-serif;
  color: #171a20;
  overflow-x: hidden;
}


.nav {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 200;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 56px;
  height: 56px;
  background: rgba(8, 12, 16, 0.7);
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
}

.nav-logo {
  font-size: 13px;
  font-weight: 500;
  letter-spacing: 0.22em;
  color: rgba(255, 255, 255, 0.9);
}

.nav-label {
  font-size: 13px;
  font-weight: 400;
  color: rgba(255, 255, 255, 0.4);
}

.screen {
  position: relative;
  width: 100vw;
  min-height: 100vh;
  display: flex;
  align-items: center;
}

.screen--dark {
  background: #080c10;
}


.bg-layer {
  position: absolute;
  inset: 0;
  z-index: 0;
}

.bg-svg {
  width: 100%;
  height: 100%;
  display: block;
}

.bg-layer--photo {
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  transition: background-image 0.6s;
}

.bg-overlay {
  background: linear-gradient(
    to right,
    rgba(0, 0, 0, 0.85) 0%,
    rgba(0, 0, 0, 0.6) 50%,
    rgba(0, 0, 0, 0.3) 100%
  );
}


.content {
  position: relative;
  z-index: 10;
  width: 100%;
  padding: 80px 56px 56px;
}

.content--left {
  max-width: 640px;
}

.content--center {
  max-width: 680px;
  margin: 0 auto;
  text-align: center;
}

.eyebrow {
  font-size: 13px;
  font-weight: 400;
  color: rgba(255, 255, 255, 0.45);
  letter-spacing: 0.1em;
  margin-bottom: 16px;
}

.title--hero {
  font-size: 48px;
  font-weight: 500;
  line-height: 1.15;
  color: #ffffff;
  margin-bottom: 20px;
}

.title--question {
  font-size: 32px;
  font-weight: 500;
  line-height: 1.3;
  color: #ffffff;
  margin-bottom: 44px;
}

.title--result {
  font-size: 52px;
  font-weight: 500;
  line-height: 1.1;
  color: #ffffff;
  margin-bottom: 20px;
  text-transform: uppercase;
  letter-spacing: 0.04em;
}

.body-text {
  font-size: 15px;
  font-weight: 400;
  line-height: 1.65;
  color: rgba(255, 255, 255, 0.55);
}

.result-detail {
  font-size: 15px;
  font-weight: 400;
  line-height: 1.75;
  color: rgba(255, 255, 255, 0.75);
  max-width: 520px;
  margin: 0 auto 36px;
}

.result-divider {
  width: 48px;
  height: 1px;
  background: rgba(255, 255, 255, 0.2);
  margin: 0 auto 36px;
}


.progress-top {
  position: absolute;
  top: 56px;
  left: 56px;
  right: 56px;
  z-index: 10;
  display: flex;
  align-items: center;
  gap: 16px;
  padding-top: 16px;
}

.progress-track {
  flex: 1;
  height: 2px;
  background: rgba(255, 255, 255, 0.12);
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: #3e6ae1;
  transition: width 0.33s cubic-bezier(0.5, 0, 0, 0.75);
}

.progress-label {
  font-size: 12px;
  color: rgba(255, 255, 255, 0.35);
  white-space: nowrap;
}


.actions {
  display: flex;
  gap: 12px;
  margin-bottom: 28px;
}

.content--center .actions {
  justify-content: center;
}


.choice-row {
  display: flex;
  gap: 12px;
  justify-content: center;
  margin-bottom: 28px;
}


.btn-primary {
  background: #3e6ae1;
  color: #fff;
  border: 3px solid transparent;
  border-radius: 4px;
  padding: 0 28px;
  height: 44px;
  font-size: 14px;
  font-weight: 500;
  font-family: inherit;
  cursor: pointer;
  min-width: 160px;
  transition:
    background-color 0.33s,
    box-shadow 0.25s;
  letter-spacing: 0.01em;
}

.btn-primary:hover {
  background: #2f59cc;
}
.btn-primary:disabled {
  background: #555;
  cursor: not-allowed;
}

.btn-outline {
  background: transparent;
  color: #fff;
  border: 2px solid rgba(255, 255, 255, 0.35);
  border-radius: 4px;
  padding: 0 28px;
  height: 44px;
  font-size: 14px;
  font-weight: 500;
  font-family: inherit;
  cursor: pointer;
  min-width: 160px;
  transition:
    border-color 0.33s,
    background-color 0.33s;
}

.btn-outline:hover {
  border-color: rgba(255, 255, 255, 0.7);
  background: rgba(255, 255, 255, 0.06);
}

.btn-back {
  background: none;
  border: none;
  color: rgba(255, 255, 255, 0.35);
  font-size: 13px;
  font-weight: 400;
  font-family: inherit;
  cursor: pointer;
  padding: 0;
  transition: color 0.33s;
  display: block;
  margin: 0 auto;
}

.btn-back:hover {
  color: rgba(255, 255, 255, 0.75);
}


.error-bar {
  font-size: 13px;
  color: #e74c3c;
  background: rgba(231, 76, 60, 0.1);
  border: 1px solid rgba(231, 76, 60, 0.3);
  border-radius: 4px;
  padding: 10px 16px;
  margin-bottom: 20px;
}

.disclaimer {
  font-size: 11px;
  color: rgba(255, 255, 255, 0.22);
  margin-top: 8px;
}

.result-screen .content {
  padding-top: 80px;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.4s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.slide-enter-active,
.slide-leave-active {
  transition:
    opacity 0.25s,
    transform 0.25s;
}
.slide-enter-from {
  opacity: 0;
  transform: translateX(20px);
}
.slide-leave-to {
  opacity: 0;
  transform: translateX(-20px);
}

@media (max-width: 768px) {
  .nav {
    padding: 0 24px;
  }

  .content {
    padding: 80px 24px 48px;
  }
  .content--left {
    max-width: 100%;
  }

  .title--hero {
    font-size: 32px;
  }
  .title--question {
    font-size: 24px;
  }
  .title--result {
    font-size: 36px;
  }

  .progress-top {
    left: 24px;
    right: 24px;
  }

  .choice-row,
  .actions {
    flex-direction: column;
    align-items: stretch;
  }
  .btn-primary,
  .btn-outline {
    min-width: unset;
    width: 100%;
  }

  .bg-overlay {
    background: rgba(0, 0, 0, 0.72);
  }
}
</style>
