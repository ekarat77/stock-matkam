<!DOCTYPE html>
<html lang="th">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>ใกล้กัน 💕</title>
<link href="https://fonts.googleapis.com/css2?family=Prompt:wght@300;400;600;700&family=Inter:wght@300;400;500&display=swap" rel="stylesheet">
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>

<style>
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
:root {
  --bg: #0A0A0F; --card: #13131F; --card2: #1A1A2E;
  --me: #6B9DFF; --partner: #FF6B9D;
  --text: #E8E8F0; --muted: #6B6B8A; --border: rgba(255,255,255,0.08);
  --font-display: 'Prompt', sans-serif; --font-body: 'Inter', sans-serif;
}
html, body { height: 100%; background: var(--bg); color: var(--text); font-family: var(--font-body); overflow: hidden; }
#map { position: fixed; inset: 0; z-index: 1; filter: brightness(0.85) saturate(1.1); }

.top-panel {
  position: fixed; top: 0; left: 0; right: 0; z-index: 10;
  padding: 12px 16px 0; display: flex; flex-direction: column; gap: 10px;
  pointer-events: none;
}
.header-bar {
  display: flex; align-items: center; justify-content: space-between;
  background: rgba(10,10,15,0.82); backdrop-filter: blur(20px);
  border: 1px solid var(--border); border-radius: 20px; padding: 10px 16px;
  pointer-events: all;
}
.app-title {
  font-family: var(--font-display); font-size: 16px; font-weight: 700;
  background: linear-gradient(135deg, var(--me), var(--partner));
  -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text;
}
.header-right { display: flex; align-items: center; gap: 10px; }
.live-badge { display: flex; align-items: center; gap: 5px; font-size: 11px; color: #4ade80; font-weight: 500; }
.live-dot { width: 7px; height: 7px; border-radius: 50%; background: #4ade80; animation: pulse-green 1.5s ease-in-out infinite; }
@keyframes pulse-green { 0%,100%{opacity:1;transform:scale(1)} 50%{opacity:0.5;transform:scale(0.7)} }
.ws-status { font-size: 10px; padding: 3px 8px; border-radius: 999px; font-weight: 500; }
.ws-status.connecting { background: rgba(250,204,21,0.15); color: #facc15; }
.ws-status.connected { background: rgba(74,222,128,0.15); color: #4ade80; }
.ws-status.disconnected { background: rgba(248,113,113,0.15); color: #f87171; }

.profiles-row { display: flex; align-items: center; gap: 8px; pointer-events: all; }
.profile-card {
  flex: 1; background: rgba(10,10,15,0.82); backdrop-filter: blur(20px);
  border: 1px solid var(--border); border-radius: 18px; padding: 12px 14px;
  display: flex; align-items: center; gap: 10px; cursor: pointer; transition: border-color 0.2s;
}
.profile-card.me { border-bottom: 2px solid var(--me); }
.profile-card.partner { border-bottom: 2px solid var(--partner); }
.profile-card:hover { border-color: rgba(255,255,255,0.2); }
.avatar-wrap { position: relative; flex-shrink: 0; width: 46px; height: 46px; }
.avatar-emoji {
  width: 46px; height: 46px; border-radius: 50%;
  display: flex; align-items: center; justify-content: center;
  font-size: 22px; border: 2px solid transparent;
}
.me .avatar-emoji { border-color: var(--me); background: rgba(107,157,255,0.15); }
.partner .avatar-emoji { border-color: var(--partner); background: rgba(255,107,157,0.15); }
.status-ring {
  position: absolute; bottom: 0; right: 0;
  width: 13px; height: 13px; border-radius: 50%;
  background: #6b6b8a; border: 2px solid #0A0A0F; transition: background 0.3s;
}
.status-ring.online { background: #4ade80; }
.profile-info { flex: 1; min-width: 0; }
.profile-name { font-family: var(--font-display); font-size: 13px; font-weight: 600; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.profile-sub { font-size: 10px; color: var(--muted); margin-top: 2px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.profile-label { font-size: 10px; font-weight: 500; padding: 2px 7px; border-radius: 999px; }
.me .profile-label { background: rgba(107,157,255,0.2); color: var(--me); }
.partner .profile-label { background: rgba(255,107,157,0.2); color: var(--partner); }

.dist-card {
  background: rgba(10,10,15,0.82); backdrop-filter: blur(20px);
  border: 1px solid var(--border); border-radius: 18px; padding: 10px 16px;
  display: flex; align-items: center; justify-content: center; gap: 6px;
  pointer-events: all;
}
.heart-icon { font-size: 14px; animation: beat 1.2s ease-in-out infinite; }
@keyframes beat { 0%,100%{transform:scale(1)} 50%{transform:scale(1.25)} }
.dist-value {
  font-family: var(--font-display); font-size: 22px; font-weight: 700;
  background: linear-gradient(135deg, var(--me), var(--partner));
  -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text;
}
.dist-unit { font-size: 12px; color: var(--muted); }
.dist-label { font-size: 11px; color: var(--muted); margin-left: 4px; }

.bottom-bar {
  position: fixed; bottom: 28px; left: 50%; transform: translateX(-50%);
  z-index: 10; display: flex; gap: 10px; pointer-events: all;
}
.btn {
  height: 44px; border-radius: 999px; border: 1px solid var(--border);
  background: rgba(10,10,15,0.85); backdrop-filter: blur(20px);
  color: var(--text); cursor: pointer; font-family: var(--font-display); font-size: 13px; font-weight: 500;
  display: flex; align-items: center; gap: 7px; padding: 0 18px;
  transition: background 0.2s, border-color 0.2s, transform 0.1s;
}
.btn:hover { border-color: rgba(255,255,255,0.2); }
.btn:active { transform: scale(0.96); }
.btn.primary { background: linear-gradient(135deg, var(--me), var(--partner)); border: none; color: #fff; }

.heart-send-btn {
  position: fixed; right: 20px; bottom: 100px; z-index: 10;
  width: 52px; height: 52px; border-radius: 50%;
  background: rgba(255,107,157,0.2); border: 1px solid rgba(255,107,157,0.4);
  backdrop-filter: blur(20px);
  display: flex; align-items: center; justify-content: center; font-size: 22px;
  cursor: pointer; transition: transform 0.15s, background 0.2s;
}
.heart-send-btn:hover { background: rgba(255,107,157,0.35); }
.heart-send-btn:active { transform: scale(0.88); }

.floating-heart {
  position: fixed; z-index: 200; pointer-events: none;
  font-size: 28px; animation: floatUp 2.5s ease-out forwards;
}
@keyframes floatUp {
  0% { transform: translateY(0) scale(1); opacity: 1; }
  100% { transform: translateY(-180px) scale(0.4); opacity: 0; }
}

.toast {
  position: fixed; top: 50%; left: 50%; transform: translate(-50%,-50%) scale(0.8);
  background: rgba(10,10,15,0.95); backdrop-filter: blur(20px);
  border: 1px solid var(--border); border-radius: 16px;
  padding: 14px 24px; font-family: var(--font-display); font-size: 15px;
  z-index: 300; opacity: 0; pointer-events: none;
  transition: opacity 0.2s, transform 0.2s; text-align: center;
}
.toast.show { opacity: 1; transform: translate(-50%,-50%) scale(1); }

.modal-overlay {
  display: none; position: fixed; inset: 0; z-index: 100;
  background: rgba(0,0,0,0.7); backdrop-filter: blur(8px);
  align-items: flex-end; justify-content: center;
}
.modal-overlay.open { display: flex; }
.modal {
  width: 100%; max-width: 420px; background: var(--card);
  border: 1px solid var(--border); border-radius: 28px 28px 0 0;
  padding: 24px 24px 40px;
  transform: translateY(100%);
  transition: transform 0.35s cubic-bezier(0.32, 0.72, 0, 1);
}
.modal-overlay.open .modal { transform: translateY(0); }
.modal-handle { width: 36px; height: 4px; border-radius: 2px; background: var(--border); margin: 0 auto 20px; }
.modal-title { font-family: var(--font-display); font-size: 18px; font-weight: 700; margin-bottom: 20px; }

.avatar-preview-wrap { display: flex; justify-content: center; margin-bottom: 20px; }
.avatar-preview {
  width: 80px; height: 80px; border-radius: 50%; font-size: 36px;
  display: flex; align-items: center; justify-content: center; border: 3px solid var(--me);
  background: rgba(107,157,255,0.1);
}
#modal-avatar-preview.partner-modal { border-color: var(--partner); background: rgba(255,107,157,0.1); }

.emoji-grid { display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 16px; }
.emoji-btn {
  width: 40px; height: 40px; border-radius: 12px;
  background: var(--card2); border: 1px solid var(--border);
  font-size: 20px; cursor: pointer; display: flex; align-items: center; justify-content: center;
  transition: background 0.15s, border-color 0.15s;
}
.emoji-btn:hover, .emoji-btn.active { background: rgba(107,157,255,0.2); border-color: var(--me); }

.form-group { margin-bottom: 14px; }
.form-label { font-size: 12px; color: var(--muted); margin-bottom: 6px; display: block; }
.form-input {
  width: 100%; background: var(--card2); border: 1px solid var(--border); border-radius: 12px;
  padding: 10px 14px; color: var(--text); font-family: var(--font-body); font-size: 14px;
  outline: none; transition: border-color 0.2s;
}
.form-input:focus { border-color: var(--me); }
.modal-actions { display: flex; gap: 10px; margin-top: 20px; }
.modal-actions .btn { flex: 1; justify-content: center; height: 48px; font-size: 14px; }

.share-code {
  background: var(--card2); border: 1px solid var(--border);
  border-radius: 14px; padding: 20px; text-align: center; margin: 8px 0 16px;
}
.share-code-val {
  font-family: monospace; font-size: 32px; font-weight: 700; letter-spacing: 8px;
  background: linear-gradient(135deg, var(--me), var(--partner));
  -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text;
}
.share-desc { font-size: 11px; color: var(--muted); margin-top: 6px; }
.connect-input-row { display: flex; gap: 8px; }
.connect-input-row .form-input { flex: 1; }
.connect-input-row .btn { height: 42px; padding: 0 16px; white-space: nowrap; }
.connect-status-box {
  margin-top: 12px; padding: 12px;
  background: rgba(107,157,255,0.08); border-radius: 12px;
  font-size: 12px; color: var(--muted); text-align: center;
}

.nearby-alert {
  position: fixed; top: 50%; left: 50%; transform: translate(-50%,-50%) scale(0);
  background: rgba(10,10,15,0.95); backdrop-filter: blur(24px);
  border: 1px solid rgba(74,222,128,0.3); border-radius: 24px;
  padding: 28px 36px; text-align: center; z-index: 300;
  transition: transform 0.4s cubic-bezier(0.34,1.56,0.64,1), opacity 0.3s;
  opacity: 0;
}
.nearby-alert.show { transform: translate(-50%,-50%) scale(1); opacity: 1; }
.nearby-alert-emoji { font-size: 48px; display: block; margin-bottom: 8px; }
.nearby-alert-title { font-family: var(--font-display); font-size: 20px; font-weight: 700; margin-bottom: 4px; }
.nearby-alert-sub { font-size: 13px; color: var(--muted); }
</style>
</head>
<body>

<div id="map"></div>

<div class="top-panel">
  <div class="header-bar">
    <span class="app-title">ใกล้กัน 💕</span>
    <div class="header-right">
      <div class="ws-status connecting" id="ws-status">กำลังเชื่อมต่อ...</div>
      <div class="live-badge"><div class="live-dot"></div> LIVE</div>
    </div>
  </div>

  <div class="profiles-row">
    <div class="profile-card me" onclick="openEditModal('me')">
      <div class="avatar-wrap">
        <div class="avatar-emoji me" id="me-avatar">😊</div>
        <div class="status-ring online" id="me-ring"></div>
      </div>
      <div class="profile-info">
        <div class="profile-name" id="me-name">ฉัน</div>
        <div class="profile-sub" id="me-coords">กำลังหาตำแหน่ง...</div>
      </div>
      <div class="profile-label me">ฉัน</div>
    </div>

    <div class="profile-card partner" onclick="openEditModal('partner')">
      <div class="avatar-wrap">
        <div class="avatar-emoji partner" id="partner-avatar">🥰</div>
        <div class="status-ring" id="partner-ring"></div>
      </div>
      <div class="profile-info">
        <div class="profile-name" id="partner-name">แฟน</div>
        <div class="profile-sub" id="partner-status">รอการเชื่อมต่อ...</div>
      </div>
      <div class="profile-label partner">แฟน</div>
    </div>
  </div>

  <div class="dist-card">
    <span class="heart-icon">💗</span>
    <span class="dist-value" id="dist-value">--</span>
    <span class="dist-unit" id="dist-unit">กม.</span>
    <span class="dist-label">ห่างกัน</span>
  </div>
</div>

<div class="heart-send-btn" onclick="sendHeart()">💌</div>

<div class="bottom-bar">
  <button class="btn" onclick="centerMap()">🎯 ดูทั้งคู่</button>
  <button class="btn primary" onclick="openShareModal()">🔗 เชื่อมต่อแฟน</button>
  <button class="btn" onclick="locateMe()">📍 ฉัน</button>
</div>

<div class="toast" id="toast"></div>

<div class="nearby-alert" id="nearby-alert">
  <span class="nearby-alert-emoji">🫶</span>
  <div class="nearby-alert-title">อยู่ใกล้กันแล้ว!</div>
  <div class="nearby-alert-sub">คุณกับแฟนห่างกันไม่ถึง 100 เมตร ❤️</div>
  <button class="btn primary" style="margin-top:16px;width:100%;justify-content:center" onclick="closeNearbyAlert()">เย้! 🎉</button>
</div>

<!-- EDIT MODAL -->
<div class="modal-overlay" id="edit-modal">
  <div class="modal">
    <div class="modal-handle"></div>
    <div class="modal-title" id="edit-modal-title">แก้ไขโปรไฟล์</div>
    <div class="avatar-preview-wrap">
      <div class="avatar-preview" id="modal-avatar-preview">😊</div>
    </div>
    <div class="form-group">
      <label class="form-label">เลือกอีโมจิโปรไฟล์</label>
      <div class="emoji-grid" id="emoji-grid"></div>
    </div>
    <div class="form-group">
      <label class="form-label">ชื่อ</label>
      <input class="form-input" id="edit-name-input" type="text" placeholder="ใส่ชื่อ...">
    </div>
    <div class="modal-actions">
      <button class="btn" onclick="closeModal('edit-modal')">ยกเลิก</button>
      <button class="btn primary" onclick="saveEdit()">💾 บันทึก</button>
    </div>
  </div>
</div>

<!-- SHARE MODAL -->
<div class="modal-overlay" id="share-modal">
  <div class="modal">
    <div class="modal-handle"></div>
    <div class="modal-title">เชื่อมต่อกับแฟน 💕</div>
    <div class="form-group">
      <label class="form-label">รหัสห้องของคุณ (แชร์ให้แฟน)</label>
      <div class="share-code">
        <div class="share-code-val" id="my-room-code">------</div>
        <div class="share-desc">ให้แฟนใส่รหัสนี้แล้วกดเชื่อมต่อ</div>
      </div>
    </div>
    <div class="form-group">
      <label class="form-label">หรือใส่รหัสห้องของแฟน</label>
      <div class="connect-input-row">
        <input class="form-input" id="partner-code-input" type="text" maxlength="6"
          placeholder="XXXXXX" style="letter-spacing:4px;text-transform:uppercase;font-family:monospace;font-size:18px">
        <button class="btn primary" onclick="connectToRoom()">เชื่อมต่อ</button>
      </div>
    </div>
    <div class="connect-status-box" id="connect-status">
      💡 ทั้งสองฝ่ายต้องใส่รหัสห้องเดียวกัน
    </div>
    <div class="modal-actions">
      <button class="btn" onclick="closeModal('share-modal')">ปิด</button>
      <button class="btn primary" onclick="copyCode()">📋 คัดลอกรหัส</button>
    </div>
  </div>
</div>

<script>
// ── SUPABASE CONFIG ───────────────────────────────────────
const SUPABASE_URL = 'https://ccrvtcmcayvjxzglrsae.supabase.co';
const SUPABASE_ANON_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImNjcnZ0Y21jYXl2anh6Z2xyc2FlIiwicm9sZSI6ImFub24iLCJpYXQiOjE3ODIyODY0MDMsImV4cCI6MjA5Nzg2MjQwM30.P2aeSJni7XDgEgq1ITcrDaUs7az14uxA-6h6lV_jBm0';
const { createClient } = supabase;
const sb = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);

// ── GLOBALS ───────────────────────────────────────────────
const EMOJIS = ['😊','🥰','😎','🤩','😍','🥳','🫶','💪','🌸','🌙','⭐','🦋','🐼','🦊','🐱','🐶','🍓','🍀','🎸','🎨'];
let map, meMarker, partnerMarker, line;
let myPos = null, partnerPos = null;
let editTarget = null, selectedEmoji = null;
let myRoomCode = null;
let myPeerId = localStorage.getItem('cpl-peerid') || Math.random().toString(36).substr(2,10);
localStorage.setItem('cpl-peerid', myPeerId);
let channel = null;
let partnerLastSeen = null;
let nearbyAlerted = false;

let userData = { me: { name: 'ฉัน', emoji: '😊' }, partner: { name: 'แฟน', emoji: '🥰' } };
try { const s = localStorage.getItem('cpl-data'); if(s) userData = JSON.parse(s); } catch(e){}
applyUserData();

// ── MAP ───────────────────────────────────────────────────
map = L.map('map', { zoomControl: false, attributionControl: false });
L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}', { maxZoom: 20 }).addTo(map);
L.control.attribution({ position:'bottomleft', prefix:false })
  .addAttribution('<span style="opacity:0.3;font-size:9px">Esri World Imagery</span>').addTo(map);
map.setView([13.7563, 100.5018], 13);

// ── MARKER ────────────────────────────────────────────────
function createMarkerIcon(emoji, color) {
  return L.divIcon({
    className: '',
    html: `<div style="position:relative;width:52px;height:60px">
      <div style="width:52px;height:52px;border-radius:50%;background:${color}22;border:3px solid ${color};
        display:flex;align-items:center;justify-content:center;font-size:24px;
        box-shadow:0 0 0 6px ${color}33,0 4px 20px ${color}55;">${emoji}</div>
      <div style="position:absolute;bottom:0;left:50%;transform:translateX(-50%);width:0;height:0;
        border-left:8px solid transparent;border-right:8px solid transparent;border-top:10px solid ${color};"></div>
    </div>`,
    iconSize:[52,60], iconAnchor:[26,60]
  });
}

// ── GEOLOCATION ───────────────────────────────────────────
function startTracking() {
  if (!navigator.geolocation) { useDemoPos(); return; }
  navigator.geolocation.watchPosition(pos => {
    myPos = { lat: pos.coords.latitude, lng: pos.coords.longitude };
    updateMyMarker(); broadcastPos();
  }, () => useDemoPos(), { enableHighAccuracy: true, maximumAge: 4000 });
}

function useDemoPos() {
  myPos = { lat: 13.7563+(Math.random()-0.5)*0.008, lng: 100.5018+(Math.random()-0.5)*0.008 };
  updateMyMarker();
  setInterval(() => {
    myPos.lat += (Math.random()-0.5)*0.0004;
    myPos.lng += (Math.random()-0.5)*0.0004;
    updateMyMarker(); broadcastPos();
  }, 5000);
}

function updateMyMarker() {
  const ll = [myPos.lat, myPos.lng];
  if (!meMarker) {
    meMarker = L.marker(ll, { icon: createMarkerIcon(userData.me.emoji,'#6B9DFF'), zIndexOffset:1000 }).addTo(map);
    map.setView(ll, 15);
  } else meMarker.setLatLng(ll);
  document.getElementById('me-coords').textContent = `${myPos.lat.toFixed(5)}, ${myPos.lng.toFixed(5)}`;
  updateLine(); updateDistance();
}

function setPartnerPos(lat, lng) {
  partnerPos = { lat, lng };
  if (!partnerMarker) {
    partnerMarker = L.marker([lat,lng], { icon: createMarkerIcon(userData.partner.emoji,'#FF6B9D') }).addTo(map);
  } else partnerMarker.setLatLng([lat,lng]);
  updateLine(); updateDistance(); checkNearby();
}

function updateLine() {
  if (!myPos||!partnerPos) return;
  const pts = [[myPos.lat,myPos.lng],[partnerPos.lat,partnerPos.lng]];
  if (!line) line = L.polyline(pts,{color:'white',weight:2,opacity:0.45,dashArray:'6,8'}).addTo(map);
  else line.setLatLngs(pts);
}

function haversine(a,b) {
  const R=6371,dLat=rad(b.lat-a.lat),dLng=rad(b.lng-a.lng);
  const x=Math.sin(dLat/2)**2+Math.cos(rad(a.lat))*Math.cos(rad(b.lat))*Math.sin(dLng/2)**2;
  return R*2*Math.atan2(Math.sqrt(x),Math.sqrt(1-x));
}
const rad = d => d*Math.PI/180;

function updateDistance() {
  if (!myPos||!partnerPos) return;
  const d = haversine(myPos,partnerPos);
  const el = document.getElementById('dist-value');
  const unit = document.getElementById('dist-unit');
  if (d<1){el.textContent=Math.round(d*1000);unit.textContent='เมตร';}
  else{el.textContent=d.toFixed(2);unit.textContent='กม.';}
}

function checkNearby() {
  if (!myPos||!partnerPos||nearbyAlerted) return;
  if (haversine(myPos,partnerPos)<0.1) {
    nearbyAlerted=true;
    document.getElementById('nearby-alert').classList.add('show');
  }
}
function closeNearbyAlert() {
  document.getElementById('nearby-alert').classList.remove('show');
  setTimeout(()=>nearbyAlerted=false, 30000);
}

function centerMap() {
  if (!myPos) return;
  if (partnerPos) map.fitBounds([[myPos.lat,myPos.lng],[partnerPos.lat,partnerPos.lng]],{padding:[80,80]});
  else map.setView([myPos.lat,myPos.lng],15);
}
function locateMe() { if(myPos) map.setView([myPos.lat,myPos.lng],16); }

// ── HEART ─────────────────────────────────────────────────
function sendHeart() {
  spawnHeart(window.innerWidth/2, window.innerHeight*0.7, true);
  if (channel) channel.send({ type:'broadcast', event:'heart', payload:{ from: myPeerId } });
  showToast('💕 ส่งหัวใจไปแล้ว!');
}
function spawnHeart(x,y,isLocal) {
  const hearts=['❤️','💕','💖','💗','💓','🩷'];
  for (let i=0;i<(isLocal?3:5);i++) {
    const h=document.createElement('div');
    h.className='floating-heart';
    h.textContent=hearts[Math.floor(Math.random()*hearts.length)];
    const ox=(Math.random()-0.5)*80;
    h.style.cssText=`left:${x+ox}px;top:${y}px;animation-delay:${i*0.15}s;animation-duration:${1.8+Math.random()*0.8}s`;
    document.body.appendChild(h);
    setTimeout(()=>h.remove(),3000);
  }
}

let toastTimer;
function showToast(msg) {
  const t=document.getElementById('toast');
  t.textContent=msg; t.classList.add('show');
  clearTimeout(toastTimer);
  toastTimer=setTimeout(()=>t.classList.remove('show'),2200);
}

// ── SUPABASE REALTIME ─────────────────────────────────────
function setStatus(s) {
  const el=document.getElementById('ws-status');
  el.className='ws-status '+s;
  el.textContent=s==='connected'?'เชื่อมต่อแล้ว':s==='connecting'?'กำลังเชื่อมต่อ...':'ขาดการเชื่อมต่อ';
}

function joinRoom(code) {
  if (channel) { sb.removeChannel(channel); channel=null; }
  setStatus('connecting');

  channel = sb.channel('room:'+code, {
    config: { broadcast: { self: false } }
  });

  channel
    .on('broadcast', { event: 'pos' }, ({ payload }) => {
      if (payload.from === myPeerId) return;
      setPartnerPos(payload.lat, payload.lng);
      partnerLastSeen = new Date();
      document.getElementById('partner-ring').classList.add('online');
      document.getElementById('partner-status').textContent =
        `${payload.lat.toFixed(5)}, ${payload.lng.toFixed(5)}`;
      if (payload.name) {
        userData.partner.name = payload.name;
        userData.partner.emoji = payload.emoji || userData.partner.emoji;
        applyUserData();
        if (partnerMarker) partnerMarker.setIcon(createMarkerIcon(userData.partner.emoji,'#FF6B9D'));
      }
    })
    .on('broadcast', { event: 'heart' }, ({ payload }) => {
      if (payload.from === myPeerId) return;
      spawnHeart(window.innerWidth/2, window.innerHeight/2, false);
      showToast(`💗 ${userData.partner.name} ส่งหัวใจมา!`);
    })
    .on('broadcast', { event: 'join' }, ({ payload }) => {
      if (payload.from === myPeerId) return;
      showToast(`💕 ${payload.name||'แฟน'} เข้าร่วมแล้ว!`);
      setTimeout(broadcastPos, 500);
    })
    .on('broadcast', { event: 'leave' }, ({ payload }) => {
      if (payload.from === myPeerId) return;
      document.getElementById('partner-ring').classList.remove('online');
      document.getElementById('partner-status').textContent='ออฟไลน์';
      showToast(`😢 ${userData.partner.name} ออกไปแล้ว`);
    })
    .subscribe(status => {
      if (status === 'SUBSCRIBED') {
        setStatus('connected');
        channel.send({ type:'broadcast', event:'join', payload:{ from:myPeerId, name:userData.me.name } });
        broadcastPos();
      } else if (status === 'CLOSED' || status === 'CHANNEL_ERROR') {
        setStatus('disconnected');
      }
    });
}

function broadcastPos() {
  if (!channel||!myPos) return;
  channel.send({ type:'broadcast', event:'pos', payload:{
    from:myPeerId, lat:myPos.lat, lng:myPos.lng,
    name:userData.me.name, emoji:userData.me.emoji
  }});
}

function leaveRoom() {
  if (channel) {
    channel.send({ type:'broadcast', event:'leave', payload:{ from:myPeerId } });
    sb.removeChannel(channel);
  }
}

// ── SHARE MODAL ───────────────────────────────────────────
function genCode() { return Math.random().toString(36).substr(2,6).toUpperCase(); }

function openShareModal() {
  if (!myRoomCode) myRoomCode = genCode();
  document.getElementById('my-room-code').textContent = myRoomCode;
  document.getElementById('share-modal').classList.add('open');
  joinRoom(myRoomCode);
}

function connectToRoom() {
  const code = document.getElementById('partner-code-input').value.trim().toUpperCase();
  if (code.length < 4) { showToast('กรุณาใส่รหัสให้ครบ'); return; }
  myRoomCode = code;
  document.getElementById('my-room-code').textContent = myRoomCode;
  document.getElementById('connect-status').textContent = '🔄 กำลังเชื่อมต่อ...';
  joinRoom(myRoomCode);
  setTimeout(() => {
    document.getElementById('connect-status').innerHTML =
      '✅ <strong style="color:#4ade80">เข้าห้องแล้ว!</strong> รอแฟนส่งตำแหน่ง...';
  }, 1500);
}

function copyCode() {
  navigator.clipboard?.writeText(myRoomCode).then(()=>showToast('📋 คัดลอกรหัสแล้ว!'));
}

// ── EDIT PROFILE ──────────────────────────────────────────
const emojiGrid = document.getElementById('emoji-grid');
EMOJIS.forEach(e => {
  const btn = document.createElement('button');
  btn.className='emoji-btn'; btn.textContent=e;
  btn.onclick = () => {
    document.querySelectorAll('.emoji-btn').forEach(b=>b.classList.remove('active'));
    btn.classList.add('active');
    selectedEmoji=e;
    document.getElementById('modal-avatar-preview').textContent=e;
  };
  emojiGrid.appendChild(btn);
});

function openEditModal(target) {
  editTarget=target;
  const d=userData[target];
  selectedEmoji=d.emoji;
  document.getElementById('edit-modal-title').textContent = target==='me'?'✏️ โปรไฟล์ของฉัน':'✏️ โปรไฟล์แฟน';
  document.getElementById('edit-name-input').value=d.name;
  document.getElementById('modal-avatar-preview').textContent=d.emoji;
  document.getElementById('modal-avatar-preview').className='avatar-preview'+(target==='partner'?' partner-modal':'');
  document.querySelectorAll('.emoji-btn').forEach(b=>b.classList.toggle('active',b.textContent===d.emoji));
  document.getElementById('edit-modal').classList.add('open');
}

function saveEdit() {
  if (!editTarget) return;
  const name=document.getElementById('edit-name-input').value.trim()||userData[editTarget].name;
  userData[editTarget].name=name;
  userData[editTarget].emoji=selectedEmoji||userData[editTarget].emoji;
  localStorage.setItem('cpl-data',JSON.stringify(userData));
  applyUserData();
  if (editTarget==='me'&&meMarker) meMarker.setIcon(createMarkerIcon(userData.me.emoji,'#6B9DFF'));
  if (editTarget==='partner'&&partnerMarker) partnerMarker.setIcon(createMarkerIcon(userData.partner.emoji,'#FF6B9D'));
  if (editTarget==='me') broadcastPos();
  closeModal('edit-modal');
  showToast('✅ บันทึกแล้ว!');
}

function applyUserData() {
  document.getElementById('me-name').textContent=userData.me.name;
  document.getElementById('partner-name').textContent=userData.partner.name;
  document.getElementById('me-avatar').textContent=userData.me.emoji;
  document.getElementById('partner-avatar').textContent=userData.partner.emoji;
}

function closeModal(id) { document.getElementById(id).classList.remove('open'); }
document.querySelectorAll('.modal-overlay').forEach(el=>{
  el.addEventListener('click',e=>{ if(e.target===el) el.classList.remove('open'); });
});

// last seen ticker
setInterval(()=>{
  if (!partnerLastSeen) return;
  const secs=Math.floor((Date.now()-partnerLastSeen)/1000);
  if (secs>20) {
    document.getElementById('partner-ring').classList.remove('online');
    document.getElementById('partner-status').textContent =
      secs<60?`เมื่อ ${secs} วิที่แล้ว`:`เมื่อ ${Math.floor(secs/60)} นาทีที่แล้ว`;
  }
},5000);

window.addEventListener('beforeunload', leaveRoom);

// ── INIT ──────────────────────────────────────────────────
startTracking();
setStatus('connected'); // Supabase connects on demand
</script>
</body>
</html>
