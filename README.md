<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>MultiShooter</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/peerjs/1.5.2/peerjs.min.js"></script>
<style>
*{margin:0;padding:0;box-sizing:border-box;}
body,html{width:100%;height:100%;background:#050810;overflow:hidden;font-family:Arial,sans-serif;color:#fff;}
.screen{position:fixed;inset:0;display:none;flex-direction:column;align-items:center;justify-content:center;background:#050810;z-index:100;gap:14px;}
h1{color:#ef4444;font-size:clamp(28px,5vw,54px);letter-spacing:6px;text-shadow:0 0 30px #ef4444;margin-bottom:8px;}
h2{color:#60a5fa;font-size:22px;margin-bottom:4px;}
.subtitle{color:#6b7280;font-size:14px;text-align:center;max-width:400px;line-height:1.6;}
input[type=text]{padding:11px 16px;border-radius:8px;border:2px solid #1e3a5f;background:#0f1729;color:#fff;font-size:15px;width:280px;text-align:center;outline:none;}
input[type=text]:focus{border-color:#3b82f6;}
input[type=number]{padding:8px 12px;border-radius:6px;border:2px solid #1e3a5f;background:#0f1729;color:#fff;font-size:14px;width:100px;text-align:center;outline:none;}
.btn{padding:12px 28px;border-radius:8px;border:2px solid #ef4444;background:#000;color:#fff;font:bold 15px Arial;cursor:pointer;width:280px;transition:background .2s;}
.btn:hover{background:#ef4444;}
.btn-blue{border-color:#3b82f6;}.btn-blue:hover{background:#3b82f6;}
.btn-green{border-color:#22c55e;}.btn-green:hover{background:#22c55e;}
.btn-sm{padding:8px 18px;font-size:13px;width:auto;}
.id-box{background:#0f1729;border:2px solid #1e3a5f;border-radius:8px;padding:11px 16px;color:#60a5fa;font-size:14px;text-align:center;width:280px;word-break:break-all;}
.status{color:#fbbf24;font-size:13px;min-height:18px;text-align:center;}
.divider{color:#374151;font-size:13px;width:280px;text-align:center;}
#colorPicker{display:flex;flex-wrap:wrap;justify-content:center;width:260px;gap:4px;}
.mode-grid{display:grid;grid-template-columns:1fr 1fr;gap:10px;width:360px;}
.mode-btn{padding:14px 10px;border-radius:8px;border:2px solid #374151;background:#0f1729;color:#9ca3af;font:bold 13px Arial;cursor:pointer;text-align:center;transition:all .2s;}
.mode-btn:hover,.mode-btn.selected{border-color:#ef4444;color:#fff;background:#1a0a0a;}
.mode-btn span{display:block;font:normal 11px Arial;color:#6b7280;margin-top:3px;}
#playerList{list-style:none;color:#d1d5db;font-size:14px;text-align:center;}
#playerList li{padding:4px 0;}
.row{display:flex;gap:10px;align-items:center;}
label{color:#9ca3af;font-size:13px;}
#gameCanvas{display:none;position:fixed;inset:0;}
.fullscreen-btn{position:fixed;bottom:18px;right:18px;padding:9px 14px;background:#000;color:#fff;border:2px solid #ef4444;border-radius:6px;font:bold 13px Arial;cursor:pointer;z-index:500;}
.fullscreen-btn:hover{background:#ef4444;}
#gTag{position:fixed;top:10px;left:10px;color:#fff;font-size:13px;z-index:500;}
#overlay{display:none;position:fixed;inset:0;background:rgba(0,0,0,.82);align-items:center;justify-content:center;flex-direction:column;gap:18px;z-index:400;}
#overlay h2{font-size:clamp(28px,5vw,54px);color:#ef4444;text-shadow:0 0 20px #ef4444;}
#overlay p{color:#d1d5db;font-size:18px;}
</style>
</head>
<body>

<p id="gTag">G+</p>
<button class="fullscreen-btn" onclick="toggleFS()">⛶ FULLSCREEN</button>

<!-- SETUP -->
<div class="screen" id="screenSetup" style="display:flex">
  <h1>MULTISHOOTER</h1>
  <p class="subtitle">Up to 8 players · 4 game modes · Weapons & powerups</p>
  <input type="text" id="nameInput" placeholder="Enter your name" maxlength="16"/>
  <div id="colorPicker"></div>
  <button class="btn btn-green" onclick="goSolo()">🤖 SOLO VS BOTS</button>
  <button class="btn" onclick="goHost()">🏠 HOST GAME</button>
  <button class="btn btn-blue" onclick="goJoin()">🔗 JOIN GAME</button>
  <div class="status" id="peerStatus">Connecting to network...</div>
</div>

<!-- HOST SETUP -->
<div class="screen" id="screenHost">
  <h1>HOST GAME</h1>
  <h2>Choose Game Mode</h2>
  <div class="mode-grid">
    <div class="mode-btn selected" onclick="selectMode('ffa',this)">FREE FOR ALL<span>Most kills wins</span></div>
    <div class="mode-btn" onclick="selectMode('tdm',this)">TEAM DEATHMATCH<span>Red vs Blue</span></div>
    <div class="mode-btn" onclick="selectMode('ctf',this)">CAPTURE THE FLAG<span>Steal enemy flag</span></div>
    <div class="mode-btn" onclick="selectMode('lms',this)">LAST MAN STANDING<span>Limited lives</span></div>
  </div>
  <div class="row">
    <label id="limitLabel">Kill Limit:</label>
    <input type="number" id="killLimitInput" value="15" min="5" max="100"/>
  </div>
  <div class="id-box" id="myRoomId">Getting room ID...</div>
  <button class="btn btn-sm" onclick="copyRoom()">📋 Copy Room ID</button>
  <div class="divider">── Players in lobby ──</div>
  <ul id="playerList"><li>Waiting...</li></ul>
  <button class="btn btn-green" onclick="hostStartGame()">▶ START GAME</button>
  <div class="status" id="hostStatus"></div>
</div>

<!-- JOIN -->
<div class="screen" id="screenJoin">
  <h1>JOIN GAME</h1>
  <p class="subtitle">Enter the Room ID your friend shared with you</p>
  <input type="text" id="joinInput" placeholder="Paste Room ID here"/>
  <button class="btn btn-blue" onclick="joinGame()">JOIN</button>
  <button class="btn btn-sm" onclick="showScreen('screenSetup')" style="border-color:#374151">← Back</button>
  <div class="status" id="joinStatus"></div>
</div>

<!-- SOLO SETUP -->
<div class="screen" id="screenSolo">
  <h1>SOLO vs BOTS</h1>
  <h2>Choose Game Mode</h2>
  <div class="mode-grid">
    <div class="mode-btn selected" onclick="selectSoloMode('ffa',this)">FREE FOR ALL<span>Most kills wins</span></div>
    <div class="mode-btn" onclick="selectSoloMode('tdm',this)">TEAM DEATHMATCH<span>You vs Bot team</span></div>
    <div class="mode-btn" onclick="selectSoloMode('lms',this)">LAST MAN STANDING<span>Limited lives</span></div>
    <div class="mode-btn" onclick="selectSoloMode('survival',this)">SURVIVAL<span>Survive endless waves</span></div>
  </div>
  <div class="row">
    <label>Bots:</label>
    <input type="number" id="botCountInput" value="4" min="1" max="10"/>
  </div>
  <div class="row">
    <label>Difficulty:</label>
    <select id="botDiff" style="padding:8px 12px;border-radius:6px;border:2px solid #1e3a5f;background:#0f1729;color:#fff;font-size:14px;outline:none;">
      <option value="easy">Easy</option>
      <option value="medium" selected>Medium</option>
      <option value="hard">Hard</option>
    </select>
  </div>
  <button class="btn btn-green" onclick="startSolo()">▶ PLAY</button>
  <button class="btn btn-sm" onclick="showScreen('screenSetup')" style="border-color:#374151">← Back</button>
</div>

<!-- WAITING -->
<div class="screen" id="screenWaiting">
  <h1>WAITING...</h1>
  <p class="subtitle" id="waitMsg">Waiting for host to start the game...</p>
  <div class="status" id="waitStatus">Connected ✓</div>
</div>

<!-- GAME CANVAS -->
<canvas id="gameCanvas"></canvas>

<!-- GAME OVER -->
<div id="overlay">
  <h2 id="overlayTitle">GAME OVER</h2>
  <p id="overlayMsg"></p>
  <button class="btn btn-green" onclick="location.reload()" style="width:200px">PLAY AGAIN</button>
</div>

<script>
// ═══ CONSTANTS ══════════════════════════════════════════════
const AW=2400, AH=1800, PS=16, PSPEED=4.2, RESP_T=180;

const WEAPONS={
  pistol: {name:'Pistol', dmg:25, spd:13, cd:10, spread:0,    pellets:1, color:'#fde68a', range:110, ammo:Infinity},
  shotgun:{name:'Shotgun',dmg:16, spd:10, cd:38, spread:0.28, pellets:6, color:'#fb923c', range:55,  ammo:8},
  sniper: {name:'Sniper', dmg:90, spd:24, cd:80, spread:0,    pellets:1, color:'#67e8f9', range:160, ammo:5},
};

const WALLS=[
  {x:0,   y:0,   w:400,h:90},
  {x:-820,y:-280,w:65, h:500},{x:820,y:-280,w:65,h:500},
  {x:-490,y:-640,w:140,h:140},{x:490,y:-640,w:140,h:140},
  {x:-490,y:500, w:140,h:140},{x:490,y:500, w:140,h:140},
  {x:-260,y:-430,w:65, h:200},{x:260,y:-430,w:65,h:200},
  {x:-260,y:230, w:65, h:200},{x:260,y:230, w:65,h:200},
  {x:-620,y:0,   w:140,h:65},{x:620,y:0,   w:140,h:65},
];

const WSPAWNS=[{x:0,y:-760},{x:0,y:760},{x:-760,y:0},{x:760,y:0},{x:-430,y:-430},{x:430,y:430},{x:430,y:-430},{x:-430,y:430}];
const HSPAWNS=[{x:-380,y:0},{x:380,y:0},{x:0,y:-360},{x:0,y:360}];
const FLAG_BASE={red:{x:980,y:0},blue:{x:-980,y:0}};
const SPAWNS={
  red: [{x:900,y:0},{x:900,y:-220},{x:900,y:220},{x:820,y:-440},{x:820,y:440}],
  blue:[{x:-900,y:0},{x:-900,y:-220},{x:-900,y:220},{x:-820,y:-440},{x:-820,y:440}],
  ffa: [{x:-960,y:-660},{x:960,y:-660},{x:-960,y:660},{x:960,y:660},{x:0,y:-860},{x:0,y:860},{x:-1100,y:0},{x:1100,y:0}]
};
const SKINS=['#60a5fa','#f87171','#4ade80','#fb923c','#c084fc','#22d3ee','#f97316','#f472b6','#a78bfa','#ffffff'];

// ═══ STATE ══════════════════════════════════════════════════
let peer, myId, myName='Player', myColor=SKINS[0];
let isHost=false, hostConn=null;
let clientConns={}, pendingPlayers={};
let gameStarted=false, gameConfig=null;
let myPlayerId=null;
let selectedMode='ffa';

const canvas=document.getElementById('gameCanvas');
const ctx=canvas.getContext('2d');
let W,H,CX,CY,camX=0,camY=0,animId=null;

const keys={}, mouse={x:300,y:300,down:false};
const inp={up:false,dn:false,lt:false,rt:false,angle:0,shoot:false};

let gs=null;           // authoritative game state (host only writes, clients read)
let renderState=null;  // what gets rendered
let particles=[];
let shownGO=false;
let _bid=0, _pid2=0, _ptimer=600, _wtimer=0;
let _teamToggle=false;

// ═══ UTILS ══════════════════════════════════════════════════
const hypot=(a,b,c,d)=>Math.hypot(a-c,b-d);
function crCol(cx,cy,cr,rx,ry,rw,rh){
  const nx=Math.max(rx-rw/2,Math.min(rx+rw/2,cx));
  const ny=Math.max(ry-rh/2,Math.min(ry+rh/2,cy));
  return Math.hypot(cx-nx,cy-ny)<cr;
}
function randSpawn(team){const arr=SPAWNS[team]||SPAWNS.ffa;return arr[Math.floor(Math.random()*arr.length)];}

// ═══ PEER SETUP ═════════════════════════════════════════════
function initPeer(){
  peer=new Peer(undefined,{debug:0});
  peer.on('open',id=>{
    myId=id;
    document.getElementById('peerStatus').textContent='Ready ✓';
    document.getElementById('myRoomId').textContent=id;
  });
  peer.on('connection',c=>{ if(isHost) onClientConn(c); });
  peer.on('error',e=>document.getElementById('peerStatus').textContent='Network error: '+e.type);
}

// ═══ HOST NETWORKING ════════════════════════════════════════
function onClientConn(conn){
  const pid=conn.peer;
  clientConns[pid]=conn;
  conn.on('data',d=>onHostRcv(pid,d));
  conn.on('close',()=>{
    delete clientConns[pid];
    delete pendingPlayers[pid];
    if(gs&&gs.players[pid]){ addKF('','',gs.players[pid].name+' disconnected');delete gs.players[pid]; }
    updateLobbyList();
  });
}
function onHostRcv(pid,d){
  if(!gameStarted){
    if(d.type==='join'){
      pendingPlayers[pid]={name:d.name,color:d.color};
      clientConns[pid].send({type:'joined',yourId:pid});
      updateLobbyList();
      setStatus('hostStatus',Object.keys(pendingPlayers).length+' player(s) joined');
    }
    return;
  }
  if(d.type==='input'&&gs?.players[pid]) gs.players[pid]._i=d;
}
function bcast(d){ Object.values(clientConns).forEach(c=>c.open&&c.send(d)); }
function setStatus(id,msg){ const el=document.getElementById(id);if(el)el.textContent=msg; }

// ═══ HOST GAME LOGIC ════════════════════════════════════════
function hostStartGame(){
  const kl=parseInt(document.getElementById('killLimitInput').value)||15;
  gameConfig={mode:selectedMode,killLimit:kl,lives:3};
  gameStarted=true; isHost=true;
  _teamToggle=false;

  gs={
    players:{}, bullets:[], pickups:[],
    flags:{
      red: {x:FLAG_BASE.red.x, y:FLAG_BASE.red.y, carriedBy:null,atBase:true,team:'red'},
      blue:{x:FLAG_BASE.blue.x,y:FLAG_BASE.blue.y,carriedBy:null,atBase:true,team:'blue'}
    },
    scores:{red:0,blue:0},
    killFeed:[],
    mode:selectedMode, killLimit:kl,
    timeLeft:300*60, gameOver:false, winner:null, frame:0
  };

  // Add host
  myPlayerId=myId;
  addPlayer(myId,myName,myColor);
  // Add waiting clients
  Object.entries(pendingPlayers).forEach(([pid,p])=>addPlayer(pid,p.name,p.color));

  bcast({type:'start',config:gameConfig});
  _ptimer=60; _wtimer=60; _bid=0; _pid2=0;
  launchGame();
}

function addPlayer(pid,name,color){
  const mode=gameConfig.mode;
  let team='none';
  if(mode==='tdm'||mode==='ctf'){ _teamToggle=!_teamToggle; team=_teamToggle?'red':'blue'; }
  const sp=randSpawn(team);
  gs.players[pid]={
    id:pid,name,color,team,
    x:sp.x,y:sp.y,angle:0,
    hp:100,weapon:'pistol',ammo:Infinity,
    kills:0,deaths:0,lives:gameConfig.lives,
    hasFlag:null,
    respawning:0,invincible:120,shotTimer:0,
    _i:{up:false,dn:false,lt:false,rt:false,angle:0,shoot:false}
  };
}

function hostUpdate(){
  if(!gameStarted||!gs||gs.gameOver) return;
  gs.frame++;

  // Spawn health packs
  _ptimer--;
  if(_ptimer<=0){
    _ptimer=600;
    HSPAWNS.forEach(p=>{
      if(!gs.pickups.find(pk=>pk.pt==='hp'&&hypot(pk.x,pk.y,p.x,p.y)<10))
        gs.pickups.push({id:_pid2++,x:p.x,y:p.y,pt:'hp'});
    });
  }
  // Spawn weapons
  _wtimer--;
  if(_wtimer<=0){
    _wtimer=900;
    const wn=['shotgun','sniper'];
    WSPAWNS.forEach((p,i)=>{
      if(!gs.pickups.find(pk=>pk.pt==='wp'&&hypot(pk.x,pk.y,p.x,p.y)<10))
        gs.pickups.push({id:_pid2++,x:p.x,y:p.y,pt:'wp',w:wn[i%2]});
    });
  }

  // Update players
  Object.values(gs.players).forEach(p=>{
    if(p.isBot) updateBotAI(p);
    if(p.respawning>0){p.respawning--;if(p.respawning===0)respawnP(p);return;}
    if(p.respawning===-1) return; // eliminated in LMS
    if(p.invincible>0) p.invincible--;
    if(p.shotTimer>0) p.shotTimer--;
    const i=p._i||{};

    // Move
    let dx=0,dy=0;
    if(i.up) dy-=PSPEED; if(i.dn) dy+=PSPEED;
    if(i.lt) dx-=PSPEED; if(i.rt) dx+=PSPEED;
    if(dx&&dy){dx*=.707;dy*=.707;}
    let nx=p.x+dx, ny=p.y+dy;
    WALLS.forEach(w=>{
      if(crCol(nx,p.y,PS,w.x,w.y,w.w,w.h)) nx=p.x;
      if(crCol(p.x,ny,PS,w.x,w.y,w.w,w.h)) ny=p.y;
    });
    p.x=Math.max(-AW/2+PS,Math.min(AW/2-PS,nx));
    p.y=Math.max(-AH/2+PS,Math.min(AH/2-PS,ny));
    p.angle=i.angle||0;

    // Shoot
    if(i.shoot&&p.shotTimer<=0&&p.ammo!==0){
      const w=WEAPONS[p.weapon];
      for(let j=0;j<w.pellets;j++){
        const a=p.angle+(Math.random()-.5)*w.spread;
        gs.bullets.push({id:_bid++,x:p.x+Math.cos(p.angle)*22,y:p.y+Math.sin(p.angle)*22,
          vx:Math.cos(a)*w.spd,vy:Math.sin(a)*w.spd,
          life:w.range,owner:p.id,team:p.team,dmg:w.dmg,color:w.color,wpn:p.weapon});
      }
      p.shotTimer=w.cd;
      if(p.ammo!==Infinity){p.ammo--;if(p.ammo<=0){p.weapon='pistol';p.ammo=Infinity;}}
    }

    // Pickups
    gs.pickups=gs.pickups.filter(pk=>{
      if(hypot(p.x,p.y,pk.x,pk.y)<PS+14){
        if(pk.pt==='hp'){p.hp=Math.min(100,p.hp+40);return false;}
        if(pk.pt==='wp'){p.weapon=pk.w;p.ammo=WEAPONS[pk.w].ammo;return false;}
      }
      return true;
    });

    // CTF
    if(gs.mode==='ctf'){
      Object.values(gs.flags).forEach(flag=>{
        if(!flag.carriedBy&&hypot(p.x,p.y,flag.x,flag.y)<PS+20){
          if(flag.team!==p.team&&!p.hasFlag){
            flag.carriedBy=p.id; p.hasFlag=flag.team;
            addKF(p.name,'',flag.team.toUpperCase()+' FLAG TAKEN');
          }
          if(flag.team===p.team&&!flag.atBase){
            flag.x=FLAG_BASE[flag.team].x;flag.y=FLAG_BASE[flag.team].y;flag.atBase=true;
            addKF(p.name,'',p.team.toUpperCase()+' FLAG RETURNED');
          }
        }
      });
      if(p.hasFlag){
        const ft=p.hasFlag;
        gs.flags[ft].x=p.x;gs.flags[ft].y=p.y;
        const mb=FLAG_BASE[p.team];
        if(mb&&hypot(p.x,p.y,mb.x,mb.y)<42){
          gs.scores[p.team]++;
          gs.flags[ft].x=FLAG_BASE[ft].x;gs.flags[ft].y=FLAG_BASE[ft].y;
          gs.flags[ft].carriedBy=null;gs.flags[ft].atBase=true;
          p.hasFlag=null;
          addKF(p.name,'',p.team.toUpperCase()+' CAPTURED FLAG! ('+gs.scores[p.team]+')');
          if(gs.scores[p.team]>=gs.killLimit) endGame(p.team==='red'?'Red Team':'Blue Team');
        }
      }
    }
  });

  // Bullets
  gs.bullets=gs.bullets.filter(b=>{
    b.x+=b.vx;b.y+=b.vy;b.life--;
    if(b.life<=0||Math.abs(b.x)>AW/2||Math.abs(b.y)>AH/2) return false;
    for(const w of WALLS) if(crCol(b.x,b.y,4,w.x,w.y,w.w,w.h)) return false;
    for(const p of Object.values(gs.players)){
      if(p.id===b.owner||p.respawning!==0||p.invincible>0) continue;
      if((gs.mode==='tdm'||gs.mode==='ctf')&&b.team===p.team&&b.team!=='none') continue;
      if(hypot(b.x,b.y,p.x,p.y)<PS+4){
        p.hp-=b.dmg;
        if(p.hp<=0){p.hp=0;killP(p,b.owner,b.wpn);}
        return false;
      }
    }
    return true;
  });

  // Time limit
  if(gs.mode==='ffa'||gs.mode==='tdm'){
    gs.timeLeft--;
    if(gs.timeLeft<=0){
      if(gs.mode==='ffa'){
        const top=Object.values(gs.players).sort((a,b)=>b.kills-a.kills)[0];
        endGame(top?top.name:'Nobody');
      } else {
        endGame(gs.scores.red>gs.scores.blue?'Red Team':'Blue Team');
      }
    }
  }
  if(gs.mode==='ffa'){const t=Object.values(gs.players).find(p=>p.kills>=gs.killLimit);if(t)endGame(t.name);}
  if(gs.mode==='tdm'){if(gs.scores.red>=gs.killLimit)endGame('Red Team');if(gs.scores.blue>=gs.killLimit)endGame('Blue Team');}
  if(isSolo) checkSurvivalWave();

  // LMS check
  if(gs.mode==='lms'||gs.mode==='survival'){
    const alive=Object.values(gs.players).filter(p=>p.lives>0&&p.respawning!==-1);
    if(gs.mode==='lms'&&alive.length<=1&&Object.keys(gs.players).length>1) endGame(alive.length?alive[0].name:'Nobody');
    if(gs.mode==='survival'){
      const me=gs.players[myPlayerId];
      if(me&&me.respawning===-1) endGame('Wave '+soloWave+' — Score: '+gs.soloScore);
    }
  }

  // Broadcast
  if(gs.frame%2===0){
    const snap={
      type:'state',
      players:Object.fromEntries(Object.entries(gs.players).map(([k,p])=>[k,{
        id:p.id,name:p.name,color:p.color,team:p.team,
        x:p.x,y:p.y,angle:p.angle,hp:p.hp,weapon:p.weapon,ammo:p.ammo,
        kills:p.kills,deaths:p.deaths,lives:p.lives,
        hasFlag:p.hasFlag,respawning:p.respawning,invincible:p.invincible,isBot:p.isBot
      }])),
      bullets:gs.bullets.map(b=>({id:b.id,x:b.x,y:b.y,color:b.color})),
      pickups:gs.pickups,
      flags:gs.flags,
      scores:gs.scores,
      killFeed:gs.killFeed.slice(0,8),
      mode:gs.mode,killLimit:gs.killLimit,
      timeLeft:gs.timeLeft,
      gameOver:gs.gameOver,winner:gs.winner,
      soloWave:gs.soloWave||1,soloScore:gs.soloScore||0
    };
    if(!isSolo) bcast(snap);
    renderState=snap;
  }
}

function killP(p,killerId,wpn){
  p.deaths++;
  if(p.hasFlag){
    const ft=p.hasFlag;
    gs.flags[ft].carriedBy=null;gs.flags[ft].atBase=false;
    p.hasFlag=null;
  }
  const killer=gs.players[killerId];
  if(killer){
    killer.kills++;
    if(gs.mode==='tdm')gs.scores[killer.team]=(gs.scores[killer.team]||0)+1;
    if(isSolo&&killer.id===myPlayerId) gs.soloScore=(gs.soloScore||0)+10*soloWave;
    addKF(killer.name,p.name,wpn||'pistol');
  } else {
    addKF('',p.name,'');
  }
  p.lives--;
  if(gs.mode==='lms'&&p.lives<=0) p.respawning=-1;
  else p.respawning=RESP_T;
}

function respawnP(p){
  const sp=randSpawn(p.team);
  p.x=sp.x;p.y=sp.y;p.hp=100;p.weapon='pistol';p.ammo=Infinity;p.invincible=120;p.hasFlag=null;
}

function addKF(killer,victim,wpn){
  gs.killFeed.unshift({killer,victim,wpn,t:gs.frame});
  if(gs.killFeed.length>10)gs.killFeed.pop();
}

function endGame(winner){gs.gameOver=true;gs.winner=winner;}

// ═══ CLIENT NETWORKING ══════════════════════════════════════
function joinGame(){
  const roomId=document.getElementById('joinInput').value.trim();
  if(!roomId){setStatus('joinStatus','Enter a Room ID first!');return;}
  setStatus('joinStatus','Connecting...');
  hostConn=peer.connect(roomId,{reliable:false,serialization:'json'});
  hostConn.on('open',()=>{
    hostConn.send({type:'join',name:myName,color:myColor});
    setStatus('joinStatus','Connected ✓');
    showScreen('screenWaiting');
  });
  hostConn.on('data',onClientRcv);
  hostConn.on('error',()=>setStatus('joinStatus','Connection failed. Check Room ID.'));
  hostConn.on('close',()=>setStatus('waitStatus','Disconnected from host.'));
}

function onClientRcv(d){
  if(d.type==='joined'){myPlayerId=d.yourId;}
  if(d.type==='start'){gameConfig=d.config;gameStarted=true;launchGame();}
  if(d.type==='state'){
    renderState=d;
    if(d.gameOver&&!shownGO)showGO(d.winner);
  }
}

function sendInput(){
  if(hostConn&&hostConn.open) hostConn.send({type:'input',...inp});
}

// ═══ GAME LOOP ══════════════════════════════════════════════
function launchGame(){
  showScreen('gameCanvas');
  myPlayerId=myPlayerId||myId;
  shownGO=false; renderState=null; particles=[];
  camX=0; camY=0;

  resize();
  canvas.addEventListener('mousemove',e=>{
    mouse.x=e.clientX;mouse.y=e.clientY;
    const p=renderState?.players?.[myPlayerId];
    const px=p?p.x:0, py=p?p.y:0;
    inp.angle=Math.atan2(mouse.y-CY+camY-py, mouse.x-CX+camX-px);
  });
  canvas.addEventListener('mousedown',()=>{mouse.down=true;inp.shoot=true;});
  canvas.addEventListener('mouseup',()=>{mouse.down=false;inp.shoot=false;});
  window.addEventListener('keydown',onKey);
  window.addEventListener('keyup',onKey);

  if(animId) cancelAnimationFrame(animId);
  loop();
}

function onKey(e){
  const d=e.type==='keydown';
  keys[e.code]=d;
  inp.up=!!(keys.KeyW||keys.ArrowUp);
  inp.dn=!!(keys.KeyS||keys.ArrowDown);
  inp.lt=!!(keys.KeyA||keys.ArrowLeft);
  inp.rt=!!(keys.KeyD||keys.ArrowRight);
  if(keys.Space!==undefined) inp.shoot=!!(keys.Space||mouse.down);
  if(['ArrowUp','ArrowDown','ArrowLeft','ArrowRight','Space'].includes(e.code)) e.preventDefault();
}

function loop(){
  if(isHost){
    // Sync host input
    const hpid=isSolo?myPlayerId:myId;
    if(gs?.players?.[hpid]) gs.players[hpid]._i={...inp};
    hostUpdate();
  } else {
    sendInput();
  }
  render();
  animId=requestAnimationFrame(loop);
}

// ═══ RENDERING ══════════════════════════════════════════════
function render(){
  const dpr=window.devicePixelRatio||1;
  ctx.clearRect(0,0,W,H);

  const st=renderState;
  if(!st){
    ctx.fillStyle='#60a5fa';ctx.font='20px Arial';ctx.textAlign='center';
    ctx.fillText('Waiting for game data...',CX,CY);
    return;
  }

  const me=st.players?.[myPlayerId];
  if(me&&me.respawning===0){
    camX+=(me.x-camX)*.1;
    camY+=(me.y-camY)*.1;
  }

  // Particles update
  for(let i=particles.length-1;i>=0;i--){
    const p=particles[i];
    p.x+=p.vx;p.y+=p.vy;p.vx*=.9;p.vy*=.9;p.life-=.025;
    if(p.life<=0)particles.splice(i,1);
  }

  ctx.save();
  ctx.translate(CX-camX,CY-camY);

  // Background
  ctx.fillStyle='#060c18';
  ctx.fillRect(-AW/2,-AH/2,AW,AH);

  // Grid
  ctx.strokeStyle='#0d1626';ctx.lineWidth=1;
  for(let x=-AW/2;x<=AW/2;x+=80){ctx.beginPath();ctx.moveTo(x,-AH/2);ctx.lineTo(x,AH/2);ctx.stroke();}
  for(let y=-AH/2;y<=AH/2;y+=80){ctx.beginPath();ctx.moveTo(-AW/2,y);ctx.lineTo(AW/2,y);ctx.stroke();}

  // Arena border
  ctx.strokeStyle='#1e3a5f';ctx.lineWidth=4;
  ctx.strokeRect(-AW/2,-AH/2,AW,AH);

  // Walls
  WALLS.forEach(w=>{
    ctx.fillStyle='#0f1f35';ctx.strokeStyle='#1e3a5f';ctx.lineWidth=2;
    ctx.fillRect(w.x-w.w/2,w.y-w.h/2,w.w,w.h);
    ctx.strokeRect(w.x-w.w/2,w.y-w.h/2,w.w,w.h);
  });

  // CTF bases
  if(st.mode==='ctf'){
    ['red','blue'].forEach(t=>{
      const b=FLAG_BASE[t];
      ctx.save();
      ctx.strokeStyle=t==='red'?'#ef444466':'#3b82f666';
      ctx.fillStyle=t==='red'?'#ef444411':'#3b82f611';
      ctx.lineWidth=2;ctx.setLineDash([6,4]);
      ctx.beginPath();ctx.arc(b.x,b.y,48,0,Math.PI*2);ctx.fill();ctx.stroke();
      ctx.setLineDash([]);
      ctx.restore();
      ctx.fillStyle=t==='red'?'#ef4444':'#60a5fa';
      ctx.font='bold 11px Arial';ctx.textAlign='center';
      ctx.fillText(t.toUpperCase()+' BASE',b.x,b.y+4);
    });
  }

  // Pickups
  (st.pickups||[]).forEach(pk=>{
    const pulse=Math.sin(Date.now()*.006)*3;
    const col=pk.pt==='hp'?'#4ade80':pk.w==='shotgun'?'#fb923c':'#67e8f9';
    const lbl=pk.pt==='hp'?'♥':pk.w==='shotgun'?'S':'R';
    ctx.save();ctx.translate(pk.x,pk.y);
    ctx.shadowBlur=14+pulse;ctx.shadowColor=col;ctx.fillStyle=col;
    ctx.beginPath();ctx.arc(0,0,11+pulse,0,Math.PI*2);ctx.fill();
    ctx.shadowBlur=0;ctx.fillStyle='#000';ctx.font='bold 11px Arial';
    ctx.textAlign='center';ctx.textBaseline='middle';
    ctx.fillText(lbl,0,0);
    ctx.restore();
  });

  // CTF flags
  if(st.flags) Object.values(st.flags).forEach(flag=>{
    if(flag.carriedBy) return;
    const pulse=Math.sin(Date.now()*.005)*3;
    const fc=flag.team==='red'?'#ef4444':'#60a5fa';
    ctx.save();ctx.translate(flag.x,flag.y);
    ctx.fillStyle=fc;ctx.shadowColor=fc;ctx.shadowBlur=14+pulse;
    ctx.fillRect(-3,-22,6,30);
    ctx.beginPath();ctx.moveTo(3,-22);ctx.lineTo(20,-14);ctx.lineTo(3,-6);ctx.closePath();ctx.fill();
    ctx.shadowBlur=0;ctx.restore();
  });

  // Particles
  particles.forEach(p=>{
    ctx.globalAlpha=Math.max(0,p.life);
    ctx.fillStyle=p.color;
    ctx.beginPath();ctx.arc(p.x,p.y,p.sz*p.life,0,Math.PI*2);ctx.fill();
  });
  ctx.globalAlpha=1;

  // Bullets
  (st.bullets||[]).forEach(b=>{
    ctx.save();ctx.fillStyle=b.color;ctx.shadowColor=b.color;ctx.shadowBlur=10;
    ctx.beginPath();ctx.arc(b.x,b.y,5,0,Math.PI*2);ctx.fill();
    ctx.restore();
  });

  // Players
  if(st.players) Object.values(st.players).forEach(p=>{
    if(p.respawning!==0){
      if(p.id===myPlayerId&&p.respawning>0){
        ctx.save();ctx.globalAlpha=.25;ctx.fillStyle=p.color;
        ctx.beginPath();ctx.arc(p.x,p.y,PS,0,Math.PI*2);ctx.fill();ctx.restore();
      }
      return;
    }
    if(p.invincible>0&&Math.floor(p.invincible/4)%2===1) return;
    const isMe=p.id===myPlayerId;

    ctx.save();ctx.translate(p.x,p.y);ctx.rotate(p.angle);

    // Body color: team color in TDM/CTF, else skin
    const bc=(st.mode==='tdm'||st.mode==='ctf')&&p.team!=='none'
      ?(p.team==='red'?'#ef4444':'#3b82f6') : p.color;
    ctx.fillStyle=bc;ctx.shadowColor=bc;ctx.shadowBlur=isMe?18:10;
    ctx.beginPath();ctx.arc(0,0,PS,0,Math.PI*2);ctx.fill();

    // Inner ring showing player's actual color
    ctx.shadowBlur=0;
    if(st.mode==='tdm'||st.mode==='ctf'){
      ctx.strokeStyle=p.color;ctx.lineWidth=2.5;
      ctx.beginPath();ctx.arc(0,0,PS-4,0,Math.PI*2);ctx.stroke();
    }

    // Barrel
    ctx.fillStyle='#e5e7eb';ctx.fillRect(PS*.4,-3.5,PS*1.2,7);

    // Eyes
    ctx.fillStyle='#111';
    ctx.beginPath();ctx.arc(5,-5,3.5,0,Math.PI*2);ctx.fill();
    ctx.beginPath();ctx.arc(5,5,3.5,0,Math.PI*2);ctx.fill();

    ctx.restore();

    // Flag on player
    if(p.hasFlag){
      const fc=p.hasFlag==='red'?'#ef4444':'#60a5fa';
      ctx.save();ctx.translate(p.x,p.y-PS-10);
      ctx.fillStyle=fc;
      ctx.fillRect(-3,-10,5,14);
      ctx.beginPath();ctx.moveTo(2,-10);ctx.lineTo(14,-5);ctx.lineTo(2,0);ctx.closePath();ctx.fill();
      ctx.restore();
    }

    // Name tag
    ctx.fillStyle=isMe?'#fff':'#d1d5db';
    ctx.font=`bold ${isMe?13:11}px Arial`;ctx.textAlign='center';
    ctx.fillText(p.name+(isMe?' ◀':'')+' '+p.kills, p.x, p.y-PS-18);

    // HP bar
    const bw=36;
    ctx.fillStyle='#1f2937';ctx.fillRect(p.x-bw/2,p.y-PS-14,bw,4);
    ctx.fillStyle=p.hp>50?'#22c55e':p.hp>25?'#f59e0b':'#ef4444';
    ctx.fillRect(p.x-bw/2,p.y-PS-14,bw*(p.hp/100),4);

    // Weapon label
    if(isMe&&p.weapon!=='pistol'){
      ctx.fillStyle='#fbbf24';ctx.font='10px Arial';
      ctx.fillText('['+p.weapon.toUpperCase()+' '+(p.ammo===Infinity?'∞':p.ammo)+']',p.x,p.y+PS+13);
    }
  });

  ctx.restore();

  // Crosshair
  ctx.strokeStyle='rgba(255,255,255,.65)';ctx.lineWidth=1.5;
  ctx.beginPath();
  ctx.moveTo(mouse.x-13,mouse.y);ctx.lineTo(mouse.x+13,mouse.y);
  ctx.moveTo(mouse.x,mouse.y-13);ctx.lineTo(mouse.x,mouse.y+13);
  ctx.stroke();
  ctx.beginPath();ctx.arc(mouse.x,mouse.y,5,0,Math.PI*2);ctx.stroke();

  drawHUD(st);
  if(st.gameOver&&!shownGO) showGO(st.winner);
}

function drawHUD(st){
  const me=st.players?.[myPlayerId];
  if(!me) return;

  // HP bar bottom left
  const hbW=Math.min(W*.24,220);
  ctx.fillStyle='#111827';ctx.fillRect(14,H-38,hbW,22);
  ctx.fillStyle=me.hp>50?'#22c55e':me.hp>25?'#f59e0b':'#ef4444';
  ctx.fillRect(14,H-38,hbW*(me.hp/100),22);
  ctx.strokeStyle='#374151';ctx.lineWidth=1;ctx.strokeRect(14,H-38,hbW,22);
  ctx.fillStyle='#fff';ctx.font='bold 13px Arial';ctx.textAlign='left';
  ctx.fillText('HP '+Math.ceil(me.hp),20,H-22);

  // Weapon
  const wclr=me.weapon==='sniper'?'#67e8f9':me.weapon==='shotgun'?'#fb923c':'#fde68a';
  ctx.fillStyle=wclr;ctx.font='bold 12px Arial';
  ctx.fillText('⚡ '+me.weapon.toUpperCase()+' '+(me.ammo===Infinity?'∞':me.ammo),14,H-46);

  // LMS lives
  if(st.mode==='lms'){
    ctx.fillStyle='#fbbf24';ctx.font='bold 13px Arial';
    ctx.fillText('❤ LIVES: '+me.lives,14,H-62);
  }

  // Scores / mode header
  ctx.textAlign='center';
  let scoreText='';
  if(st.mode==='ffa'){
    const sorted=Object.values(st.players||{}).sort((a,b)=>b.kills-a.kills);
    const top=sorted[0];
    scoreText=top?top.name+': '+top.kills+' kills | YOU: '+me.kills:'FFA';
  } else if(st.mode==='tdm'||st.mode==='ctf'){
    scoreText='🔴 '+( st.scores?.red||0)+'  VS  '+(st.scores?.blue||0)+' 🔵';
  } else if(st.mode==='lms'){
    const alive=Object.values(st.players||{}).filter(p=>p.lives>0&&p.respawning!==-1).length;
    scoreText=alive+' players alive';
  }

  ctx.fillStyle='rgba(0,0,0,.6)';
  const sw=ctx.measureText(scoreText).width+28;
  roundRect(ctx,CX-sw/2,6,sw,28,6,true,false);
  ctx.fillStyle='#f1f5f9';ctx.font='bold 15px Arial';
  ctx.fillText(scoreText,CX,25);

  ctx.fillStyle='#6b7280';ctx.font='11px Arial';
  ctx.fillText(st.mode?.toUpperCase()||'',CX,44);

  // Timer
  if(st.mode==='ffa'||st.mode==='tdm'){
    const s=Math.max(0,Math.ceil(st.timeLeft/60));
    const mm=Math.floor(s/60), ss=s%60;
    ctx.fillStyle='#9ca3af';ctx.font='bold 13px Arial';
    ctx.fillText('⏱ '+mm+':'+String(ss).padStart(2,'0'),CX,60);
  }

  // Kill feed top right
  ctx.textAlign='right';
  (st.killFeed||[]).slice(0,6).forEach((k,i)=>{
    const y=54+i*22;
    let txt='';
    if(k.killer) txt=k.killer+' ['+k.wpn+'] '+k.victim;
    else if(k.victim) txt=k.victim+' – '+k.wpn;
    else return;
    const tw=ctx.measureText(txt).width+16;
    ctx.fillStyle='rgba(0,0,0,.55)';
    roundRect(ctx,W-tw-10,y-15,tw+10,19,4,true,false);
    ctx.fillStyle=k.killer?'#e5e7eb':'#f87171';
    ctx.font='11px Arial';ctx.fillText(txt,W-10,y-1);
  });

  // Solo wave/score display
  if(isSolo&&(st.mode==='survival')){
    ctx.textAlign='left';
    ctx.fillStyle='rgba(0,0,0,.6)';roundRect(ctx,14,50,200,28,6,true,false);
    ctx.fillStyle='#fbbf24';ctx.font='bold 14px Arial';
    ctx.fillText('🌊 WAVE '+( st.soloWave||1)+'   ⭐ '+( st.soloScore||0),22,70);
  }
  if(isSolo&&st.mode!=='survival'){
    ctx.textAlign='left';
    ctx.fillStyle='rgba(0,0,0,.6)';roundRect(ctx,14,50,160,26,6,true,false);
    ctx.fillStyle='#4ade80';ctx.font='bold 13px Arial';
    ctx.fillText('⭐ SCORE: '+(st.soloScore||0),22,68);
  }
  if(me.respawning>0){
    ctx.textAlign='center';
    ctx.fillStyle='rgba(0,0,0,.72)';roundRect(ctx,CX-130,CY-35,260,60,10,true,false);
    ctx.fillStyle='#ef4444';ctx.font='bold 20px Arial';
    ctx.fillText('Respawning in '+Math.ceil(me.respawning/60)+'s...',CX,CY+7);
  }
  if(me.respawning===-1){
    ctx.textAlign='center';
    ctx.fillStyle='rgba(0,0,0,.72)';roundRect(ctx,CX-130,CY-35,260,60,10,true,false);
    ctx.fillStyle='#ef4444';ctx.font='bold 22px Arial';ctx.fillText('ELIMINATED',CX,CY+7);
  }
}

function roundRect(c,x,y,w,h,r,fill,stroke){
  c.beginPath();c.moveTo(x+r,y);c.lineTo(x+w-r,y);c.quadraticCurveTo(x+w,y,x+w,y+r);
  c.lineTo(x+w,y+h-r);c.quadraticCurveTo(x+w,y+h,x+w-r,y+h);c.lineTo(x+r,y+h);
  c.quadraticCurveTo(x,y+h,x,y+h-r);c.lineTo(x,y+r);c.quadraticCurveTo(x,y,x+r,y);c.closePath();
  if(fill) c.fill();if(stroke) c.stroke();
}

function showGO(winner){
  shownGO=true;
  document.getElementById('overlayTitle').textContent='GAME OVER';
  document.getElementById('overlayMsg').textContent='🏆 Winner: '+winner;
  document.getElementById('overlay').style.display='flex';
}

// ═══ SOLO MODE ══════════════════════════════════════════════
let isSolo=false, soloMode='ffa', soloBotDiff='medium', soloBotCount=4;
let soloWave=1, soloWaveTimer=0;

const BOT_NAMES=['Zephyr','Toxin','Blaze','Reaper','Viper','Ghost','Havoc','Storm','Kira','Neon'];
const BOT_COLORS=['#f87171','#fb923c','#4ade80','#c084fc','#f472b6','#22d3ee','#a78bfa','#fbbf24','#6ee7b7','#f9a8d4'];

const DIFF={
  easy:  {speed:2.2, cd:55, spread:.35, aimErr:.55, sight:420, react:80},
  medium:{speed:3.0, cd:35, spread:.18, aimErr:.28, sight:580, react:45},
  hard:  {speed:3.8, cd:18, spread:.06, aimErr:.08, sight:800, react:15},
};

function goSolo(){
  myName=document.getElementById('nameInput').value.trim()||'Player';
  showScreen('screenSolo');
}

let _soloModeSelected='ffa';
function selectSoloMode(mode,el){
  _soloModeSelected=mode;
  document.querySelectorAll('#screenSolo .mode-btn').forEach(b=>b.classList.remove('selected'));
  el.classList.add('selected');
}

function startSolo(){
  isSolo=true; isHost=true;
  soloMode=_soloModeSelected;
  soloBotDiff=document.getElementById('botDiff').value;
  soloBotCount=parseInt(document.getElementById('botCountInput').value)||4;
  gameConfig={mode:soloMode,killLimit:20,lives:3};
  gameStarted=true;
  _teamToggle=false; _bid=0; _pid2=0; _ptimer=600; _wtimer=0; soloWave=1; soloWaveTimer=300;

  gs={
    players:{}, bullets:[], pickups:[],
    flags:{
      red:{x:FLAG_BASE.red.x,y:FLAG_BASE.red.y,carriedBy:null,atBase:true,team:'red'},
      blue:{x:FLAG_BASE.blue.x,y:FLAG_BASE.blue.y,carriedBy:null,atBase:true,team:'blue'}
    },
    scores:{red:0,blue:0},
    killFeed:[],
    mode:soloMode, killLimit:20,
    timeLeft:300*60, gameOver:false, winner:null, frame:0,
    soloWave:1, soloScore:0
  };

  myPlayerId='solo_player';
  // Add human player
  const sp=soloMode==='tdm'?SPAWNS.blue[0]:randSpawn('ffa');
  gs.players[myPlayerId]={
    id:myPlayerId,name:myName,color:myColor,
    team:soloMode==='tdm'?'blue':'none',
    x:sp.x,y:sp.y,angle:0,
    hp:100,weapon:'pistol',ammo:Infinity,
    kills:0,deaths:0,lives:gameConfig.lives,
    hasFlag:null, respawning:0, invincible:120, shotTimer:0,
    isBot:false,
    _i:{up:false,dn:false,lt:false,rt:false,angle:0,shoot:false}
  };

  // Add bots
  spawnBots(soloBotCount);
  launchGame();
}

function spawnBots(count){
  for(let i=0;i<count;i++){
    const bname=BOT_NAMES[i%BOT_NAMES.length];
    const bcolor=BOT_COLORS[i%BOT_COLORS.length];
    const bteam=soloMode==='tdm'?'red':'none';
    const sp=soloMode==='tdm'?SPAWNS.red[i%SPAWNS.red.length]:randSpawn('ffa');
    const bid='bot_'+i+'_'+Date.now();
    gs.players[bid]={
      id:bid,name:bname,color:bcolor,team:bteam,
      x:sp.x,y:sp.y,angle:0,
      hp:100,weapon:'pistol',ammo:Infinity,
      kills:0,deaths:0,lives:gameConfig.lives,
      hasFlag:null, respawning:0, invincible:80, shotTimer:0,
      isBot:true,
      _botTarget:null,_botTimer:0,_botWander:{x:0,y:0},
      _i:{up:false,dn:false,lt:false,rt:false,angle:0,shoot:false}
    };
  }
}

function updateBotAI(bot){
  if(bot.respawning!==0||bot.respawning===-1) return;
  const diff=DIFF[soloBotDiff];
  bot._botTimer=(bot._botTimer||0)-1;

  // Find nearest enemy
  const targets=Object.values(gs.players).filter(p=>{
    if(p.id===bot.id||p.respawning!==0) return false;
    if(soloMode==='tdm'&&p.team===bot.team) return false;
    return true;
  });

  let target=null, minD=diff.sight;
  targets.forEach(t=>{
    const d=hypot(bot.x,bot.y,t.x,t.y);
    if(d<minD){minD=d;target=t;}
  });

  const i={up:false,dn:false,lt:false,rt:false,angle:bot.angle,shoot:false};

  if(target){
    // Aim with error
    const err=(Math.random()-.5)*diff.aimErr;
    const ang=Math.atan2(target.y-bot.y,target.x-bot.x)+err;
    i.angle=ang;

    // Move toward target if far, strafe if close
    const dist=hypot(bot.x,bot.y,target.x,target.y);
    if(dist>160){
      i.up=Math.sin(ang)<-.2;i.dn=Math.sin(ang)>.2;
      i.lt=Math.cos(ang)<-.2;i.rt=Math.cos(ang)>.2;
    } else {
      // Strafe
      const sa=ang+Math.PI/2;
      const sf=Math.sin(bot._botTimer*.04)>.0?1:-1;
      i.lt=Math.cos(sa)*sf<-.2;i.rt=Math.cos(sa)*sf>.2;
      i.up=Math.sin(sa)*sf<-.2;i.dn=Math.sin(sa)*sf>.2;
    }

    // Shoot after reaction delay
    if(minD<diff.sight&&bot._botTimer<=-diff.react) i.shoot=true;

    // Seek health packs when low
    if(bot.hp<40){
      const hp=gs.pickups.find(p=>p.pt==='hp');
      if(hp){
        const ha=Math.atan2(hp.y-bot.y,hp.x-bot.x);
        i.up=Math.sin(ha)<-.2;i.dn=Math.sin(ha)>.2;
        i.lt=Math.cos(ha)<-.2;i.rt=Math.cos(ha)>.2;
        i.angle=ha;i.shoot=false;
      }
    }
  } else {
    // Wander
    if(bot._botTimer<=0){
      bot._botTimer=80+Math.random()*120;
      bot._botWander={x:(Math.random()-.5)*AW*.7,y:(Math.random()-.5)*AH*.7};
    }
    const wa=Math.atan2(bot._botWander.y-bot.y,bot._botWander.x-bot.x);
    i.angle=wa;
    i.up=Math.sin(wa)<-.15;i.dn=Math.sin(wa)>.15;
    i.lt=Math.cos(wa)<-.15;i.rt=Math.cos(wa)>.15;
  }

  bot._i=i;
}

// Survival wave spawner
function checkSurvivalWave(){
  if(soloMode!=='survival') return;
  const bots=Object.values(gs.players).filter(p=>p.isBot&&p.respawning===0&&p.respawning!==-1);
  soloWaveTimer--;
  if(bots.length===0||soloWaveTimer<=0){
    soloWave++;
    gs.soloWave=soloWave;
    const newCount=Math.min(2+soloWave,12);
    soloWaveTimer=300;
    for(let i=0;i<newCount;i++){
      const idx=Object.keys(gs.players).length;
      const sp=randSpawn('ffa');
      gs.players['bot_w'+soloWave+'_'+i]={
        id:'bot_w'+soloWave+'_'+i,
        name:BOT_NAMES[i%BOT_NAMES.length],
        color:BOT_COLORS[i%BOT_COLORS.length],team:'none',
        x:sp.x,y:sp.y,angle:0,
        hp:80+soloWave*10,weapon:soloWave>3?'shotgun':soloWave>6?'sniper':'pistol',
        ammo:Infinity,kills:0,deaths:0,lives:1,
        hasFlag:null,respawning:0,invincible:80,shotTimer:0,
        isBot:true,_botTarget:null,_botTimer:0,_botWander:{x:0,y:0},
        _i:{up:false,dn:false,lt:false,rt:false,angle:0,shoot:false}
      };
    }
    addKF('','','⚠ WAVE '+soloWave+' — '+newCount+' enemies!');
  }
}

// ═══ UI HELPERS ═══════════════════════════════════════════
function showScreen(id){
  document.querySelectorAll('.screen').forEach(s=>s.style.display='none');
  document.getElementById('gameCanvas').style.display='none';
  const el=document.getElementById(id);
  if(el){
    if(id==='gameCanvas') el.style.display='block';
    else el.style.display='flex';
  }
}

function goHost(){
  myName=document.getElementById('nameInput').value.trim()||'Player';
  isHost=true;
  showScreen('screenHost');
  updateLobbyList();
}

function goJoin(){
  myName=document.getElementById('nameInput').value.trim()||'Player';
  showScreen('screenJoin');
}

function selectMode(mode,el){
  selectedMode=mode;
  document.querySelectorAll('.mode-btn').forEach(b=>b.classList.remove('selected'));
  el.classList.add('selected');
  const lbl=document.getElementById('limitLabel');
  if(mode==='ctf') lbl.textContent='Flag Captures to Win:';
  else if(mode==='lms') lbl.textContent='Starting Lives:';
  else lbl.textContent='Kill Limit:';
}

function copyRoom(){
  navigator.clipboard.writeText(myId).then(()=>setStatus('hostStatus','Room ID copied! Share with friends.'));
}

function updateLobbyList(){
  const el=document.getElementById('playerList');if(!el)return;
  const names=[myName+'(Host)',...Object.values(pendingPlayers).map(p=>p.name)];
  el.innerHTML=names.map(n=>`<li>• ${n}</li>`).join('');
}

function buildColorPicker(){
  const cp=document.getElementById('colorPicker');
  SKINS.forEach((c,i)=>{
    const btn=document.createElement('button');
    btn.style.cssText=`width:30px;height:30px;border-radius:50%;background:${c};border:3px solid ${i===0?'#fff':'transparent'};cursor:pointer;`;
    btn.onclick=()=>{
      myColor=c;
      document.querySelectorAll('#colorPicker button').forEach(b=>b.style.border='3px solid transparent');
      btn.style.border='3px solid #fff';
    };
    cp.appendChild(btn);
  });
}

function resize(){
  W=canvas.width=window.innerWidth;
  H=canvas.height=window.innerHeight;
  CX=W/2;CY=H/2;
}
window.addEventListener('resize',resize);

function toggleFS(){
  const el=document.documentElement;
  if(!document.fullscreenElement)(el.requestFullscreen||el.webkitRequestFullscreen).call(el);
  else(document.exitFullscreen||document.webkitExitFullscreen).call(document);
}

// ═══ BOOT ═══════════════════════════════════════════════════
window.addEventListener('load',()=>{
  buildColorPicker();
  initPeer();
  resize();
});
</script>
</body>
</html>
