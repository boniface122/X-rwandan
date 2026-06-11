<template>
  <div class="app">

    <!-- ── HEADER ── -->
    <div class="header">
      <div class="header-inner">
        <span class="logo">🏛️</span>
        <div>
          <h1>X Rwandan Museum</h1>
          <p class="location">📍 Musanze District · Muhoza Sector</p>
        </div>
      </div>
    </div>

    <!-- ── MAIN CARD ── -->
    <div class="card">
      <p class="desc">Arrange the 9 pieces to restore the historic photo!</p>

      <!-- Stats -->
      <div class="stats">
        <div class="stat">
          <div class="stat-lbl">🎯 Moves</div>
          <div class="stat-val">{{ moves }}</div>
        </div>
        <div class="stat">
          <div class="stat-lbl">⏱ Time</div>
          <div class="stat-val">{{ formatTime(secs) }}</div>
        </div>
        <div class="stat">
          <div class="stat-lbl">✅ Correct</div>
          <div class="stat-val" :style="{ color: correct === 9 ? '#27ae60' : '#e67e22' }">
            {{ correct }}/9
          </div>
        </div>
      </div>

      <!-- Progress -->
      <div class="prog-wrap">
        <div class="prog-fill" :style="{ width: pct + '%' }"></div>
        <span class="prog-label">{{ pct }}% complete</span>
      </div>

      <!-- Win Banner -->
      <transition name="pop">
        <div v-if="solved" class="win-banner">
          <span class="trophy">🏆</span>
          <strong>Congratulations!</strong><br/>
          Puzzle Complete in <b>{{ moves }} moves</b> &amp; <b>{{ formatTime(secs) }}</b>!
        </div>
      </transition>

      <!-- Puzzle Grid -->
      <div class="grid" :class="{ shake: shaking }">
        <div
          v-for="(val, idx) in pieces"
          :key="idx"
          class="piece"
          :class="{
            selected:  selected === idx,
            'in-place': val === idx && !solved,
            solved: solved
          }"
          :style="tileStyle(val)"
          @click="click(idx)"
        >
          <span class="badge">{{ val + 1 }}</span>
          <span v-if="val === idx && !solved" class="check">✓</span>
        </div>
      </div>

      <!-- Buttons -->
      <div class="btn-row">
        <button class="btn" @click="restart">
          <span>↺</span> Restart
        </button>
        <button class="btn btn-hint" @click="hint">
          <span>💡</span> Hint
        </button>
        <button class="btn btn-preview" @click="preview = true">
          <span>👁</span> Preview
        </button>
      </div>
    </div>

    <!-- ── FOOTER ── -->
    <p class="footer">X Rwandan Museum · Musanze · Rwanda 🇷🇼</p>

    <!-- ── PREVIEW MODAL ── -->
    <transition name="fade">
      <div v-if="preview" class="modal" @click="preview = false">
        <div class="modal-box" @click.stop>
          <h3>🖼 Full Image</h3>
          <img :src="img" alt="rudahigwa.jpg" />
          <p class="modal-hint">This is what the solved puzzle looks like</p>
          <button class="btn btn-close" @click="preview = false">✕ Close</button>
        </div>
      </div>
    </transition>

  </div>
</template>

<script>
import { IMG } from './puzzle.js'

export default {
  name: 'App',

  data() {
    return {
      img: IMG,
      pieces: [],
      selected: null,
      moves: 0,
      solved: false,
      preview: false,
      secs: 0,
      shaking: false,
      _timer: null
    }
  },

  computed: {
    correct() {
      return this.pieces.filter((v, i) => v === i).length
    },
    pct() {
      return Math.round((this.correct / 9) * 100)
    }
  },

  mounted() {
    this.restart()
  },

  beforeUnmount() {
    clearInterval(this._timer)
  },

  methods: {
    tileStyle(val) {
      const col = val % 3
      const row = Math.floor(val / 3)
      return {
        backgroundImage:    `url(${this.img})`,
        backgroundSize:     '300% 300%',
        backgroundPosition: `${col * 50}% ${row * 50}%`
      }
    },

    shuffle(a) {
      let arr = [...a]
      const sorted = JSON.stringify([0,1,2,3,4,5,6,7,8])
      do {
        for (let i = arr.length - 1; i > 0; i--) {
          const j = Math.floor(Math.random() * (i + 1));
          [arr[i], arr[j]] = [arr[j], arr[i]]
        }
      } while (JSON.stringify(arr) === sorted)
      return arr
    },

    restart() {
      this.pieces   = this.shuffle([0,1,2,3,4,5,6,7,8])
      this.selected = null
      this.moves    = 0
      this.solved   = false
      this.secs     = 0
      clearInterval(this._timer)
      this._timer   = setInterval(() => {
        if (!this.solved) this.secs++
      }, 1000)
      // shake animation on restart
      this.shaking = true
      setTimeout(() => { this.shaking = false }, 500)
    },

    click(idx) {
      if (this.solved) return
      if (this.selected === null) {
        this.selected = idx
      } else if (this.selected === idx) {
        this.selected = null
      } else {
        ;[this.pieces[this.selected], this.pieces[idx]] =
         [this.pieces[idx], this.pieces[this.selected]]
        this.moves++
        this.selected = null
        if (this.pieces.every((v, i) => v === i)) {
          this.solved = true
          clearInterval(this._timer)
        }
      }
    },

    hint() {
      if (this.solved) return
      for (let i = 0; i < 9; i++) {
        if (this.pieces[i] !== i) {
          const j = this.pieces.indexOf(i)
          alert(`💡 Hint: Swap piece at position ${i + 1} with piece at position ${j + 1}`)
          return
        }
      }
    },

    formatTime(secs) {
      const m = String(Math.floor(secs / 60)).padStart(2, '0')
      const s = String(secs % 60).padStart(2, '0')
      return `${m}:${s}`
    }
  }
}
</script>

<style>
* { box-sizing: border-box; margin: 0; padding: 0; }

body {
  font-family: 'Segoe UI', sans-serif;
  background: linear-gradient(160deg, #0d0d1a, #1a1a3e, #0f3460);
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 1.5rem;
}

.app {
  width: 100%;
  max-width: 460px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
}

/* ── Header ── */
.header {
  width: 100%;
  background: rgba(255,255,255,0.07);
  border: 1px solid rgba(255,255,255,0.12);
  border-radius: 16px;
  padding: 1rem 1.4rem;
  backdrop-filter: blur(8px);
}
.header-inner { display: flex; align-items: center; gap: 14px; }
.logo { font-size: 36px; }
.header h1 { font-size: 20px; font-weight: 800; color: #fff; }
.location { font-size: 11px; color: #e67e22; font-weight: 600; letter-spacing: 1px; margin-top: 2px; }

/* ── Card ── */
.card {
  width: 100%;
  background: #fff;
  border-radius: 20px;
  padding: 1.6rem;
  box-shadow: 0 12px 50px rgba(0,0,0,0.45);
}
.desc { font-size: 12px; color: #999; text-align: center; margin-bottom: 1.1rem; }

/* ── Stats ── */
.stats { display: flex; gap: 8px; margin-bottom: 1rem; }
.stat { flex: 1; background: #f4f6fb; border-radius: 12px; padding: .6rem .4rem; text-align: center; }
.stat-lbl { font-size: 10px; color: #aaa; margin-bottom: 3px; }
.stat-val { font-size: 18px; font-weight: 700; color: #12122a; }

/* ── Progress ── */
.prog-wrap {
  position: relative;
  background: #eee;
  border-radius: 20px;
  height: 22px;
  margin-bottom: 1.1rem;
  overflow: hidden;
}
.prog-fill {
  height: 100%;
  background: linear-gradient(90deg, #e67e22, #27ae60);
  border-radius: 20px;
  transition: width .45s ease;
}
.prog-label {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 11px;
  font-weight: 700;
  color: #fff;
  mix-blend-mode: difference;
}

/* ── Win Banner ── */
.win-banner {
  background: linear-gradient(135deg, #d4f5e2, #a8f0c6);
  border: 2px solid #27ae60;
  border-radius: 14px;
  padding: 1rem;
  margin-bottom: 1rem;
  color: #145a32;
  font-size: 14px;
  text-align: center;
  line-height: 1.7;
}
.trophy { font-size: 34px; display: block; margin-bottom: 4px; }

/* ── Grid ── */
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 4px;
  margin-bottom: 1.1rem;
  border: 4px solid #12122a;
  border-radius: 12px;
  overflow: hidden;
  background: #12122a;
}

@keyframes shake {
  0%,100% { transform: translateX(0); }
  20%      { transform: translateX(-6px); }
  40%      { transform: translateX(6px); }
  60%      { transform: translateX(-4px); }
  80%      { transform: translateX(4px); }
}
.grid.shake { animation: shake .45s ease; }

.piece {
  aspect-ratio: 1;
  cursor: pointer;
  background-repeat: no-repeat;
  position: relative;
  transition: transform .15s, opacity .15s;
}
.piece:hover             { transform: scale(.95); opacity: .85; }
.piece.selected          { outline: 4px solid #e67e22; outline-offset: -4px; opacity: .7; }
.piece.in-place          { outline: 3px solid #27ae60; outline-offset: -3px; }
.piece.solved            { cursor: default; }

.badge {
  position: absolute;
  bottom: 4px; right: 5px;
  background: rgba(0,0,0,.55);
  color: #fff;
  font-size: 9px;
  font-weight: 700;
  padding: 1px 5px;
  border-radius: 4px;
}
.check {
  position: absolute;
  top: 4px; left: 5px;
  background: #27ae60;
  color: #fff;
  font-size: 9px;
  font-weight: 700;
  padding: 1px 5px;
  border-radius: 4px;
}

/* ── Buttons ── */
.btn-row { display: flex; gap: 8px; }
.btn {
  flex: 1;
  padding: .6rem .4rem;
  border-radius: 10px;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  border: 1.5px solid #d8dde6;
  background: #fff;
  color: #333;
  transition: background .15s, transform .1s;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 5px;
}
.btn:hover          { background: #f0f4f9; transform: translateY(-1px); }
.btn:active         { transform: scale(.97); }
.btn-hint           { border-color: #f0ad4e; color: #c87f00; }
.btn-hint:hover     { background: #fff8ec; }
.btn-preview        { border-color: #7bafd4; color: #1a6699; }
.btn-preview:hover  { background: #edf5ff; }

/* ── Footer ── */
.footer {
  font-size: 11px;
  color: rgba(255,255,255,.4);
  text-align: center;
}

/* ── Modal ── */
.modal {
  position: fixed; inset: 0;
  background: rgba(0,0,0,.82);
  display: flex; align-items: center; justify-content: center;
  z-index: 1000;
}
.modal-box {
  background: #fff;
  border-radius: 18px;
  padding: 1.5rem;
  text-align: center;
  width: 320px;
  box-shadow: 0 10px 40px rgba(0,0,0,.5);
}
.modal-box h3 { font-size: 16px; margin-bottom: 1rem; color: #12122a; }
.modal-box img {
  width: 100%;
  border-radius: 10px;
  border: 3px solid #12122a;
  margin-bottom: .8rem;
}
.modal-hint { font-size: 11px; color: #999; margin-bottom: 1rem; }
.btn-close {
  background: #12122a;
  color: #fff;
  border: none;
  border-radius: 8px;
  padding: .55rem 1.5rem;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  width: 100%;
}
.btn-close:hover { background: #1f1f40; }

/* ── Transitions ── */
.pop-enter-active  { animation: popIn .4s ease; }
@keyframes popIn {
  0%   { transform: scale(.7); opacity: 0; }
  70%  { transform: scale(1.05); }
  100% { transform: scale(1);   opacity: 1; }
}

.fade-enter-active, .fade-leave-active { transition: opacity .25s; }
.fade-enter-from,  .fade-leave-to      { opacity: 0; }
</style>