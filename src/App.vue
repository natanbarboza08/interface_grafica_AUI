<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { io } from 'socket.io-client'

const socket = io('http://localhost:3000', { autoConnect: false })

const opcConectado = ref(false)
const contPecasVermelhas = ref(0)
const contPecasVerdes = ref(0)
const contPecasAmarelas = ref(0)
const contPecasTotal = ref(0)

const colors = computed(() => [
  { name: 'Verde', label: 'Verdes', amount: contPecasVerdes.value, color: '#38d69a', id: 'green' },
  { name: 'Amarelo', label: 'Amarelas', amount: contPecasAmarelas.value, color: '#ffc857', id: 'yellow' },
  { name: 'Vermelho', label: 'Vermelhas', amount: contPecasVermelhas.value, color: '#ff6584', id: 'red' },
])

const total = computed(() => contPecasTotal.value)

function porcentagem(amount) {
  return total.value > 0 ? Math.round((amount / total.value) * 100) : 0
}

function estiloAnel(amount, color) {
  const porcentagemAtual = porcentagem(amount)
  return {
    '--ring-value': `${porcentagemAtual * 3.6}deg`,
    '--ring-color': color,
  }
}

function atualizarDados(dados) {
  contPecasVermelhas.value = dados.contPecasVermelhas
  contPecasVerdes.value = dados.contPecasVerdes
  contPecasAmarelas.value = dados.contPecasAmarelas
  contPecasTotal.value = dados.contPecasTotal
}

onMounted(() => {
  socket.on('opcStatus', status => { opcConectado.value = status })
  socket.on('dadosProducao', atualizarDados)
  socket.on('disconnect', () => { opcConectado.value = false })
  socket.connect()
})

onUnmounted(() => {
  socket.off('opcStatus')
  socket.off('dadosProducao', atualizarDados)
  socket.off('disconnect')
  socket.disconnect()
})
</script>

<template>
  <div class="dashboard">
    <aside class="sidebar">
      <div class="brand" aria-label="Interface CLP">
        <span class="brand-mark">
          <svg viewBox="0 0 24 24" fill="none" aria-hidden="true">
            <path d="M4 7.5 12 3l8 4.5v9L12 21l-8-4.5v-9Z" />
            <path d="m4 7.5 8 4.5 8-4.5M12 12v9" />
          </svg>
        </span>
        <span>CLP<span class="brand-dot">/</span>CTRL</span>
      </div>

      <nav class="side-nav" aria-label="Navegação principal">
        <span class="nav-label">MONITORAMENTO</span>
        <a class="nav-item active" href="#overview" aria-current="page">
          <svg viewBox="0 0 24 24" fill="none" aria-hidden="true">
            <rect x="3" y="3" width="7" height="7" rx="1" />
            <rect x="14" y="3" width="7" height="7" rx="1" />
            <rect x="3" y="14" width="7" height="7" rx="1" />
            <rect x="14" y="14" width="7" height="7" rx="1" />
          </svg>
          Visão geral
        </a>
      </nav>

      <div class="sidebar-footer">
        <div class="system-mini">
          <span :class="{ offline: !opcConectado }"></span>
          <div>
            <small>CONTROLLER</small>
            <strong>Siemens S7-1500</strong>
          </div>
        </div>
        <div class="sidebar-line"></div>
        <p>SENAI ROBERTO MANGE<br />Campinas · SP</p>
      </div>
    </aside>

    <main class="workspace" id="overview">
      <header class="topbar">
        <div>
          <p class="breadcrumb"><span>SISTEMA</span> / LINHA 01</p>
          <h1>Centro de triagem</h1>
        </div>

        <div class="topbar-status">
          <div class="status-text">
            <span :class="{ offline: !opcConectado }" class="status-indicator"></span>
            <div>
              <small>STATUS OPC UA</small>
              <strong>{{ opcConectado ? 'Sistema online' : 'Sistema offline' }}</strong>
            </div>
          </div>
        </div>
      </header>

      <section class="command-strip" aria-label="Resumo da linha">
        <div class="command-item">
          <span class="command-icon cyan">
            <svg viewBox="0 0 24 24" fill="none" aria-hidden="true">
              <path d="M4 20V8l8-4 8 4v12M8 20v-8h8v8M10 8h4" />
            </svg>
          </span>
          <div>
            <small>Linha</small>
            <strong>Triagem — 01</strong>
          </div>
        </div>
        <div class="command-item">
          <span class="command-icon purple">
            <svg viewBox="0 0 24 24" fill="none" aria-hidden="true">
              <circle cx="12" cy="12" r="8" />
              <path d="M12 7v5l3 2" />
            </svg>
          </span>
          <div>
            <small>Atualização</small>
            <strong>500 ms</strong>
          </div>
        </div>
        <div class="command-item">
          <span class="command-icon amber">
            <svg viewBox="0 0 24 24" fill="none" aria-hidden="true">
              <path d="M4 7h16v10H4zM8 11h8M8 14h5" />
            </svg>
          </span>
          <div>
            <small>Protocolo</small>
            <strong>OPC UA</strong>
          </div>
        </div>
      </section>

      <section class="section-block" id="production" aria-labelledby="production-title">
        <div class="section-heading">
          <div>
            <p>CONTADORES EM TEMPO REAL</p>
            <h2 id="production-title">Peças identificadas</h2>
          </div>
          <div class="live-tag" :class="{ offline: !opcConectado }">
            <span></span>
            {{ opcConectado ? 'Ao vivo' : 'Offline' }}
          </div>
        </div>

        <div class="counter-grid">
          <article
            v-for="item in colors"
            :key="item.name"
            class="counter-card"
            :style="{ '--accent': item.color }"
          >
            <div class="counter-card-header">
              <div>
                <span class="color-dot"></span>
                <span>PEÇAS {{ item.label.toUpperCase() }}</span>
              </div>
              <span class="counter-code">{{ item.id.toUpperCase() }} / 0{{ colors.indexOf(item) + 1 }}</span>
            </div>

            <div class="counter-main">
              <div class="progress-ring" :style="estiloAnel(item.amount, item.color)">
                <div class="ring-center">
                  <strong>{{ String(item.amount).padStart(3, '0') }}</strong>
                  <small>{{ porcentagem(item.amount) }}%</small>
                </div>
              </div>

              <div class="counter-details">
                <small>CLASSIFICADAS</small>
                <strong>{{ item.name }}</strong>
                <div class="detail-line">
                  <span>Participação</span>
                  <b>{{ porcentagem(item.amount) }}%</b>
                </div>
              </div>
            </div>

            <div class="counter-footer">
              <span>CONTADOR</span>
              <div class="signal-bars" aria-hidden="true">
                <i></i><i></i><i></i><i></i><i></i>
              </div>
            </div>
          </article>
        </div>
      </section>

      <section class="analytics-grid">
        <article class="analytics-card">
          <div class="analytics-header">
            <div>
              <p>RESUMO DA PRODUÇÃO</p>
              <h2>Distribuição total</h2>
            </div>
            <div class="total-production">
              <strong>{{ String(total).padStart(4, '0') }}</strong>
              <span>UNIDADES</span>
            </div>
          </div>

          <div class="distribution-track" role="img" aria-label="Distribuição das peças por cor">
            <span
              v-for="item in colors"
              :key="item.name"
              :style="{ '--segment-color': item.color, flexGrow: item.amount }"
            ></span>
          </div>

          <div class="distribution-legend">
            <div v-for="item in colors" :key="item.name">
              <span class="legend-name">
                <i :style="{ backgroundColor: item.color }"></i>
                {{ item.name }}
              </span>
              <strong>{{ porcentagem(item.amount) }}%</strong>
            </div>
          </div>
        </article>

        <article class="communication-card" :class="{ offline: !opcConectado }">
          <div class="communication-top">
            <div class="communication-icon">
              <svg viewBox="0 0 24 24" fill="none" aria-hidden="true">
                <path d="M5 12h14M12 5v14" />
                <circle cx="12" cy="12" r="9" />
              </svg>
            </div>
            <span>OPC UA</span>
          </div>
          <div class="communication-state">
            <p>{{ opcConectado ? 'Comunicação ativa' : 'Sem comunicação' }}</p>
            <strong>{{ opcConectado ? 'Recebendo dados' : 'Aguardando conexão' }}</strong>
          </div>
          <div class="communication-footer">
            <span>Siemens S7-1500</span>
            <i></i>
          </div>
        </article>
      </section>
    </main>
  </div>
</template>

<style scoped>
:global(*) {
  box-sizing: border-box;
}

:global(body) {
  min-width: 320px;
  min-height: 100vh;
  margin: 0;
  background:
    radial-gradient(circle at 78% 8%, rgb(75 103 255 / 7%), transparent 22rem),
    #090c12;
  color: #edf2f7;
}

:global(button),
:global(a) {
  font: inherit;
}

.dashboard {
  display: grid;
  grid-template-columns: 220px minmax(0, 1fr);
  min-height: 100vh;
}

.sidebar {
  position: sticky;
  top: 0;
  display: flex;
  height: 100vh;
  flex-direction: column;
  padding: 28px 22px 24px;
  border-right: 1px solid #1b202c;
  background: #0b0e15;
}

.brand {
  display: flex;
  align-items: center;
  gap: 10px;
  color: #f5f7fb;
  font-size: 14px;
  font-weight: 700;
  letter-spacing: 1.2px;
}

.brand-mark {
  display: grid;
  width: 38px;
  height: 38px;
  place-items: center;
  border: 1px solid #30384a;
  border-radius: 10px;
  background: linear-gradient(145deg, #171c29, #10141d);
  color: #6f86ff;
}

.brand-mark svg {
  width: 21px;
  stroke: currentColor;
  stroke-linecap: round;
  stroke-linejoin: round;
  stroke-width: 1.5;
}

.brand-dot {
  color: #617cff;
}

.side-nav {
  display: grid;
  gap: 7px;
  margin-top: 52px;
}

.nav-label {
  margin: 0 10px 8px;
  color: #596174;
  font-size: 9px;
  font-weight: 700;
  letter-spacing: 1.4px;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: 11px;
  height: 44px;
  padding: 0 12px;
  border: 1px solid transparent;
  border-radius: 8px;
  color: #747d91;
  text-decoration: none;
  font-size: 12px;
  font-weight: 500;
}

.nav-item svg {
  width: 17px;
  stroke: currentColor;
  stroke-linecap: round;
  stroke-linejoin: round;
  stroke-width: 1.5;
}

.nav-item.active {
  border-color: #242c40;
  background: linear-gradient(90deg, rgb(97 124 255 / 15%), rgb(97 124 255 / 4%));
  color: #cbd3e8;
}

.nav-item.active svg {
  color: #7188ff;
}

.sidebar-footer {
  margin-top: auto;
}

.system-mini {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 11px 10px;
  border: 1px solid #1d2431;
  border-radius: 9px;
  background: #10141d;
}

.system-mini > span {
  width: 8px;
  height: 8px;
  flex: 0 0 auto;
  border-radius: 50%;
  background: #38d69a;
  box-shadow: 0 0 12px rgb(56 214 154 / 70%);
}

.system-mini > span.offline {
  background: #ff6584;
  box-shadow: 0 0 12px rgb(255 101 132 / 70%);
}

.system-mini div {
  display: grid;
  gap: 2px;
}

.system-mini small,
.sidebar-footer p {
  color: #626b7d;
  font-size: 8px;
  letter-spacing: 0.8px;
}

.system-mini strong {
  color: #afb7c7;
  font-size: 9px;
  font-weight: 600;
}

.sidebar-line {
  height: 1px;
  margin: 15px 2px 11px;
  background: #1b202c;
}

.sidebar-footer p {
  margin: 0;
  line-height: 1.7;
}

.workspace {
  width: 100%;
  max-width: 1280px;
  margin: 0 auto;
  padding: 30px 38px 42px;
}

.topbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  min-height: 55px;
  padding-bottom: 22px;
  border-bottom: 1px solid #1a1f29;
}

.breadcrumb {
  margin: 0 0 6px;
  color: #697286;
  font-size: 9px;
  font-weight: 700;
  letter-spacing: 1.2px;
}

.breadcrumb span {
  color: #697cff;
}

.topbar h1 {
  margin: 0;
  font-size: 25px;
  font-weight: 650;
  letter-spacing: -0.8px;
}

.topbar-status,
.status-text {
  display: flex;
  align-items: center;
}

.topbar-status {
  gap: 14px;
}

.status-text {
  gap: 9px;
  min-width: 130px;
}

.status-indicator {
  width: 9px;
  height: 9px;
  border-radius: 50%;
  background: #38d69a;
  box-shadow: 0 0 0 4px rgb(56 214 154 / 8%), 0 0 14px rgb(56 214 154 / 45%);
}

.status-indicator.offline {
  background: #ff6584;
  box-shadow: 0 0 0 4px rgb(255 101 132 / 8%), 0 0 14px rgb(255 101 132 / 45%);
}

.status-text div {
  display: grid;
  gap: 2px;
}

.status-text small {
  color: #626b7d;
  font-size: 8px;
  font-weight: 700;
  letter-spacing: 0.9px;
}

.status-text strong {
  color: #b9c1cf;
  font-size: 10px;
  font-weight: 600;
}

.command-strip {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  margin-top: 18px;
  border: 1px solid #1c222d;
  border-radius: 10px;
  background: #0e1219;
}

.command-item {
  display: flex;
  align-items: center;
  gap: 11px;
  min-height: 67px;
  padding: 10px 18px;
  border-right: 1px solid #1c222d;
}

.command-item:last-child {
  border-right: 0;
}

.command-icon {
  display: grid;
  width: 35px;
  height: 35px;
  flex: 0 0 auto;
  place-items: center;
  border: 1px solid #263044;
  border-radius: 8px;
  background: #151a24;
}

.command-icon svg {
  width: 18px;
  stroke: currentColor;
  stroke-linecap: round;
  stroke-linejoin: round;
  stroke-width: 1.5;
}

.command-icon.cyan { color: #69b8ff; }
.command-icon.purple { color: #9c8cff; }
.command-icon.amber { color: #ffc857; }

.command-item div {
  display: grid;
  gap: 3px;
}

.command-item small {
  color: #626b7d;
  font-size: 9px;
}

.command-item strong {
  color: #bdc5d3;
  font-size: 11px;
  font-weight: 600;
}

.section-block {
  margin-top: 28px;
}

.section-heading,
.analytics-header,
.counter-card-header,
.counter-footer,
.communication-top,
.communication-footer {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.section-heading {
  margin-bottom: 14px;
}

.section-heading p,
.analytics-header p {
  margin: 0 0 4px;
  color: #6976df;
  font-size: 8px;
  font-weight: 700;
  letter-spacing: 1.3px;
}

.section-heading h2,
.analytics-header h2 {
  margin: 0;
  font-size: 15px;
  font-weight: 600;
  letter-spacing: -0.2px;
}

.live-tag {
  display: inline-flex;
  align-items: center;
  gap: 7px;
  height: 28px;
  padding: 0 10px;
  border: 1px solid #1d4d40;
  border-radius: 999px;
  background: rgb(56 214 154 / 5%);
  color: #63dcb0;
  font-size: 9px;
  font-weight: 600;
}

.live-tag span {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: #38d69a;
  box-shadow: 0 0 8px rgb(56 214 154 / 70%);
}

.live-tag.offline {
  border-color: #5c2835;
  background: rgb(255 101 132 / 5%);
  color: #ff8aa0;
}

.live-tag.offline span {
  background: #ff6584;
  box-shadow: 0 0 8px rgb(255 101 132 / 70%);
}

.counter-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 14px;
}

.counter-card {
  position: relative;
  overflow: hidden;
  min-width: 0;
  padding: 16px;
  border: 1px solid #202631;
  border-radius: 11px;
  background:
    linear-gradient(145deg, rgb(255 255 255 / 1.5%), transparent 35%),
    #10151d;
}

.counter-card::after {
  position: absolute;
  right: -30px;
  bottom: -60px;
  width: 130px;
  height: 130px;
  border-radius: 50%;
  background: var(--accent);
  content: '';
  filter: blur(70px);
  opacity: 0.04;
}

.counter-card-header {
  position: relative;
  z-index: 1;
  padding-bottom: 13px;
  border-bottom: 1px solid #202630;
}

.counter-card-header div {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #aeb6c5;
  font-size: 9px;
  font-weight: 700;
  letter-spacing: 0.7px;
}

.color-dot {
  width: 7px;
  height: 7px;
  border-radius: 2px;
  background: var(--accent);
  box-shadow: 0 0 9px color-mix(in srgb, var(--accent) 55%, transparent);
}

.counter-code {
  color: #51596a;
  font-size: 8px;
  font-weight: 700;
  letter-spacing: 0.8px;
}

.counter-main {
  position: relative;
  z-index: 1;
  display: grid;
  grid-template-columns: 122px minmax(0, 1fr);
  align-items: center;
  gap: 14px;
  min-height: 170px;
}

.progress-ring {
  display: grid;
  width: 116px;
  height: 116px;
  place-items: center;
  border-radius: 50%;
  background:
    radial-gradient(circle, #10151d 58%, transparent 59%),
    conic-gradient(var(--ring-color) 0 var(--ring-value), #242b37 var(--ring-value) 360deg);
  box-shadow:
    0 0 0 1px rgb(255 255 255 / 2%),
    inset 0 0 25px rgb(255 255 255 / 2%);
}

.ring-center {
  display: grid;
  width: 88px;
  height: 88px;
  place-content: center;
  text-align: center;
  border: 1px solid #242b37;
  border-radius: 50%;
  background: #10151d;
}

.ring-center strong {
  font-size: 26px;
  font-weight: 570;
  letter-spacing: -1.3px;
  line-height: 1.1;
}

.ring-center small {
  margin-top: 4px;
  color: var(--accent);
  font-size: 8px;
  font-weight: 700;
}

.counter-details {
  min-width: 0;
}

.counter-details > small {
  color: #596174;
  font-size: 8px;
  font-weight: 700;
  letter-spacing: 0.8px;
}

.counter-details > strong {
  display: block;
  margin: 5px 0 14px;
  color: #dce2ec;
  font-size: 14px;
  font-weight: 600;
}

.detail-line {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  padding-top: 9px;
  border-top: 1px solid #202630;
  color: #697286;
  font-size: 8px;
}

.detail-line b {
  color: var(--accent);
  font-size: 9px;
}

.counter-footer {
  position: relative;
  z-index: 1;
  height: 23px;
  border-top: 1px solid #202630;
  color: #50586a;
  font-size: 7px;
  font-weight: 700;
  letter-spacing: 0.9px;
}

.signal-bars {
  display: flex;
  align-items: flex-end;
  gap: 2px;
  height: 11px;
}

.signal-bars i {
  width: 3px;
  border-radius: 1px;
  background: var(--accent);
  opacity: 0.75;
}

.signal-bars i:nth-child(1) { height: 3px; }
.signal-bars i:nth-child(2) { height: 5px; }
.signal-bars i:nth-child(3) { height: 7px; }
.signal-bars i:nth-child(4) { height: 9px; }
.signal-bars i:nth-child(5) { height: 11px; }

.analytics-grid {
  display: grid;
  grid-template-columns: minmax(0, 1fr) 250px;
  gap: 14px;
  margin-top: 14px;
}

.analytics-card,
.communication-card {
  min-height: 138px;
  border: 1px solid #202631;
  border-radius: 11px;
  background: #10151d;
}

.analytics-card {
  padding: 17px;
}

.total-production {
  display: grid;
  text-align: right;
}

.total-production strong {
  font-size: 23px;
  font-weight: 550;
  letter-spacing: -1px;
  line-height: 1;
}

.total-production span {
  margin-top: 4px;
  color: #5d6678;
  font-size: 7px;
  font-weight: 700;
  letter-spacing: 0.8px;
}

.distribution-track {
  display: flex;
  overflow: hidden;
  gap: 3px;
  height: 8px;
  margin-top: 19px;
  border-radius: 99px;
  background: #202630;
}

.distribution-track span {
  min-width: 0;
  border-radius: 99px;
  background: var(--segment-color);
  opacity: 0.9;
}

.distribution-legend {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  margin-top: 13px;
}

.distribution-legend div {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 12px;
  border-right: 1px solid #202630;
  color: #7d8697;
  font-size: 8px;
}

.distribution-legend div:first-child {
  padding-left: 0;
}

.distribution-legend div:last-child {
  border-right: 0;
  padding-right: 0;
}

.distribution-legend strong {
  color: #c7cfda;
  font-size: 9px;
}

.legend-name {
  display: flex;
  align-items: center;
  gap: 6px;
}

.legend-name i {
  width: 6px;
  height: 6px;
  border-radius: 50%;
}

.communication-card {
  display: flex;
  flex-direction: column;
  padding: 16px;
  border-color: #1d483c;
  background:
    radial-gradient(circle at 100% 100%, rgb(56 214 154 / 7%), transparent 45%),
    #10151d;
}

.communication-card.offline {
  border-color: #56303a;
  background:
    radial-gradient(circle at 100% 100%, rgb(255 101 132 / 7%), transparent 45%),
    #10151d;
}

.communication-top span {
  color: #596174;
  font-size: 8px;
  font-weight: 700;
  letter-spacing: 0.8px;
}

.communication-icon {
  display: grid;
  width: 32px;
  height: 32px;
  place-items: center;
  border: 1px solid #285344;
  border-radius: 8px;
  background: rgb(56 214 154 / 7%);
  color: #38d69a;
}

.communication-card.offline .communication-icon {
  border-color: #59313b;
  background: rgb(255 101 132 / 7%);
  color: #ff6584;
}

.communication-icon svg {
  width: 18px;
  stroke: currentColor;
  stroke-linecap: round;
  stroke-linejoin: round;
  stroke-width: 1.4;
}

.communication-state {
  margin-top: 13px;
}

.communication-state p {
  margin: 0;
  color: #38d69a;
  font-size: 12px;
  font-weight: 600;
}

.communication-card.offline .communication-state p {
  color: #ff6584;
}

.communication-state strong {
  display: block;
  margin-top: 3px;
  color: #737d8f;
  font-size: 9px;
  font-weight: 500;
}

.communication-footer {
  margin-top: auto;
  color: #596174;
  font-size: 8px;
}

.communication-footer i {
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: #38d69a;
  box-shadow: 0 0 8px rgb(56 214 154 / 70%);
}

.communication-card.offline .communication-footer i {
  background: #ff6584;
  box-shadow: 0 0 8px rgb(255 101 132 / 70%);
}

@media (max-width: 1050px) {
  .dashboard {
    grid-template-columns: 180px minmax(0, 1fr);
  }

  .workspace {
    padding-inline: 24px;
  }

  .counter-main {
    grid-template-columns: 108px minmax(0, 1fr);
  }

  .progress-ring {
    width: 102px;
    height: 102px;
  }

  .ring-center {
    width: 78px;
    height: 78px;
  }
}

@media (max-width: 860px) {
  .dashboard {
    grid-template-columns: 72px minmax(0, 1fr);
  }

  .sidebar {
    align-items: center;
    padding-inline: 12px;
  }

  .brand {
    justify-content: center;
  }

  .brand > span:last-child,
  .nav-label,
  .nav-item:not(.active),
  .nav-item.active:not(.active),
  .nav-item {
    font-size: 0;
  }

  .nav-item {
    width: 44px;
    justify-content: center;
    padding: 0;
  }

  .nav-item svg {
    width: 18px;
  }

  .sidebar-footer {
    display: none;
  }

  .counter-grid {
    grid-template-columns: 1fr;
  }

  .counter-main {
    grid-template-columns: 125px minmax(0, 1fr);
  }

  .progress-ring {
    width: 116px;
    height: 116px;
  }

  .ring-center {
    width: 88px;
    height: 88px;
  }

  .analytics-grid {
    grid-template-columns: 1fr;
  }

  .communication-card {
    min-height: 150px;
  }
}

@media (max-width: 600px) {
  .dashboard {
    display: block;
  }

  .sidebar {
    position: static;
    width: 100%;
    height: 64px;
    flex-direction: row;
    justify-content: space-between;
    padding: 12px 16px;
  }

  .brand > span:last-child {
    display: inline;
    font-size: 12px;
  }

  .brand-mark {
    width: 34px;
    height: 34px;
  }

  .side-nav {
    display: flex;
    margin: 0;
  }

  .side-nav .nav-item:not(.active) {
    display: none;
  }

  .nav-item.active {
    width: 38px;
    height: 38px;
  }

  .workspace {
    padding: 20px 14px 30px;
  }

  .topbar {
    align-items: flex-start;
  }

  .topbar h1 {
    font-size: 21px;
  }

  .status-text {
    display: none;
  }

  .command-strip {
    grid-template-columns: 1fr;
  }

  .command-item {
    border-right: 0;
    border-bottom: 1px solid #1c222d;
  }

  .command-item:last-child {
    border-bottom: 0;
  }

  .counter-main {
    grid-template-columns: 120px minmax(0, 1fr);
  }

  .analytics-grid {
    grid-template-columns: 1fr;
  }

  .distribution-legend {
    grid-template-columns: 1fr;
    gap: 8px;
  }

  .distribution-legend div {
    padding: 0;
    border-right: 0;
  }
}
</style>
