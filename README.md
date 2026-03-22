# wep<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>3D Portfolio World</title>
<link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700;900&family=Rajdhani:wght@300;400;600&family=Share+Tech+Mono&display=swap" rel="stylesheet">
 
<!-- Firebase -->
<script type="module">
  import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js";
  import { getAuth, createUserWithEmailAndPassword, signInWithEmailAndPassword, signOut, onAuthStateChanged }
    from "https://www.gstatic.com/firebasejs/10.12.0/firebase-auth.js";
  import { getFirestore, doc, getDoc, setDoc, updateDoc, arrayUnion }
    from "https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore.js";
 
  // ── PASTE YOUR FIREBASE CONFIG HERE ──────────────────────────────────
  const firebaseConfig = {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_PROJECT.firebaseapp.com",
    projectId: "YOUR_PROJECT_ID",
    storageBucket: "YOUR_PROJECT.appspot.com",

 messagingSenderId: "YOUR_SENDER_ID",
    appId: "YOUR_APP_ID"
  };
  // ─────────────────────────────────────────────────────────────────────
 
  const app  = initializeApp(firebaseConfig);
  const auth = getAuth(app);
  const db   = getFirestore(app);
 
  window.__FB = { auth, db, createUserWithEmailAndPassword, signInWithEmailAndPassword,
                  signOut, onAuthStateChanged, doc, getDoc, setDoc, updateDoc, arrayUnion };
</script>
 
<style>
/* ═══════════════ CSS VARIABLES ═══════════════ */
:root {
  --neon-blue:   #00f3ff;
  --neon-pink:   #ff00e5;
  --neon-purple: #9d00ff;
  --neon-green:  #00ff88;
  --bg-dark:     #020408;
  --bg-panel:    rgba(0,8,20,.75);
  --glass:       rgba(0,243,255,.06);
  --glass-border:rgba(0,243,255,.22);
  --text-main:   #e8f4ff;
  --text-dim:    #7090a8;
  --font-head:   'Orbitron', monospace;
  --font-body:   'Rajdhani', sans-serif;
  --font-mono:   'Share Tech Mono', monospace;
  --glow-blue:   0 0 8px #00f3ff, 0 0 24px rgba(0,243,255,.4);
  --glow-pink:   0 0 8px #ff00e5, 0 0 24px rgba(255,0,229,.4);
  --glow-purple: 0 0 8px #9d00ff, 0 0 24px rgba(157,0,255,.4);
}
.light-theme {
  --bg-dark:     #eef4ff;
  --bg-panel:    rgba(220,235,255,.85);
  --glass:       rgba(0,100,200,.06);
  --glass-border:rgba(0,100,200,.25);
  --text-main:   #0a1628;
  --text-dim:    #4a6080;
  --neon-blue:   #0055cc;
  --neon-pink:   #cc0088;
  --neon-purple: #6600cc;
  --neon-green:  #008844;
  --glow-blue:   0 0 8px rgba(0,85,204,.5);
  --glow-pink:   0 0 8px rgba(204,0,136,.5);
}
 
/* ═══════════════ RESET / BASE ═══════════════ */
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0}
html,body{width:100%;height:100%;overflow:hidden;background:var(--bg-dark);color:var(--text-main);font-family:var(--font-body)}
canvas{display:block;position:fixed;top:0;left:0;z-index:0;pointer-events:none}
button{cursor:pointer;font-family:var(--font-head);border:none;outline:none}
input{font-family:var(--font-body);outline:none}
a{color:var(--neon-blue);text-decoration:none}
 
/* ═══════════════ SCROLLBAR ═══════════════ */
::-webkit-scrollbar{width:4px}
::-webkit-scrollbar-track{background:transparent}
::-webkit-scrollbar-thumb{background:var(--neon-blue);border-radius:2px}
 
/* ═══════════════ GLOBAL ANIMATIONS ═══════════════ */
@keyframes float      {0%,100%{transform:translateY(0)}50%{transform:translateY(-12px)}}
@keyframes pulse-glow {0%,100%{opacity:.7}50%{opacity:1}}
@keyframes scan-line  {0%{top:0}100%{top:100%}}
@keyframes spin       {to{transform:rotate(360deg)}}
@keyframes fadeIn     {from{opacity:0;transform:translateY(20px)}to{opacity:1;transform:translateY(0)}}
@keyframes slideIn    {from{opacity:0;transform:translateX(-40px)}to{opacity:1;transform:translateX(0)}}
@keyframes glitch1    {0%,100%{clip-path:inset(0 0 98% 0)}20%{clip-path:inset(20% 0 60% 0)}40%{clip-path:inset(50% 0 30% 0)}60%{clip-path:inset(10% 0 80% 0)}80%{clip-path:inset(70% 0 10% 0)}}
@keyframes glitch2    {0%,100%{clip-path:inset(98% 0 0 0)}20%{clip-path:inset(60% 0 20% 0)}40%{clip-path:inset(30% 0 50% 0)}60%{clip-path:inset(80% 0 10% 0)}80%{clip-path:inset(10% 0 70% 0)}}
@keyframes marquee    {from{transform:translateX(0)}to{transform:translateX(-50%)}}
@keyframes blink      {0%,100%{opacity:1}50%{opacity:0}}
@keyframes neon-border{0%,100%{box-shadow:0 0 5px var(--neon-blue),0 0 10px var(--neon-blue),inset 0 0 5px rgba(0,243,255,.1)}50%{box-shadow:0 0 10px var(--neon-blue),0 0 30px var(--neon-blue),0 0 60px rgba(0,243,255,.3),inset 0 0 10px rgba(0,243,255,.15)}}
 
/* ═══════════════ SCANLINES OVERLAY ═══════════════ */
#scanlines{position:fixed;top:0;left:0;width:100%;height:100%;z-index:1;pointer-events:none;background:repeating-linear-gradient(0deg,transparent,transparent 2px,rgba(0,0,0,.08) 2px,rgba(0,0,0,.08) 4px)}
#scanlines::after{content:'';position:absolute;top:-100%;left:0;width:100%;height:40px;background:linear-gradient(transparent,rgba(0,243,255,.04),transparent);animation:scan-line 8s linear infinite}
 
/* ═══════════════ TOP HUD BAR ═══════════════ */
#hud{display:none;position:fixed;top:0;left:0;right:0;z-index:50;height:44px;background:linear-gradient(90deg,rgba(0,243,255,.06),rgba(157,0,255,.06));border-bottom:1px solid var(--glass-border);backdrop-filter:blur(10px);align-items:center;padding:0 16px;gap:12px}
#hud.visible{display:flex}
.hud-logo{font-family:var(--font-head);font-size:13px;font-weight:900;color:var(--neon-blue);text-shadow:var(--glow-blue);letter-spacing:3px;white-space:nowrap}
.hud-marquee{flex:1;overflow:hidden;mask-image:linear-gradient(90deg,transparent,#000 60px,#000 calc(100% - 60px),transparent)}
.hud-marquee-inner{display:inline-flex;gap:40px;animation:marquee 18s linear infinite;white-space:nowrap;font-family:var(--font-mono);font-size:10px;color:var(--text-dim)}
.hud-marquee-inner span{color:var(--neon-green)}
.hud-actions{display:flex;gap:8px;align-items:center}
.hud-btn{background:var(--glass);border:1px solid var(--glass-border);color:var(--text-main);font-size:10px;padding:5px 10px;border-radius:4px;letter-spacing:1px;transition:all .2s;white-space:nowrap;font-family:var(--font-head)}
.hud-btn:hover{background:rgba(0,243,255,.15);border-color:var(--neon-blue);color:var(--neon-blue);box-shadow:var(--glow-blue)}
.hud-btn.danger:hover{background:rgba(255,0,100,.15);border-color:#ff0064;color:#ff0064}
#user-email-display{font-family:var(--font-mono);font-size:10px;color:var(--neon-green);background:rgba(0,255,136,.08);border:1px solid rgba(0,255,136,.2);padding:4px 10px;border-radius:4px}
 
/* ═══════════════ LOGIN SCREEN ═══════════════ */
#login-screen{position:fixed;inset:0;z-index:100;display:flex;flex-direction:column;align-items:center;justify-content:center;background:radial-gradient(ellipse at 60% 40%,rgba(157,0,255,.18) 0%,transparent 60%),radial-gradient(ellipse at 30% 70%,rgba(0,243,255,.12) 0%,transparent 55%),var(--bg-dark)}
.login-bg-grid{position:absolute;inset:0;background-image:linear-gradient(rgba(0,243,255,.04) 1px,transparent 1px),linear-gradient(90deg,rgba(0,243,255,.04) 1px,transparent 1px);background-size:40px 40px;pointer-events:none}
.login-hero{text-align:center;margin-bottom:40px;position:relative;z-index:1}
.login-hero-title{font-family:var(--font-head);font-size:clamp(22px,4vw,42px);font-weight:900;letter-spacing:4px;line-height:1.2;color:var(--text-main);position:relative}
.login-hero-title .glitch{position:relative;display:inline-block}
.login-hero-title .glitch::before,.login-hero-title .glitch::after{content:attr(data-text);position:absolute;inset:0;opacity:.7}
.login-hero-title .glitch::before{color:var(--neon-pink);animation:glitch1 4s infinite;left:2px;text-shadow:none}
.login-hero-title .glitch::after{color:var(--neon-blue);animation:glitch2 4s infinite;left:-2px;text-shadow:none}
.login-hero-sub{font-family:var(--font-mono);font-size:12px;color:var(--neon-blue);letter-spacing:6px;margin-top:10px;opacity:.8}
.login-hero-sub span{animation:blink 1.2s step-end infinite}
 
.login-card{position:relative;z-index:1;width:clamp(300px,90vw,420px);background:var(--bg-panel);border:1px solid var(--glass-border);border-radius:12px;padding:32px;backdrop-filter:blur(20px);animation:neon-border 4s ease-in-out infinite}
.login-card::before{content:'';position:absolute;top:-1px;left:20%;right:20%;height:2px;background:linear-gradient(90deg,transparent,var(--neon-blue),var(--neon-purple),transparent);border-radius:2px}
 
.tab-row{display:flex;gap:0;margin-bottom:24px;border:1px solid var(--glass-border);border-radius:8px;overflow:hidden}
.tab-btn{flex:1;padding:10px;background:transparent;color:var(--text-dim);font-size:12px;font-weight:700;letter-spacing:2px;transition:all .2s;font-family:var(--font-head)}
.tab-btn.active{background:linear-gradient(135deg,rgba(0,243,255,.15),rgba(157,0,255,.1));color:var(--neon-blue);text-shadow:var(--glow-blue)}
 
.form-group{margin-bottom:16px}
.form-label{display:block;font-family:var(--font-mono);font-size:10px;color:var(--neon-blue);letter-spacing:2px;margin-bottom:6px}
.form-input{width:100%;background:rgba(0,243,255,.04);border:1px solid var(--glass-border);border-radius:6px;padding:11px 14px;color:var(--text-main);font-size:14px;transition:all .25s;font-family:var(--font-body)}
.form-input:focus{border-color:var(--neon-blue);background:rgba(0,243,255,.08);box-shadow:0 0 0 2px rgba(0,243,255,.12)}
.form-input::placeholder{color:var(--text-dim)}
.form-error{font-size:11px;color:#ff4466;margin-top:4px;font-family:var(--font-mono);display:none}
 
.cta-btn{width:100%;padding:13px;border-radius:8px;font-size:13px;font-weight:700;letter-spacing:3px;transition:all .3s;position:relative;overflow:hidden;margin-top:8px}
.cta-btn::before{content:'';position:absolute;inset:0;background:linear-gradient(135deg,var(--neon-blue),var(--neon-purple));opacity:0;transition:opacity .3s}
.cta-btn:hover::before{opacity:1}
.cta-btn-primary{background:linear-gradient(135deg,rgba(0,243,255,.2),rgba(157,0,255,.2));border:1px solid var(--neon-blue);color:var(--neon-blue)}
.cta-btn-primary:hover{color:#fff;border-color:var(--neon-purple);box-shadow:var(--glow-blue)}
.cta-btn span{position:relative;z-index:1}
 
.divider{text-align:center;font-family:var(--font-mono);font-size:10px;color:var(--text-dim);margin:12px 0;position:relative}
.divider::before,.divider::after{content:'';position:absolute;top:50%;width:40%;height:1px;background:var(--glass-border)}
.divider::before{left:0}.divider::after{right:0}
 
/* ═══════════════ LOADING TRANSITION ═══════════════ */
#loader{position:fixed;inset:0;z-index:200;background:var(--bg-dark);display:flex;flex-direction:column;align-items:center;justify-content:center;transition:opacity .8s}
#loader.hidden{opacity:0;pointer-events:none}
.loader-ring{width:80px;height:80px;border:2px solid var(--glass-border);border-top-color:var(--neon-blue);border-right-color:var(--neon-purple);border-radius:50%;animation:spin .9s linear infinite}
.loader-text{font-family:var(--font-mono);font-size:11px;color:var(--neon-blue);letter-spacing:4px;margin-top:20px;animation:pulse-glow 1.5s ease-in-out infinite}
.loader-progress{width:200px;height:2px;background:var(--glass-border);border-radius:2px;margin-top:16px;overflow:hidden}
.loader-bar{height:100%;background:linear-gradient(90deg,var(--neon-blue),var(--neon-purple));border-radius:2px;animation:loader-fill 1.8s ease forwards}
@keyframes loader-fill{from{width:0}to{width:100%}}
 
/* ═══════════════ PANELS (MODALS) ═══════════════ */
.panel-overlay{position:fixed;inset:0;z-index:80;background:rgba(0,4,12,.85);backdrop-filter:blur(4px);display:flex;align-items:center;justify-content:center;padding:16px;opacity:0;pointer-events:none;transition:opacity .3s}
.panel-overlay.open{opacity:1;pointer-events:all}
.panel-box{width:100%;max-width:600px;max-height:90vh;overflow-y:auto;background:var(--bg-panel);border:1px solid var(--glass-border);border-radius:14px;padding:28px;backdrop-filter:blur(24px);animation:fadeIn .35s ease;position:relative}
.panel-box::before{content:'';position:absolute;top:-1px;left:10%;right:10%;height:2px;background:linear-gradient(90deg,transparent,var(--neon-pink),var(--neon-purple),transparent)}
.panel-title{font-family:var(--font-head);font-size:16px;font-weight:700;color:var(--neon-blue);letter-spacing:2px;margin-bottom:20px;display:flex;align-items:center;gap:10px}
.panel-title .icon{font-size:20px}
.close-btn{position:absolute;top:16px;right:16px;background:rgba(255,0,100,.1);border:1px solid rgba(255,0,100,.3);color:#ff4466;width:30px;height:30px;border-radius:6px;font-size:14px;transition:all .2s}
.close-btn:hover{background:rgba(255,0,100,.25);box-shadow:0 0 10px rgba(255,0,100,.3)}
.p-row{display:grid;grid-template-columns:1fr 1fr;gap:12px}
@media(max-width:500px){.p-row{grid-template-columns:1fr}}
.p-full{grid-column:1/-1}
.save-btn{background:linear-gradient(135deg,rgba(0,255,136,.15),rgba(0,243,255,.1));border:1px solid var(--neon-green);color:var(--neon-green);padding:11px 28px;border-radius:8px;font-size:12px;font-weight:700;letter-spacing:2px;transition:all .3s;margin-top:8px}
.save-btn:hover{background:linear-gradient(135deg,rgba(0,255,136,.3),rgba(0,243,255,.2));box-shadow:0 0 20px rgba(0,255,136,.3)}
 
/* ═══════════════ SKILLS TAGS ═══════════════ */
.skill-tags{display:flex;flex-wrap:wrap;gap:8px;margin-top:8px}
.skill-tag{background:rgba(0,243,255,.08);border:1px solid rgba(0,243,255,.25);color:var(--neon-blue);padding:4px 12px;border-radius:20px;font-size:11px;font-family:var(--font-mono);letter-spacing:1px;display:flex;align-items:center;gap:6px;cursor:pointer;transition:all .2s}
.skill-tag:hover{background:rgba(255,0,100,.12);border-color:#ff4466;color:#ff4466}
.skill-tag .rm{font-size:13px;line-height:1}
 
/* ═══════════════ PROJECTS GRID ═══════════════ */
.projects-grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(200px,1fr));gap:14px;margin-top:8px}
.project-card{background:rgba(0,243,255,.04);border:1px solid var(--glass-border);border-radius:10px;padding:14px;transition:all .25s;cursor:pointer}
.project-card:hover{border-color:var(--neon-pink);background:rgba(255,0,229,.06);box-shadow:0 0 20px rgba(255,0,229,.15);transform:translateY(-3px)}
.project-card h4{font-family:var(--font-head);font-size:12px;color:var(--text-main);margin-bottom:6px}
.project-card p{font-size:12px;color:var(--text-dim);line-height:1.5}
.project-card .tech-list{display:flex;flex-wrap:wrap;gap:4px;margin-top:8px}
.project-card .tech-badge{background:rgba(157,0,255,.12);border:1px solid rgba(157,0,255,.3);color:var(--neon-purple);padding:2px 8px;border-radius:10px;font-size:10px;font-family:var(--font-mono)}
.project-card .p-links{display:flex;gap:8px;margin-top:10px}
.project-card .p-link{font-size:10px;font-family:var(--font-mono);color:var(--neon-blue);border:1px solid var(--glass-border);padding:3px 8px;border-radius:4px;transition:all .2s}
.project-card .p-link:hover{border-color:var(--neon-blue);background:rgba(0,243,255,.1)}
 
/* ═══════════════ HUD MINI-NAV ═══════════════ */
#mini-nav{position:fixed;right:16px;top:50%;transform:translateY(-50%);z-index:50;display:none;flex-direction:column;gap:8px}
#mini-nav.visible{display:flex}
.nav-dot{width:10px;height:10px;border-radius:50%;background:var(--glass-border);border:1px solid var(--glass-border);transition:all .3s;cursor:pointer;position:relative}
.nav-dot::after{content:attr(data-label);position:absolute;right:18px;top:50%;transform:translateY(-50%);background:var(--bg-panel);border:1px solid var(--glass-border);color:var(--text-dim);font-family:var(--font-mono);font-size:9px;padding:3px 8px;border-radius:4px;white-space:nowrap;opacity:0;pointer-events:none;transition:all .2s;letter-spacing:1px}
.nav-dot:hover{background:var(--neon-blue);box-shadow:var(--glow-blue)}
.nav-dot:hover::after{opacity:1}
.nav-dot.active{background:var(--neon-blue);box-shadow:var(--glow-blue)}
 
/* ═══════════════ TOAST ═══════════════ */
#toast{position:fixed;bottom:24px;left:50%;transform:translateX(-50%) translateY(80px);z-index:300;background:var(--bg-panel);border:1px solid var(--glass-border);border-radius:8px;padding:10px 20px;font-family:var(--font-mono);font-size:12px;color:var(--text-main);transition:transform .4s cubic-bezier(.34,1.56,.64,1);white-space:nowrap;backdrop-filter:blur(16px)}
#toast.show{transform:translateX(-50%) translateY(0)}
#toast.success{border-color:var(--neon-green);color:var(--neon-green)}
#toast.error{border-color:#ff4466;color:#ff4466}
 
/* ═══════════════ FLOATING 3D INFO LABELS ═══════════════ */
#labels-container{position:fixed;top:44px;left:0;right:0;bottom:0;z-index:10;pointer-events:none}
.world-label{position:absolute;transform:translate(-50%,-50%);pointer-events:all;cursor:pointer}
.world-label-inner{background:var(--bg-panel);border:1px solid var(--glass-border);border-radius:8px;padding:8px 16px;font-family:var(--font-head);font-size:11px;letter-spacing:2px;white-space:nowrap;backdrop-filter:blur(8px);transition:all .25s;animation:float 4s ease-in-out infinite}
.world-label:hover .world-label-inner{border-color:var(--neon-blue);color:var(--neon-blue);box-shadow:var(--glow-blue)}
.wl-profile{color:var(--neon-blue)}
.wl-projects{color:var(--neon-pink)}
.wl-skills{color:var(--neon-purple)}
.wl-contact{color:var(--neon-green)}
 
/* ═══════════════ PROFILE PANEL ═══════════════ */
.profile-avatar{width:80px;height:80px;border-radius:50%;background:linear-gradient(135deg,var(--neon-blue),var(--neon-purple));display:flex;align-items:center;justify-content:center;font-size:32px;margin:0 auto 16px;border:2px solid var(--neon-blue);box-shadow:var(--glow-blue);position:relative;overflow:hidden}
.profile-avatar::after{content:'';position:absolute;inset:-4px;border-radius:50%;border:1px solid var(--neon-purple);animation:spin 6s linear infinite}
.profile-name{text-align:center;font-family:var(--font-head);font-size:20px;font-weight:700;color:var(--text-main);letter-spacing:2px}
.profile-bio{text-align:center;font-size:13px;color:var(--text-dim);margin:8px 0 16px;line-height:1.6}
.profile-stats{display:flex;gap:0;border:1px solid var(--glass-border);border-radius:8px;overflow:hidden;margin-bottom:16px}
.stat-item{flex:1;text-align:center;padding:12px 8px;border-right:1px solid var(--glass-border)}
.stat-item:last-child{border-right:none}
.stat-num{font-family:var(--font-head);font-size:18px;font-weight:700;color:var(--neon-blue)}
.stat-lbl{font-family:var(--font-mono);font-size:9px;color:var(--text-dim);letter-spacing:2px;margin-top:2px}
 
/* ═══════════════ CONTACT FORM ═══════════════ */
.contact-links{display:flex;flex-wrap:wrap;gap:10px;margin-top:12px}
.contact-link{background:var(--glass);border:1px solid var(--glass-border);color:var(--text-dim);padding:8px 16px;border-radius:6px;font-size:12px;font-family:var(--font-mono);letter-spacing:1px;transition:all .2s;display:flex;align-items:center;gap:6px}
.contact-link:hover{border-color:var(--neon-blue);color:var(--neon-blue);background:rgba(0,243,255,.08)}
 
/* ═══════════════ MUSIC BTN ═══════════════ */
#music-indicator{position:fixed;bottom:20px;right:16px;z-index:60;display:none;align-items:center;gap:6px;background:var(--bg-panel);border:1px solid var(--glass-border);border-radius:20px;padding:6px 14px;backdrop-filter:blur(8px)}
#music-indicator.visible{display:flex}
.music-bars{display:flex;gap:2px;align-items:flex-end;height:14px}
.music-bar{width:3px;background:var(--neon-blue);border-radius:2px;animation:music-dance .8s ease-in-out infinite}
.music-bar:nth-child(2){animation-delay:.1s;height:60%}
.music-bar:nth-child(3){animation-delay:.2s;height:90%}
.music-bar:nth-child(4){animation-delay:.15s;height:50%}
.music-bar:nth-child(5){animation-delay:.05s;height:70%}
@keyframes music-dance{0%,100%{transform:scaleY(1)}50%{transform:scaleY(.3)}}
.music-bars.paused .music-bar{animation-play-state:paused}
#music-indicator span{font-family:var(--font-mono);font-size:9px;color:var(--text-dim);letter-spacing:1px}
 
/* ═══════════════ CROSSHAIR / CURSOR ═══════════════ */
#world-crosshair{position:fixed;top:50%;left:50%;transform:translate(-50%,-50%);pointer-events:none;z-index:5;opacity:0;transition:opacity .3s;display:none}
#world-crosshair.visible{display:block}
.xh-ring{width:20px;height:20px;border:1px solid rgba(0,243,255,.4);border-radius:50%;position:absolute;top:50%;left:50%;transform:translate(-50%,-50%)}
.xh-dot{width:4px;height:4px;background:var(--neon-blue);border-radius:50%;position:absolute;top:50%;left:50%;transform:translate(-50%,-50%)}
.xh-line-h,.xh-line-v{position:absolute;background:rgba(0,243,255,.3)}
.xh-line-h{width:14px;height:1px;top:50%;left:50%;transform:translate(-50%,-50%)}
.xh-line-v{width:1px;height:14px;top:50%;left:50%;transform:translate(-50%,-50%)}
 
/* ═══════════════ PDF EXPORT ═══════════════ */
#pdf-preview{display:none}
</style>
</head>
<body>
<!-- Scanlines -->
<div id="scanlines"></div>
 
<!-- Three.js Canvas -->
<canvas id="three-canvas"></canvas>
 
<!-- LOADER -->
<div id="loader">
  <div class="loader-ring"></div>
  <div class="loader-text">INITIALIZING PORTFOLIO WORLD</div>
  <div class="loader-progress"><div class="loader-bar"></div></div>
</div>
 
<!-- ══════════ LOGIN SCREEN ══════════ -->
<div id="login-screen">
  <div class="login-bg-grid"></div>
  <div class="login-hero">
    <div class="login-hero-title">
      <span class="glitch" data-text="ENTER YOUR">ENTER YOUR</span><br>
      <span class="glitch" data-text="3D PORTFOLIO WORLD">3D PORTFOLIO WORLD</span>
    </div>
    <div class="login-hero-sub">// CYBERPUNK METAVERSE EXPERIENCE <span>_</span></div>
  </div>
 
  <div class="login-card">
    <div class="tab-row">
      <button class="tab-btn active" id="tab-login" onclick="switchTab('login')">LOGIN</button>
      <button class="tab-btn" id="tab-signup" onclick="switchTab('signup')">SIGN UP</button>
    </div>
 
    <!-- LOGIN FORM -->
    <div id="form-login">
      <div class="form-group">
        <label class="form-label">// EMAIL ADDRESS</label>
        <input class="form-input" id="login-email" type="email" placeholder="user@cyberworld.io">
        <div class="form-error" id="le-err"></div>
      </div>
      <div class="form-group">
        <label class="form-label">// PASSWORD</label>
        <input class="form-input" id="login-pass" type="password" placeholder="••••••••">
        <div class="form-error" id="lp-err"></div>
      </div>
      <button class="cta-btn cta-btn-primary" onclick="doLogin()"><span>⚡ ENTER WORLD</span></button>
      <div class="divider">OR</div>
      <button class="cta-btn" style="background:transparent;border:1px solid var(--glass-border);color:var(--text-dim)" onclick="demoLogin()"><span>🎮 DEMO MODE (NO ACCOUNT)</span></button>
    </div>
 
    <!-- SIGNUP FORM -->
    <div id="form-signup" style="display:none">
      <div class="form-group">
        <label class="form-label">// YOUR NAME</label>
        <input class="form-input" id="su-name" type="text" placeholder="Alex Cyberdev">
      </div>
      <div class="form-group">
        <label class="form-label">// EMAIL ADDRESS</label>
        <input class="form-input" id="su-email" type="email" placeholder="user@cyberworld.io">
        <div class="form-error" id="se-err"></div>
      </div>
      <div class="form-group">
        <label class="form-label">// PASSWORD (MIN 6 CHARS)</label>
        <input class="form-input" id="su-pass" type="password" placeholder="••••••••">
        <div class="form-error" id="sp-err"></div>
      </div>
      <button class="cta-btn cta-btn-primary" onclick="doSignup()"><span>🚀 CREATE ACCOUNT</span></button>
    </div>
  </div>
</div>
 
<!-- ══════════ HUD ══════════ -->
<div id="hud">
  <div class="hud-logo">◈ 3D WORLD</div>
  <div class="hud-marquee">
    <div class="hud-marquee-inner">
      <span>SYS</span> PORTFOLIO ACTIVE &nbsp;•&nbsp; <span>NET</span> CONNECTED &nbsp;•&nbsp;
      <span>VER</span> 2.077 &nbsp;•&nbsp; <span>3D</span> ENGINE ONLINE &nbsp;•&nbsp;
      <span>FIREBASE</span> SYNC READY &nbsp;•&nbsp; <span>SYS</span> PORTFOLIO ACTIVE &nbsp;•&nbsp;
      <span>NET</span> CONNECTED &nbsp;•&nbsp; <span>VER</span> 2.077 &nbsp;•&nbsp;
      <span>3D</span> ENGINE ONLINE &nbsp;•&nbsp; <span>FIREBASE</span> SYNC READY &nbsp;•&nbsp;
    </div>
  </div>
  <div class="hud-actions">
    <div id="user-email-display">user@demo.io</div>
    <button class="hud-btn" onclick="toggleTheme()">☀ THEME</button>
    <button class="hud-btn" onclick="toggleMusic()">♫ MUSIC</button>
    <button class="hud-btn" onclick="exportPDF()">⬇ PDF</button>
    <button class="hud-btn danger" onclick="doLogout()">⏏ LOGOUT</button>
  </div>
</div>
 
<!-- MINI NAV -->
<div id="mini-nav">
  <div class="nav-dot active" data-label="PROFILE"  onclick="flyToSection('profile')"></div>
  <div class="nav-dot" data-label="PROJECTS" onclick="flyToSection('projects')"></div>
  <div class="nav-dot" data-label="SKILLS"   onclick="flyToSection('skills')"></div>
  <div class="nav-dot" data-label="CONTACT"  onclick="flyToSection('contact')"></div>
</div>
 
<!-- LABELS -->
<div id="labels-container"></div>
 
<!-- CROSSHAIR -->
<div id="world-crosshair"><div class="xh-ring"></div><div class="xh-dot"></div><div class="xh-line-h"></div><div class="xh-line-v"></div></div>
 
<!-- MUSIC INDICATOR -->
<div id="music-indicator">
  <div class="music-bars paused" id="music-bars">
    <div class="music-bar" style="height:40%"></div>
    <div class="music-bar" style="height:60%"></div>
    <div class="music-bar" style="height:90%"></div>
    <div class="music-bar" style="height:50%"></div>
    <div class="music-bar" style="height:70%"></div>
  </div>
  <span id="music-label">BGM OFF</span>
</div>
 
<!-- TOAST -->
<div id="toast"></div>
 
<!-- ══════════ PANELS ══════════ -->
 
<!-- PROFILE PANEL -->
<div class="panel-overlay" id="panel-profile">
  <div class="panel-box">
    <button class="close-btn" onclick="closePanel('panel-profile')">✕</button>
    <div class="panel-title"><span class="icon">◈</span> PROFILE NODE</div>
    <div id="profile-view"></div>
    <button class="save-btn" onclick="openEdit()" style="width:100%;margin-top:16px">⚙ EDIT PROFILE</button>
  </div>
</div>
 
<!-- EDIT PROFILE PANEL -->
<div class="panel-overlay" id="panel-edit">
  <div class="panel-box">
    <button class="close-btn" onclick="closePanel('panel-edit')">✕</button>
    <div class="panel-title"><span class="icon">⚙</span> EDIT PROFILE</div>
    <div class="p-row">
      <div class="form-group">
        <label class="form-label">// NAME</label>
        <input class="form-input" id="ep-name" type="text" placeholder="Your Name">
      </div>
      <div class="form-group">
        <label class="form-label">// TITLE</label>
        <input class="form-input" id="ep-title" type="text" placeholder="Full Stack Dev">
      </div>
      <div class="form-group p-full">
        <label class="form-label">// BIO</label>
        <textarea class="form-input" id="ep-bio" rows="3" placeholder="Tell the world who you are..." style="resize:vertical"></textarea>
      </div>
      <div class="form-group">
        <label class="form-label">// GITHUB URL</label>
        <input class="form-input" id="ep-github" type="url" placeholder="https://github.com/...">
      </div>
      <div class="form-group">
        <label class="form-label">// LINKEDIN URL</label>
        <input class="form-input" id="ep-linkedin" type="url" placeholder="https://linkedin.com/...">
      </div>
      <div class="form-group">
        <label class="form-label">// WEBSITE</label>
        <input class="form-input" id="ep-website" type="url" placeholder="https://yoursite.dev">
      </div>
      <div class="form-group">
        <label class="form-label">// EMAIL (PUBLIC)</label>
        <input class="form-input" id="ep-contact-email" type="email" placeholder="hello@yoursite.dev">
      </div>
      <div class="form-group p-full">
        <label class="form-label">// SKILLS (COMMA SEPARATED)</label>
        <input class="form-input" id="ep-skills" type="text" placeholder="React, Node.js, Three.js, Python">
      </div>
    </div>
    <button class="save-btn" onclick="saveProfile()">💾 SAVE CHANGES</button>
  </div>
</div>
 
<!-- PROJECTS PANEL -->
<div class="panel-overlay" id="panel-projects">
  <div class="panel-box" style="max-width:700px">
    <button class="close-btn" onclick="closePanel('panel-projects')">✕</button>
    <div class="panel-title"><span class="icon">◧</span> PROJECTS MATRIX</div>
    <div id="projects-view"></div>
    <button class="save-btn" onclick="openAddProject()" style="margin-top:16px">+ ADD PROJECT</button>
  </div>
</div>
 
<!-- ADD PROJECT PANEL -->
<div class="panel-overlay" id="panel-add-project">
  <div class="panel-box">
    <button class="close-btn" onclick="closePanel('panel-add-project')">✕</button>
    <div class="panel-title"><span class="icon">+</span> NEW PROJECT</div>
    <div class="p-row">
      <div class="form-group">
        <label class="form-label">// PROJECT NAME</label>
        <input class="form-input" id="np-name" type="text" placeholder="My Awesome App">
      </div>
      <div class="form-group">
        <label class="form-label">// TECH STACK (COMMA SEP)</label>
        <input class="form-input" id="np-tech" type="text" placeholder="React, Firebase, CSS">
      </div>
      <div class="form-group p-full">
        <label class="form-label">// DESCRIPTION</label>
        <textarea class="form-input" id="np-desc" rows="3" placeholder="What does it do?" style="resize:vertical"></textarea>
      </div>
      <div class="form-group">
        <label class="form-label">// LIVE URL</label>
        <input class="form-input" id="np-live" type="url" placeholder="https://myapp.com">
      </div>
      <div class="form-group">
        <label class="form-label">// GITHUB URL</label>
        <input class="form-input" id="np-github" type="url" placeholder="https://github.com/...">
      </div>
    </div>
    <button class="save-btn" onclick="saveProject()">🚀 LAUNCH PROJECT</button>
  </div>
</div>
 
<!-- SKILLS PANEL -->
<div class="panel-overlay" id="panel-skills">
  <div class="panel-box">
    <button class="close-btn" onclick="closePanel('panel-skills')">✕</button>
    <div class="panel-title"><span class="icon">◉</span> SKILLS MATRIX</div>
    <div id="skills-view"></div>
  </div>
</div>
 
<!-- CONTACT PANEL -->
<div class="panel-overlay" id="panel-contact">
  <div class="panel-box">
    <button class="close-btn" onclick="closePanel('panel-contact')">✕</button>
    <div class="panel-title"><span class="icon">◫</span> CONTACT NODE</div>
    <div id="contact-view"></div>
  </div>
</div>
 
<!-- THREE.JS + APP LOGIC -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
// ═══════════════════════════════════════════════
//  APP STATE
// ═══════════════════════════════════════════════
let currentUser = null;
let userData = null;
let isDemoMode = false;
let isLightTheme = false;
let isMusicOn = false;
let bgMusic = null;
 
const DEMO_DATA = {
  name: "Alex Cyberdev",
  title: "Full Stack Developer",
  bio: "Passionate developer crafting immersive digital experiences. Specializing in 3D web, AI integrations, and scalable systems. Coffee-fueled, pixel-perfect, universe-curious.",
  skills: ["Three.js","React","Node.js","Firebase","Python","WebGL","TypeScript","Tailwind","Docker","AWS"],
  projects: [
    { name:"NeuroChat AI",       desc:"Real-time AI chat powered by GPT-4 with voice synthesis and emotion detection.",             tech:["React","OpenAI","WebSockets"],  live:"#",github:"#" },
    { name:"VoxelVerse",         desc:"Voxel-based multiplayer world builder with physics and real-time collaboration.",            tech:["Three.js","Socket.io","MongoDB"], live:"#",github:"#" },
    { name:"CryptoFlow",         desc:"DeFi analytics dashboard with live on-chain data, charts and wallet integration.",           tech:["Next.js","Web3.js","Chart.js"],  live:"#",github:"#" },
    { name:"PixelForge Studio",  desc:"Browser-based pixel art editor with animation support, layers and cloud export.",            tech:["Canvas API","Tailwind","Supabase"],live:"#",github:"#" },
  ],
  github:"https://github.com",
  linkedin:"https://linkedin.com",
  website:"https://cyberdev.io",
  contactEmail:"hello@cyberdev.io"
};
 
// ═══════════════════════════════════════════════
//  FIREBASE HELPERS
// ═══════════════════════════════════════════════
async function fbGetUserData(uid){
  try{
    const {db,doc,getDoc}=window.__FB;
    const snap=await getDoc(doc(db,'users',uid));
    return snap.exists()?snap.data():null;
  }catch(e){return null}
}
async function fbSetUserData(uid,data){
  try{
    const {db,doc,setDoc}=window.__FB;
    await setDoc(doc(db,'users',uid),data,{merge:true});
    return true;
  }catch(e){return false}
}
 
// ═══════════════════════════════════════════════
//  AUTH FLOW
// ═══════════════════════════════════════════════
function switchTab(t){
  document.getElementById('form-login').style.display  = t==='login' ?'block':'none';
  document.getElementById('form-signup').style.display = t==='signup'?'block':'none';
  document.getElementById('tab-login').classList.toggle('active',t==='login');
  document.getElementById('tab-signup').classList.toggle('active',t==='signup');
}
 
function validateEmail(e){return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(e)}
function showErr(id,msg){const el=document.getElementById(id);el.textContent=msg;el.style.display='block'}
function clearErrs(...ids){ids.forEach(id=>{const el=document.getElementById(id);if(el){el.style.display='none'}})}
 
async function doLogin(){
  clearErrs('le-err','lp-err');
  const email=document.getElementById('login-email').value.trim();
  const pass =document.getElementById('login-pass').value;
  if(!validateEmail(email)){showErr('le-err','⚠ Invalid email format');return}
  if(pass.length<6){showErr('lp-err','⚠ Password must be ≥ 6 characters');return}
  showLoader('AUTHENTICATING...');
  try{
    const {auth,signInWithEmailAndPassword}=window.__FB;
    const cred=await signInWithEmailAndPassword(auth,email,pass);
    currentUser=cred.user;
    let data=await fbGetUserData(currentUser.uid);
    if(!data){data={...DEMO_DATA,name:email.split('@')[0]};await fbSetUserData(currentUser.uid,data)}
    userData=data;
    enterWorld(email);
  }catch(e){
    hideLoader();
    showErr('lp-err','⚠ '+(e.code==='auth/user-not-found'?'No account found':'Invalid credentials'));
  }
}
 
async function doSignup(){
  clearErrs('se-err','sp-err');
  const name =document.getElementById('su-name').value.trim();
  const email=document.getElementById('su-email').value.trim();
  const pass =document.getElementById('su-pass').value;
  if(!validateEmail(email)){showErr('se-err','⚠ Invalid email format');return}
  if(pass.length<6){showErr('sp-err','⚠ Password must be ≥ 6 characters');return}
  showLoader('CREATING ACCOUNT...');
  try{
    const {auth,createUserWithEmailAndPassword}=window.__FB;
    const cred=await createUserWithEmailAndPassword(auth,email,pass);
    currentUser=cred.user;
    userData={...DEMO_DATA,name:name||email.split('@')[0]};
    await fbSetUserData(currentUser.uid,userData);
    enterWorld(email);
  }catch(e){
    hideLoader();
    showErr('sp-err','⚠ '+(e.code==='auth/email-already-in-use'?'Email already registered':'Signup failed: '+e.message));
  }
}
 
function demoLogin(){
  isDemoMode=true;
  userData={...DEMO_DATA};
  showLoader('LOADING DEMO WORLD...');
  setTimeout(()=>enterWorld('demo@cyberworld.io'),600);
}
 
async function doLogout(){
  if(!isDemoMode&&window.__FB){
    try{await window.__FB.signOut(window.__FB.auth)}catch(e){}
  }
  currentUser=null; userData=null; isDemoMode=false;
  document.getElementById('hud').classList.remove('visible');
  document.getElementById('mini-nav').classList.remove('visible');
  document.getElementById('music-indicator').classList.remove('visible');
  document.getElementById('login-screen').style.display='flex';
  if(bgMusic){bgMusic.pause();isMusicOn=false}
  showToast('Logged out successfully','success');
}
 
function enterWorld(email){
  document.getElementById('login-screen').style.display='none';
  document.getElementById('user-email-display').textContent=email;
  document.getElementById('hud').classList.add('visible');
  document.getElementById('mini-nav').classList.add('visible');
  document.getElementById('music-indicator').classList.add('visible');
  hideLoader();
  initThreeJS();
  showToast('Welcome to 3D Portfolio World!','success');
}
 
// ═══════════════════════════════════════════════
//  LOADER HELPERS
// ═══════════════════════════════════════════════
let loaderTimer=null;
function showLoader(msg='LOADING...'){
  const l=document.getElementById('loader');
  l.classList.remove('hidden');
  l.querySelector('.loader-text').textContent=msg;
}
function hideLoader(){
  setTimeout(()=>document.getElementById('loader').classList.add('hidden'),1000);
}
 
// ═══════════════════════════════════════════════
//  THREE.JS ENGINE
// ═══════════════════════════════════════════════
let scene,camera,renderer,clock;
let objects=[]; // {mesh, type, section, label}
let particles,particleSystem;
let camTarget={x:0,y:0,z:5};
let camCurrent={x:0,y:0,z:5};
let mouseX=0,mouseY=0;
let SECTIONS={profile:{pos:[-6,2,0]},projects:{pos:[6,2,-2]},skills:{pos:[0,4,-4]},contact:{pos:[0,-2,-2]}};
let animFrame;
 
function initThreeJS(){
  if(animFrame){cancelAnimationFrame(animFrame)}
 
  const canvas=document.getElementById('three-canvas');
  scene=new THREE.Scene();
  scene.fog=new THREE.FogExp2(0x000814,0.04);
 
  camera=new THREE.PerspectiveCamera(60,innerWidth/innerHeight,0.1,200);
  camera.position.set(0,0,12);
 
  renderer=new THREE.WebGLRenderer({canvas,antialias:true,alpha:true});
  renderer.setSize(innerWidth,innerHeight);
  renderer.setPixelRatio(Math.min(devicePixelRatio,2));
  renderer.toneMapping=THREE.ACESFilmicToneMapping;
  renderer.toneMappingExposure=1.2;
 
  clock=new THREE.Clock();
  buildScene();
  buildLabels();
  addEventListeners();
  animate();
}
 
function buildScene(){
  // ── Grid floor ──────────────────────────────
  const gridHelper=new THREE.GridHelper(100,40,0x003366,0x001122);
  gridHelper.position.y=-4;
  scene.add(gridHelper);
 
  // ── Ambient + point lights ──────────────────
  scene.add(new THREE.AmbientLight(0x001133,0.8));
  const pl1=new THREE.PointLight(0x00f3ff,3,30);pl1.position.set(-5,5,5);scene.add(pl1);
  const pl2=new THREE.PointLight(0xff00e5,3,30);pl2.position.set(5,3,-3);scene.add(pl2);
  const pl3=new THREE.PointLight(0x9d00ff,2,20);pl3.position.set(0,8,-6);scene.add(pl3);
 
  // ── Background stars ────────────────────────
  const starsGeo=new THREE.BufferGeometry();
  const count=3000;
  const pos=new Float32Array(count*3);
  for(let i=0;i<count*3;i++)pos[i]=(Math.random()-.5)*200;
  starsGeo.setAttribute('position',new THREE.BufferAttribute(pos,3));
  const starsMat=new THREE.PointsMaterial({color:0xaaccff,size:.15,transparent:true,opacity:.8});
  scene.add(new THREE.Points(starsGeo,starsMat));
 
  // ── Particles ───────────────────────────────
  const pGeo=new THREE.BufferGeometry();
  const pCount=800;
  const pPos=new Float32Array(pCount*3);
  for(let i=0;i<pCount*3;i++)pPos[i]=(Math.random()-.5)*50;
  pGeo.setAttribute('position',new THREE.BufferAttribute(pPos,3));
  const pMat=new THREE.PointsMaterial({color:0x00f3ff,size:.06,transparent:true,opacity:.5});
  particleSystem=new THREE.Points(pGeo,pMat);
  scene.add(particleSystem);
 
  // ── PROFILE cube ────────────────────────────
  addSection('profile', [-6,2,0],  0x00f3ff);
  addSection('projects',[6,2,-2],  0xff00e5);
  addSection('skills',  [0,4,-4],  0x9d00ff);
  addSection('contact', [0,-2,-2], 0x00ff88);
 
  // ── Decorative rings ────────────────────────
  for(let i=0;i<4;i++){
    const r=new THREE.TorusGeometry(8+i*3,.03,8,60);
    const rm=new THREE.MeshBasicMaterial({color:[0x00f3ff,0xff00e5,0x9d00ff,0x00ff88][i],transparent:true,opacity:.25});
    const ring=new THREE.Mesh(r,rm);
    ring.rotation.x=Math.PI/2+(i*.3);
    ring.position.y=-4;
    scene.add(ring);
  }
 
  // ── Floating geometric shapes ────────────────
  const shapes=[
    {geo:new THREE.OctahedronGeometry(.5),pos:[3,5,2],  col:0x00f3ff},
    {geo:new THREE.TetrahedronGeometry(.6),pos:[-4,-1,1],col:0xff00e5},
    {geo:new THREE.IcosahedronGeometry(.4),pos:[2,-3,-2],col:0x9d00ff},
    {geo:new THREE.OctahedronGeometry(.35),pos:[-2,6,-3],col:0x00ff88},
  ];
  shapes.forEach(s=>{
    const mat=new THREE.MeshStandardMaterial({color:s.col,emissive:s.col,emissiveIntensity:.4,wireframe:true});
    const mesh=new THREE.Mesh(s.geo,mat);
    mesh.position.set(...s.pos);
    mesh.userData={type:'deco',offset:Math.random()*Math.PI*2,speed:.5+Math.random()*.5};
    scene.add(mesh);
    objects.push({mesh,type:'deco'});
  });
}
 
function addSection(name,pos,color){
  const geo=new THREE.BoxGeometry(2.2,2.2,2.2);
  const mat=new THREE.MeshStandardMaterial({
    color,emissive:color,emissiveIntensity:.3,
    transparent:true,opacity:.8,
    roughness:.2,metalness:.8,
    wireframe:false
  });
  const mesh=new THREE.Mesh(geo,mat);
  mesh.position.set(...pos);
  mesh.userData={section:name,offset:Math.random()*Math.PI*2,color};
 
  // Wireframe overlay
  const wireMat=new THREE.MeshBasicMaterial({color,wireframe:true,transparent:true,opacity:.25});
  const wire=new THREE.Mesh(geo,wireMat);
  mesh.add(wire);
 
  // Glow ring
  const ring=new THREE.TorusGeometry(1.6,.04,8,40);
  const ringMat=new THREE.MeshBasicMaterial({color,transparent:true,opacity:.5});
  const ringMesh=new THREE.Mesh(ring,ringMat);
  ringMesh.rotation.x=Math.PI/2;
  mesh.add(ringMesh);
 
  scene.add(mesh);
  objects.push({mesh,type:'section',name});
}
 
function buildLabels(){
  const labelsEl=document.getElementById('labels-container');
  labelsEl.innerHTML='';
  Object.entries(SECTIONS).forEach(([key,{pos}])=>{
    const div=document.createElement('div');
    div.className='world-label';
    div.dataset.section=key;
    div.innerHTML=`<div class="world-label-inner wl-${key}">${key.toUpperCase()}</div>`;
    div.onclick=()=>openPanel(key);
    labelsEl.appendChild(div);
  });
}
 
function updateLabels(){
  if(!camera||!renderer)return;
  const labelsEl=document.getElementById('labels-container');
  const labels=labelsEl.querySelectorAll('.world-label');
  const W=renderer.domElement.clientWidth;
  const H=renderer.domElement.clientHeight;
 
  labels.forEach(lbl=>{
    const sec=lbl.dataset.section;
    const pos=SECTIONS[sec].pos;
    const v=new THREE.Vector3(pos[0],pos[1]+1.8,pos[2]);
    v.project(camera);
    const x=(v.x*.5+.5)*W;
    const y=(-.5*v.y+.5)*H;
    lbl.style.left=x+'px';
    lbl.style.top=y+'px';
    lbl.style.opacity=v.z<1?.9:0;
  });
}
 
function animate(){
  animFrame=requestAnimationFrame(animate);
  const t=clock.getElapsedTime();
 
  // Camera gentle sway
  camCurrent.x+=(camTarget.x-camCurrent.x)*.03;
  camCurrent.y+=(camTarget.y-camCurrent.y)*.03;
  camCurrent.z+=(camTarget.z-camCurrent.z)*.03;
  camera.position.x=camCurrent.x+Math.sin(t*.15)*0.5;
  camera.position.y=camCurrent.y+Math.cos(t*.12)*0.3;
  camera.position.z=camCurrent.z;
  camera.lookAt(camCurrent.x,camCurrent.y,0);
 
  // Section cubes float + spin
  objects.forEach(o=>{
    if(o.type==='section'){
      const off=o.mesh.userData.offset;
      o.mesh.position.y=SECTIONS[o.name].pos[1]+Math.sin(t*.6+off)*.3;
      o.mesh.rotation.y+=.005;
      o.mesh.rotation.x+=.003;
    }
    if(o.type==='deco'){
      const {offset,speed}=o.mesh.userData;
      o.mesh.rotation.x+=.01*speed;
      o.mesh.rotation.y+=.008*speed;
      o.mesh.position.y+=Math.sin(t*speed+offset)*.003;
    }
  });
 
  // Particles drift
  if(particleSystem){
    particleSystem.rotation.y+=.0003;
    particleSystem.rotation.x+=.0001;
  }
 
  updateLabels();
  renderer.render(scene,camera);
}
 
function flyToSection(sec){
  const pos=SECTIONS[sec].pos;
  camTarget={x:pos[0]*.4,y:pos[1]*.3,z:8};
  document.querySelectorAll('#mini-nav .nav-dot').forEach((d,i)=>{
    d.classList.toggle('active',['profile','projects','skills','contact'][i]===sec);
  });
}
 
// ═══════════════════════════════════════════════
//  RAYCASTING – CLICK ON CUBES
// ═══════════════════════════════════════════════
function addEventListeners(){
  const canvas=document.getElementById('three-canvas');
  canvas.style.pointerEvents='all';
  canvas.addEventListener('mousemove',e=>{
    mouseX=(e.clientX/innerWidth-.5)*2;
    mouseY=-(e.clientY/innerHeight-.5)*2;
    camTarget.x=mouseX*.8;
    camTarget.y=mouseY*.4+1;
  });
  canvas.addEventListener('click',e=>{
    const raycaster=new THREE.Raycaster();
    const mouse=new THREE.Vector2((e.clientX/innerWidth)*2-1,-(e.clientY/innerHeight)*2+1);
    raycaster.setFromCamera(mouse,camera);
    const hits=raycaster.intersectObjects(objects.filter(o=>o.type==='section').map(o=>o.mesh));
    if(hits.length>0){
      const sec=hits[0].object.userData.section;
      if(sec)openPanel(sec);
    }
  });
  window.addEventListener('resize',()=>{
    if(!camera||!renderer)return;
    camera.aspect=innerWidth/innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(innerWidth,innerHeight);
  });
}
 
// ═══════════════════════════════════════════════
//  PANELS
// ═══════════════════════════════════════════════
function openPanel(sec){
  flyToSection(sec);
  renderPanelContent(sec);
  document.getElementById('panel-'+sec).classList.add('open');
}
function closePanel(id){document.getElementById(id).classList.remove('open')}
 
function renderPanelContent(sec){
  if(!userData)return;
  if(sec==='profile') renderProfile();
  if(sec==='projects')renderProjects();
  if(sec==='skills')  renderSkills();
  if(sec==='contact') renderContact();
}
 
function renderProfile(){
  const d=userData;
  const el=document.getElementById('profile-view');
  const skills=(d.skills||[]).slice(0,5).map(s=>`<span style="background:rgba(0,243,255,.08);border:1px solid rgba(0,243,255,.2);color:var(--neon-blue);padding:3px 10px;border-radius:12px;font-size:11px;font-family:var(--font-mono)">${s}</span>`).join('');
  el.innerHTML=`
    <div class="profile-avatar">👾</div>
    <div class="profile-name">${d.name||'Anonymous Dev'}</div>
    <div style="text-align:center;font-family:var(--font-mono);font-size:11px;color:var(--neon-pink);letter-spacing:2px;margin-bottom:8px">${d.title||'Developer'}</div>
    <div class="profile-bio">${d.bio||'No bio yet.'}</div>
    <div class="profile-stats">
      <div class="stat-item"><div class="stat-num">${(d.projects||[]).length}</div><div class="stat-lbl">PROJECTS</div></div>
      <div class="stat-item"><div class="stat-num">${(d.skills||[]).length}</div><div class="stat-lbl">SKILLS</div></div>
      <div class="stat-item"><div class="stat-num">∞</div><div class="stat-lbl">PASSION</div></div>
    </div>
    <div style="display:flex;flex-wrap:wrap;gap:6px;justify-content:center">${skills}</div>`;
}
 
function renderProjects(){
  const el=document.getElementById('projects-view');
  const projs=(userData.projects||[]);
  if(!projs.length){el.innerHTML='<p style="color:var(--text-dim);font-family:var(--font-mono);font-size:12px">No projects yet. Add your first one!</p>';return}
  el.innerHTML=`<div class="projects-grid">${projs.map((p,i)=>`
    <div class="project-card" onclick="this.style.borderColor='var(--neon-blue)'">
      <h4>${p.name}</h4>
      <p>${p.desc}</p>
      <div class="tech-list">${(p.tech||[]).map(t=>`<span class="tech-badge">${t}</span>`).join('')}</div>
      <div class="p-links">
        ${p.live?`<a class="p-link" href="${p.live}" target="_blank">↗ LIVE</a>`:''}
        ${p.github?`<a class="p-link" href="${p.github}" target="_blank">⌥ CODE</a>`:''}
        <span class="p-link" style="cursor:pointer;color:#ff4466;border-color:rgba(255,68,102,.3)" onclick="deleteProject(${i})">✕</span>
      </div>
    </div>`).join('')}</div>`;
}
 
function renderSkills(){
  const el=document.getElementById('skills-view');
  const skills=(userData.skills||[]);
  if(!skills.length){el.innerHTML='<p style="color:var(--text-dim);font-family:var(--font-mono);font-size:12px">No skills yet. Edit your profile to add some!</p>';return}
  const colors=['var(--neon-blue)','var(--neon-pink)','var(--neon-purple)','var(--neon-green)'];
  el.innerHTML=`<div class="skill-tags">${skills.map((s,i)=>`
    <span class="skill-tag" style="border-color:${colors[i%4]};color:${colors[i%4]};background:${colors[i%4]}15">
      ${s}
    </span>`).join('')}</div>
    <div style="margin-top:20px">
      ${skills.map((s,i)=>{
        const pct=65+((i*17+37)%35);
        return `<div style="margin-bottom:12px">
          <div style="display:flex;justify-content:space-between;font-family:var(--font-mono);font-size:11px;margin-bottom:4px">
            <span style="color:${colors[i%4]}">${s}</span><span style="color:var(--text-dim)">${pct}%</span>
          </div>
          <div style="height:4px;background:var(--glass-border);border-radius:2px;overflow:hidden">
            <div style="height:100%;width:${pct}%;background:linear-gradient(90deg,${colors[i%4]},${colors[(i+1)%4]});border-radius:2px;transition:width 1s ease"></div>
          </div>
        </div>`;
      }).join('')}
    </div>`;
}
 
function renderContact(){
  const d=userData;
  document.getElementById('contact-view').innerHTML=`
    <p style="color:var(--text-dim);font-size:13px;line-height:1.7;margin-bottom:16px">Let's connect and build something amazing together.</p>
    <div class="contact-links">
      ${d.contactEmail?`<a class="contact-link" href="mailto:${d.contactEmail}">✉ ${d.contactEmail}</a>`:''}
      ${d.github?`<a class="contact-link" href="${d.github}" target="_blank">⌥ GitHub</a>`:''}
      ${d.linkedin?`<a class="contact-link" href="${d.linkedin}" target="_blank">◈ LinkedIn</a>`:''}
      ${d.website?`<a class="contact-link" href="${d.website}" target="_blank">↗ Website</a>`:''}
    </div>
    <div style="margin-top:20px;background:rgba(0,243,255,.04);border:1px solid var(--glass-border);border-radius:10px;padding:16px">
      <div style="font-family:var(--font-mono);font-size:10px;color:var(--neon-blue);letter-spacing:2px;margin-bottom:12px">// SEND MESSAGE</div>
      <input class="form-input" id="msg-name" placeholder="Your name" style="margin-bottom:8px">
      <input class="form-input" id="msg-email" type="email" placeholder="Your email" style="margin-bottom:8px">
      <textarea class="form-input" id="msg-text" rows="3" placeholder="Your message..." style="resize:vertical;margin-bottom:8px"></textarea>
      <button class="save-btn" onclick="sendContactMsg()" style="width:100%">⚡ TRANSMIT MESSAGE</button>
    </div>`;
}
 
function sendContactMsg(){
  const name=document.getElementById('msg-name').value;
  const email=document.getElementById('msg-email').value;
  const text=document.getElementById('msg-text').value;
  if(!name||!email||!text){showToast('Please fill all fields','error');return}
  showToast('Message transmitted! ✓','success');
  document.getElementById('msg-name').value='';
  document.getElementById('msg-email').value='';
  document.getElementById('msg-text').value='';
}
 
// ═══════════════════════════════════════════════
//  EDIT PROFILE
// ═══════════════════════════════════════════════
function openEdit(){
  const d=userData||{};
  document.getElementById('ep-name').value=d.name||'';
  document.getElementById('ep-title').value=d.title||'';
  document.getElementById('ep-bio').value=d.bio||'';
  document.getElementById('ep-github').value=d.github||'';
  document.getElementById('ep-linkedin').value=d.linkedin||'';
  document.getElementById('ep-website').value=d.website||'';
  document.getElementById('ep-contact-email').value=d.contactEmail||'';
  document.getElementById('ep-skills').value=(d.skills||[]).join(', ');
  document.getElementById('panel-edit').classList.add('open');
}
async function saveProfile(){
  const skills=document.getElementById('ep-skills').value.split(',').map(s=>s.trim()).filter(Boolean);
  const updated={
    name:document.getElementById('ep-name').value.trim(),
    title:document.getElementById('ep-title').value.trim(),
    bio:document.getElementById('ep-bio').value.trim(),
    github:document.getElementById('ep-github').value.trim(),
    linkedin:document.getElementById('ep-linkedin').value.trim(),
    website:document.getElementById('ep-website').value.trim(),
    contactEmail:document.getElementById('ep-contact-email').value.trim(),
    skills
  };
  userData={...userData,...updated};
  if(!isDemoMode&&currentUser)await fbSetUserData(currentUser.uid,userData);
  closePanel('panel-edit');
  showToast('Profile updated!','success');
  renderProfile();
}
 
// ═══════════════════════════════════════════════
//  PROJECTS
// ═══════════════════════════════════════════════
function openAddProject(){document.getElementById('panel-add-project').classList.add('open')}
async function saveProject(){
  const name=document.getElementById('np-name').value.trim();
  const desc=document.getElementById('np-desc').value.trim();
  const tech=document.getElementById('np-tech').value.split(',').map(s=>s.trim()).filter(Boolean);
  const live=document.getElementById('np-live').value.trim();
  const github=document.getElementById('np-github').value.trim();
  if(!name||!desc){showToast('Name and description required','error');return}
  const proj={name,desc,tech,live,github,createdAt:Date.now()};
  if(!userData.projects)userData.projects=[];
  userData.projects.push(proj);
  if(!isDemoMode&&currentUser)await fbSetUserData(currentUser.uid,{projects:userData.projects});
  closePanel('panel-add-project');
  showToast('Project added!','success');
  ['np-name','np-desc','np-tech','np-live','np-github'].forEach(id=>document.getElementById(id).value='');
  renderProjects();
}
async function deleteProject(i){
  userData.projects.splice(i,1);
  if(!isDemoMode&&currentUser)await fbSetUserData(currentUser.uid,{projects:userData.projects});
  showToast('Project removed','success');
  renderProjects();
}
 
// ═══════════════════════════════════════════════
//  THEME TOGGLE
// ═══════════════════════════════════════════════
function toggleTheme(){
  isLightTheme=!isLightTheme;
  document.documentElement.classList.toggle('light-theme',isLightTheme);
  if(scene&&renderer)renderer.setClearColor(isLightTheme?0xeef4ff:0x000000,isLightTheme?.1:0);
  showToast(isLightTheme?'Light theme active':'Dark theme active','success');
}
 
// ═══════════════════════════════════════════════
//  MUSIC TOGGLE
// ═══════════════════════════════════════════════
function toggleMusic(){
  if(!bgMusic){
    // Generate simple synth music via Web Audio API
    bgMusic=createSynthMusic();
  }
  isMusicOn=!isMusicOn;
  const bars=document.getElementById('music-bars');
  const lbl=document.getElementById('music-label');
  if(isMusicOn){
    bars.classList.remove('paused');
    lbl.textContent='BGM ON';
    bgMusic.play();
    showToast('Background music ON','success');
  }else{
    bars.classList.add('paused');
    lbl.textContent='BGM OFF';
    bgMusic.pause();
    showToast('Background music OFF','success');
  }
}
function createSynthMusic(){
  const ctx=new(window.AudioContext||window.webkitAudioContext)();
  const notes=[261.63,293.66,329.63,349.23,392,440,493.88];
  let noteIdx=0,nextTime=ctx.currentTime;
  const gainNode=ctx.createGain();gainNode.gain.value=.12;gainNode.connect(ctx.destination);
  const seq=()=>{
    const osc=ctx.createOscillator();
    const g=ctx.createGain();
    osc.type='sine';
    osc.frequency.value=notes[noteIdx%notes.length]*(Math.random()<.3?2:1);
    osc.connect(g);g.connect(gainNode);
    g.gain.setValueAtTime(0,nextTime);
    g.gain.linearRampToValueAtTime(.3,nextTime+.1);
    g.gain.linearRampToValueAtTime(0,nextTime+.5);
    osc.start(nextTime);osc.stop(nextTime+.55);
    nextTime+=.42+Math.random()*.2;
    noteIdx++;
  };
  let timer;
  return{
    play(){timer=setInterval(seq,420)},
    pause(){clearInterval(timer)}
  };
}
 
// ═══════════════════════════════════════════════
//  PDF EXPORT (using print-to-PDF approach)
// ═══════════════════════════════════════════════
function exportPDF(){
  if(!userData){showToast('No data to export','error');return}
  const d=userData;
  const w=window.open('','_blank');
  w.document.write(`<!DOCTYPE html><html><head><title>${d.name} - Portfolio</title>
  <style>
    body{font-family:monospace;background:#020408;color:#e8f4ff;padding:40px;max-width:800px;margin:0 auto}
    h1{color:#00f3ff;border-bottom:1px solid #003355;padding-bottom:10px}
    h2{color:#9d00ff;margin-top:24px}
    .tag{display:inline-block;background:#002244;border:1px solid #004488;color:#00f3ff;padding:2px 10px;border-radius:10px;margin:3px;font-size:12px}
    .proj{background:#001122;border:1px solid #002244;border-radius:6px;padding:14px;margin-bottom:12px}
    .proj h3{color:#ff00e5;margin:0 0 6px}
    @media print{body{background:white;color:#000}.tag{border-color:#0066cc;color:#0066cc;background:#e6f0ff}.proj{border-color:#ccc;background:#f5f5f5}.proj h3{color:#cc0088}h1{color:#003399}h2{color:#660099}}
  </style></head><body>
  <h1>◈ ${d.name} — Portfolio</h1>
  <p><em>${d.title||''}</em></p>
  <p>${d.bio||''}</p>
  <h2>Skills</h2>
  ${(d.skills||[]).map(s=>`<span class="tag">${s}</span>`).join('')}
  <h2>Projects</h2>
  ${(d.projects||[]).map(p=>`<div class="proj"><h3>${p.name}</h3><p>${p.desc}</p><p><strong>Tech:</strong> ${(p.tech||[]).join(', ')}</p>${p.live?`<p><a href="${p.live}">↗ Live</a></p>`:''}</div>`).join('')}
  <h2>Contact</h2>
  ${d.contactEmail?`<p>✉ ${d.contactEmail}</p>`:''}
  ${d.github?`<p>⌥ ${d.github}</p>`:''}
  ${d.linkedin?`<p>◈ ${d.linkedin}</p>`:''}
  <script>window.onload=()=>window.print()<\/script>
  </body></html>`);
  w.document.close();
  showToast('Opening PDF export…','success');
}
 
// ═══════════════════════════════════════════════
//  TOAST
// ═══════════════════════════════════════════════
let toastTimer;
function showToast(msg,type='success'){
  const t=document.getElementById('toast');
  t.textContent=msg;
  t.className='show '+(type||'');
  clearTimeout(toastTimer);
  toastTimer=setTimeout(()=>t.className='',2800);
}
 
// ═══════════════════════════════════════════════
//  FIREBASE AUTH OBSERVER
// ═══════════════════════════════════════════════
window.addEventListener('load',async()=>{
  // Hide loader after 1.8s animation
  setTimeout(()=>document.getElementById('loader').classList.add('hidden'),2200);
 
  if(window.__FB){
    const {auth,onAuthStateChanged}=window.__FB;
    onAuthStateChanged(auth,async user=>{
      if(user&&!currentUser){
        currentUser=user;
        let data=await fbGetUserData(user.uid);
        if(!data){data={...DEMO_DATA};await fbSetUserData(user.uid,data)}
        userData=data;
        enterWorld(user.email);
      }
    });
  }
});
 
// Close panels on overlay click
document.querySelectorAll('.panel-overlay').forEach(el=>{
  el.addEventListener('click',e=>{if(e.target===el)el.classList.remove('open')});
});
</script>
</body>
</html>
 
