<template>
  <div class="bg-wrapper">
    <div
  class="card"
  :class="{
    'ring-active': mode === 'countdown' && hasStarted,
    'ring-paused': mode === 'countdown' && hasStarted && !running && remaining > 0
  }"
  :style="{ '--progress': progressPercent }"
>
      <div class="status">{{ statusText }}</div>

      <div v-if="!hasStarted" class="presets">
        <div class="preset-field">
          <input
            type="number"
            min="0"
            max="23"
            v-model.number="presetHours"
            class="preset-input"
          />
          <label>hrs</label>
        </div>
        <div class="preset-field">
          <input
            type="number"
            min="0"
            max="59"
            v-model.number="presetMinutes"
            class="preset-input"
          />
          <label>min</label>
        </div>
        <div class="preset-field">
          <input
            type="number"
            min="0"
            max="59"
            v-model.number="presetSeconds"
            class="preset-input"
          />
          <label>sec</label>
        </div>
      </div>

      <div class="display">{{ formatted }}</div>

      <div class="buttons">
        <button
          class="btn-start"
          :disabled="running"
          @click="start"
        >{{ hasStarted ? 'Resume' : 'Start' }}</button>
        <button
          class="btn-pause"
          :disabled="!running"
          @click="pause"
        >Pause</button>
        <button
          class="btn-stop"
          :disabled="!running && !hasStarted"
          @click="stop"
        >Stop</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      presetHours: 0,
      presetMinutes: 0,
      presetSeconds: 0,

      mode: 'stopwatch',
      elapsed: 0,
      remaining: 0,
      totalMs: 0,
      running: false,
      hasStarted: false,
      lastTick: null,
      rafId: null,
    };
  },
  computed: {
    statusText() {
      if (this.running) return 'Running';
      if (this.hasStarted) {
        if (this.mode === 'countdown' && this.remaining <= 0) return 'Done';
        return 'Paused';
      }
      return 'Ready';
    },
    formatted() {
      if (this.mode === 'countdown') {
        const ms = Math.max(0, this.remaining);
        const totalSeconds = Math.ceil(ms / 1000);
        const hours = Math.floor(totalSeconds / 3600);
        const minutes = Math.floor((totalSeconds % 3600) / 60);
        const seconds = totalSeconds % 60;
        const pad = (n) => String(n).padStart(2, '0');
        return hours > 0
          ? `${pad(hours)}:${pad(minutes)}:${pad(seconds)}`
          : `${pad(minutes)}:${pad(seconds)}`;
      }

      const ms = this.elapsed;
      const totalTenths = Math.floor(ms / 100);
      const tenths = totalTenths % 10;
      const totalSeconds = Math.floor(ms / 1000);
      const seconds = totalSeconds % 60;
      const minutes = Math.floor(totalSeconds / 60);
      const pad = (n) => String(n).padStart(2, '0');
      return `${pad(minutes)}:${pad(seconds)}.${tenths}`;
    },
    progressPercent() {
  if (this.mode !== 'countdown' || this.totalMs <= 0) return 0;
  return Math.min(100, Math.max(0, (Math.max(0, this.remaining) / this.totalMs) * 100));
},
  },
  beforeUnmount() {
    if (this.rafId) cancelAnimationFrame(this.rafId);
  },
  methods: {
    presetTotalMs() {
      const h = Number(this.presetHours) || 0;
      const m = Number(this.presetMinutes) || 0;
      const s = Number(this.presetSeconds) || 0;
      return (h * 3600 + m * 60 + s) * 1000;
    },
    tick(now) {
      if (!this.running) return;
      const delta = now - this.lastTick;
      this.lastTick = now;

      if (this.mode === 'countdown') {
        this.remaining -= delta;
        if (this.remaining <= 0) {
          this.remaining = 0;
          this.running = false;
          return;
        }
      } else {
        this.elapsed += delta;
      }
      this.rafId = requestAnimationFrame(this.tick);
    },
    start() {
      if (!this.hasStarted) {
        const presetMs = this.presetTotalMs();
        if (presetMs > 0) {
          this.mode = 'countdown';
          this.remaining = presetMs;
          this.totalMs = presetMs;
        } else {
          this.mode = 'stopwatch';
          this.elapsed = 0;
        }
      }
      this.running = true;
      this.hasStarted = true;
      this.lastTick = performance.now();
      this.rafId = requestAnimationFrame(this.tick);
    },
    pause() {
      this.running = false;
      if (this.rafId) cancelAnimationFrame(this.rafId);
    },
    stop() {
      this.running = false;
      this.hasStarted = false;
      if (this.rafId) cancelAnimationFrame(this.rafId);
      this.elapsed = 0;
      this.remaining = 0;
      this.totalMs = 0;
    },
  },
};
</script>

<style scoped>
.bg-wrapper {
  min-height: 100vh;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background-image: url('/img_lights.jpg');
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  padding: 1.5rem;
}

.card {
  position: relative;
  background: rgba(20, 20, 25, 0.28);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  border-radius: 50%;
  width: 420px;
  height: 420px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 2rem;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.35);
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
  color: #fff;
  border: 1px solid rgba(255, 255, 255, 0.15);
}

.card::before {
  content: '';
  position: absolute;
  inset: -6px;
  border-radius: 50%;
  padding: 4px;
  background: conic-gradient(
    rgba(255, 255, 255, 0.12) 0%,
    rgba(255, 255, 255, 0.12) 100%
  );
  -webkit-mask: linear-gradient(#000 0 0) content-box, linear-gradient(#000 0 0);
  -webkit-mask-composite: xor;
  mask-composite: exclude;
  pointer-events: none;
}

.card.ring-active::before {
  background: conic-gradient(
    #22d3ee calc(var(--progress) * 1%),
    rgba(255, 255, 255, 0.12) 0
  );
  transition: background 0.15s linear;
}

.card.ring-paused::before {
  background: conic-gradient(
    #faf9f8 calc(var(--progress) * 1%),
    rgba(255, 255, 255, 0.12) 0
  );
}

.status {
  color: rgba(255, 255, 255, 0.75);
  font-size: 0.85rem;
  text-transform: uppercase;
  letter-spacing: 1px;
  margin-bottom: 1rem;
}
.presets {
  display: flex;
  gap: 0.6rem;
  justify-content: center;
  margin-bottom: 1.5rem;
}
.preset-field {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.preset-input {
  width: 70px;
  text-align: center;
  font-size: 1.3rem;
  padding: 0.6rem;
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.3);
  background: rgba(255, 255, 255, 0.1);
  color: #fff;
}


.preset-field label {
  font-size: 1rem;
  color: rgba(255, 255, 255, 0.65);
  margin-top: 0.25rem;
  text-transform: uppercase;
}


.display {
  font-size: 2.5rem;
  font-variant-numeric: tabular-nums;
  font-weight: 600;
  margin: 0.25rem 0 1.5rem;
  letter-spacing: 1px;
}
.buttons {
  display: flex;
  gap: 0.6rem;
  justify-content: center;
  flex-wrap: wrap;
}
button {
  border: none;
  border-radius: 10px;
  padding: 0.65rem 1.1rem;
  font-size: 0.95rem;
  font-weight: 600;
  cursor: pointer;
  min-width: 80px;
  transition: opacity 0.15s;
}
button:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}
button:not(:disabled):hover {
  opacity: 1;
}
.btn-start,
.btn-pause,
.btn-stop {
  background: #ffffff;
  color: #1a1a1a;
  border: 2px solid #ffffff;
  border-radius: 999px;
  transition: background 0.2s ease, color 0.2s ease;
}
.btn-start:hover:not(:disabled),
.btn-pause:hover:not(:disabled),
.btn-stop:hover:not(:disabled) {
  background: transparent;
  color: #ffffff;
}
</style>