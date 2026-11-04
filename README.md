[index.html](https://github.com/user-attachments/files/34380298/index.html)
<!DOCTYPE html>
<html lang="en"><head><meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Sky Dash</title>
<link href="https://fonts.googleapis.com/css2?family=Fredoka:wght@500;700&display=swap" rel="stylesheet">
<style>
html,body{margin:0;height:100%;overflow:hidden;font-family:'Fredoka',system-ui,sans-serif;color:#4a2c5a;touch-action:none;user-select:none;-webkit-user-select:none;
background:linear-gradient(#4fc3f7 0%,#9be0f5 34%,#ffd6b3 60%,#ff9ec4 100%)}
canvas{position:fixed;inset:0;width:100%;height:100%;display:block}
#hud{position:fixed;left:0;right:0;top:0;display:flex;justify-content:space-between;gap:8px;padding:calc(env(safe-area-inset-top,0px) + 12px) 14px 0;pointer-events:none}
.pill{background:#fff4e0;border:3px solid #4a2c5a;border-radius:999px;padding:4px 16px;font-weight:700;font-size:22px;box-shadow:0 4px 0 #4a2c5a}
#hearts{color:#ff4d6d;letter-spacing:2px}
#lvl{background:#ffc23d}
#toast{position:fixed;left:50%;top:22%;transform:translateX(-50%);font-size:clamp(26px,6vw,44px);font-weight:700;color:#fff4e0;-webkit-text-stroke:2px #4a2c5a;paint-order:stroke fill;text-shadow:0 5px 0 #4a2c5a;opacity:0;pointer-events:none;white-space:nowrap}
#toast.show{animation:pop 1.1s ease-out}
@keyframes pop{0%{opacity:0;transform:translate(-50%,10px) scale(.6)}15%{opacity:1;transform:translate(-50%,0) scale(1.1)}70%{opacity:1}100%{opacity:0;transform:translate(-50%,-20px) scale(1)}}
.panel{position:fixed;left:50%;top:50%;transform:translate(-50%,-50%);width:min(88vw,380px);box-sizing:border-box;background:#fff4e0;border:4px solid #4a2c5a;border-radius:28px;box-shadow:0 8px 0 #4a2c5a;padding:26px 24px;text-align:center}
.panel h1{margin:0 0 6px;font-size:clamp(44px,12vw,60px);line-height:1;color:#ff6fa5;-webkit-text-stroke:2px #4a2c5a;paint-order:stroke fill;text-shadow:0 4px 0 #4a2c5a}
.panel p{margin:8px 0;font-size:18px;font-weight:500}
.panel small{display:block;margin-top:14px;font-size:14px;opacity:.75}
.big{font-size:44px;font-weight:700;margin:4px 0}
button{font:inherit;font-size:24px;font-weight:700;color:#4a2c5a;background:#4ddbb4;border:4px solid #4a2c5a;border-radius:999px;padding:8px 36px;margin-top:10px;cursor:pointer;box-shadow:0 5px 0 #4a2c5a;transition:transform .08s,box-shadow .08s}
button:active{transform:translateY(4px);box-shadow:0 1px 0 #4a2c5a}
button:focus-visible{outline:4px solid #ffc23d;outline-offset:3px}
.hidden{display:none}
</style></head>
<body>
<div id="hud"><div class="pill" id="score">0 m</div><div class="pill" id="lvl">LV 1</div><div class="pill" id="hearts">♥♥♥</div></div>
<div id="toast"></div>
<div class="panel" id="menu"><h1>Sky Dash</h1><p>Dodge the grumpy birds flying straight at you!</p><button id="play">Play</button><small>Move mouse or finger to steer.<br>Keyboard: arrows or WASD.</small></div>
<div class="panel hidden" id="over"><h1 style="font-size:40px">Bonked!</h1><div class="big" id="fs">0 m</div><p id="fb">Best: 0 m</p><button id="again">Fly again</button></div>
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
const $=id=>document.getElementById(id),R=(a,b)=>a+Math.random()*(b-a),clamp=(v,a,b)=>Math.max(a,Math.min(b,v));
const YMIN=-1,YMAX=6.5;let XR=9;
const renderer=new THREE.WebGLRenderer({antialias:true,alpha:true});
renderer.setPixelRatio(Math.min(devicePixelRatio,2));document.body.prepend(renderer.domElement);
const scene=new THREE.Scene();const FOG=new THREE.Color(0xffd6b3);scene.fog=new THREE.Fog(FOG,40,150);
const cam=new THREE.PerspectiveCamera(62,1,.1,320);
scene.add(new THREE.HemisphereLight(0xfff1e0,0xff9ec4,.95));
const dl=new THREE.DirectionalLight(0xfff0cc,.85);dl.position.set(-6,10,8);scene.add(dl);
const sunM=new THREE.Mesh(new THREE.SphereGeometry(24,24,16),new THREE.MeshBasicMaterial({color:0xffe08a,fog:false}));sunM.position.set(40,22,-250);scene.add(sunM);
const sea=new THREE.Mesh(new THREE.PlaneGeometry(700,700),new THREE.MeshLambertMaterial({color:0x7fe3c9}));sea.rotation.x=-Math.PI/2;sea.position.y=-14;scene.add(sea);
function resize(){const w=innerWidth,h=innerHeight;renderer.setSize(w,h);cam.aspect=w/h;cam.updateProjectionMatrix();XR=Math.min(9,5.1*cam.aspect*1.9)}
addEventListener('resize',resize);resize();

/* ---------- birds ---------- */
const mats=new Map(),mat=c=>{if(!mats.has(c))mats.set(c,new THREE.MeshLambertMaterial({color:c,flatShading:true}));return mats.get(c)};
const G={body:new THREE.SphereGeometry(.6,10,8),head:new THREE.SphereGeometry(.38,10,8),beak:new THREE.ConeGeometry(.14,.4,6),eye:new THREE.SphereGeometry(.11,8,6),pup:new THREE.SphereGeometry(.055,6,6),brow:new THREE.BoxGeometry(.3,.06,.06),tail:new THREE.BoxGeometry(.5,.08,.6),wing:new THREE.BoxGeometry(1.5,.08,.85),cheek:new THREE.SphereGeometry(.09,6,6),crest:new THREE.ConeGeometry(.1,.3,5)};
function makeBird(c,angry){
  const g=new THREE.Group();
  const add=(geo,col,x,y,z,p=g)=>{const m=new THREE.Mesh(geo,mat(col));m.position.set(x,y,z);p.add(m);return m};
  add(G.body,c.body,0,0,0).scale.set(1,.9,1.3);
  add(G.head,c.body,0,.35,-.75);
  add(G.beak,0xffa21f,0,.3,-1.2).rotation.x=-Math.PI/2;
  add(G.tail,c.wing,0,.1,.95).rotation.x=.3;
  add(G.crest,c.wing,0,.78,-.7);
  [-1,1].forEach(s=>{
    add(G.eye,0xffffff,s*.2,.5,-1.0);add(G.pup,0x222222,s*.2,.5,-1.09);
    if(angry)add(G.brow,0x2a1030,s*.2,.68,-1.05).rotation.z=s*.55;
    else add(G.cheek,0xff8fb1,s*.3,.3,-1.0);
  });
  const w=[-1,1].map(s=>{const p=new THREE.Group();p.position.set(s*.5,.15,0);g.add(p);add(G.wing,c.wing,s*.75,0,0,p);return p});
  g.userData.w=w;return g;
}
function flap(b,t,sp){const a=Math.sin(t*sp)*.75;b.userData.w[0].rotation.z=-a;b.userData.w[1].rotation.z=a}
const player=makeBird({body:0xffc23d,wing:0xff7a3d},false);player.scale.setScalar(1.05);scene.add(player);
const PAL=[{body:0xb14dff,wing:0x6a2bd1},{body:0xff4d6d,wing:0xb3203f},{body:0x2db6d6,wing:0x1a5f9e},{body:0x7a5cff,wing:0x3a2a99}];

/* ---------- scenery ---------- */
const clouds=[],cmat=new THREE.MeshLambertMaterial({color:0xfff6f0,flatShading:true});
for(let i=0;i<20;i++){const c=new THREE.Group();for(let j=0;j<4;j++){const s=new THREE.Mesh(new THREE.SphereGeometry(R(2,3.5),7,6),cmat);s.position.set(j*2.6-4,R(-.6,.8),R(-1,1));s.scale.y=.7;c.add(s)}
  placeCloud(c,R(-170,20));scene.add(c);clouds.push(c)}
function placeCloud(c,z){let x=R(-55,55),y=R(-8,22);if(Math.abs(x)<14&&y>-4&&y<12)y=Math.random()<.5?R(13,22):R(-10,-5);c.position.set(x,y,z)}
const streaks=[],smat=new THREE.MeshBasicMaterial({color:0xffffff,transparent:true,opacity:.55});
for(let i=0;i<26;i++){const m=new THREE.Mesh(new THREE.BoxGeometry(.05,.05,7),smat);m.position.set(R(-20,20),R(-6,12),R(-140,10));scene.add(m);streaks.push(m)}

/* ---------- particles ---------- */
const NP=520,pp=new Float32Array(NP*3).fill(-999),pc=new Float32Array(NP*3),pv=new Float32Array(NP*3),pl=new Float32Array(NP);let pi=0;
const pg=new THREE.BufferGeometry();pg.setAttribute('position',new THREE.BufferAttribute(pp,3));pg.setAttribute('color',new THREE.BufferAttribute(pc,3));
const pts=new THREE.Points(pg,new THREE.PointsMaterial({size:.5,vertexColors:true,fog:false}));pts.frustumCulled=false;scene.add(pts);
const SPARK=[0xffe14d,0xff6fa5,0x4ddbb4,0xffffff,0xff8a3d];
function emit(x,y,z,cols,n,sp,life){for(let i=0;i<n;i++){const k=pi++%NP,c=new THREE.Color(Array.isArray(cols)?cols[i%cols.length]:cols);
  const a=R(0,6.283),b=R(-1,1),r=Math.sqrt(1-b*b),s=R(.3,1)*sp;
  pp[k*3]=x;pp[k*3+1]=y;pp[k*3+2]=z;pv[k*3]=Math.cos(a)*r*s;pv[k*3+1]=b*s;pv[k*3+2]=Math.sin(a)*r*s;
  pc[k*3]=c.r;pc[k*3+1]=c.g;pc[k*3+2]=c.b;pl[k]=life*R(.6,1)}}
function updP(dt,sp){for(let k=0;k<NP;k++){if(pl[k]<=0)continue;pl[k]-=dt;
  if(pl[k]<=0){pp[k*3+1]=-999;continue}
  const d=1-2.2*dt;pv[k*3]*=d;pv[k*3+1]=pv[k*3+1]*d-2.5*dt;pv[k*3+2]*=d;
  pp[k*3]+=pv[k*3]*dt;pp[k*3+1]+=pv[k*3+1]*dt;pp[k*3+2]+=(pv[k*3+2]+sp)*dt;
  const f=Math.min(1,dt*2.5);pc[k*3]+=(FOG.r-pc[k*3])*f;pc[k*3+1]+=(FOG.g-pc[k*3+1])*f;pc[k*3+2]+=(FOG.b-pc[k*3+2])*f}
  pg.attributes.position.needsUpdate=true;pg.attributes.color.needsUpdate=true}

/* ---------- game state ---------- */
let state='menu',T=0,t=0,dist=0,lives=3,level=1,spawnT=1,invuln=0,shake=0,best=0,trailT=0,speed=16,lastScore=-1;
try{best=+localStorage.getItem('skydash-best')||0}catch(e){}
const pos=new THREE.Vector3(0,2,0),vel=new THREE.Vector3(),target=new THREE.Vector3(0,2,0),enemies=[],keys={};
function toast(m){const e=$('toast');e.textContent=m;e.classList.remove('show');void e.offsetWidth;e.classList.add('show')}
function hud(){const s=Math.floor(dist);if(s!==lastScore){$('score').textContent=s+' m';lastScore=s}
  $('lvl').textContent='LV '+level;$('hearts').textContent='♥'.repeat(lives)+'♡'.repeat(3-lives)}
function reset(){enemies.forEach(e=>scene.remove(e.m));enemies.length=0;lives=3;t=0;dist=0;level=1;spawnT=1.2;invuln=0;pos.set(0,2,0);vel.set(0,0,0);target.set(0,2,0);
  player.visible=true;state='play';$('menu').classList.add('hidden');$('over').classList.add('hidden');hud()}
$('play').onclick=$('again').onclick=reset;
addEventListener('keydown',e=>{keys[e.code]=1;if((e.code==='Space'||e.code==='Enter')&&state!=='play'&&!(state==='over'&&$('over').classList.contains('hidden')))reset()});
addEventListener('keyup',e=>keys[e.code]=0);
function aim(e){if(state!=='play')return;target.x=(e.clientX/innerWidth*2-1)*XR;target.y=YMIN+(1-e.clientY/innerHeight)*(YMAX-YMIN)}
addEventListener('pointermove',aim);addEventListener('pointerdown',aim);

function spawn(){
  const n=level>=7?1+(Math.random()*3|0):(level>=3&&Math.random()<.5?2:1);
  const aimed=Math.random()<Math.min(.75,.25+level*.07);
  for(let i=0;i<n;i++){
    const m=makeBird(PAL[Math.random()*PAL.length|0],true);m.rotation.y=Math.PI;m.scale.setScalar(1.15);
    const x=aimed?pos.x+(i-(n-1)/2)*3.4+R(-1,1):R(-XR,XR),y=aimed?pos.y+R(-1,1):R(YMIN,YMAX);
    const e={m,x,y:clamp(y,YMIN,YMAX),z:-150-i*R(0,8),sp:R(0,5)+level*.7,amp:level>=3&&Math.random()<.6?R(.8,2.6):0,f:R(1,2.4),ph:R(0,6.3),hom:Math.max(0,(level-4)*.1),passed:false,near:1e9,bx:x};
    scene.add(m);enemies.push(e)}
}
function hit(e){
  lives--;shake=.8;invuln=1.6;emit(pos.x,pos.y,pos.z,[0xffc23d,0xff7a3d,0xff4d6d,0xffffff],60,9,1.1);
  emit(e.x,e.y,e.z,[e.m.children[0].material.color.getHex(),0xffffff],35,8,1);
  scene.remove(e.m);enemies.splice(enemies.indexOf(e),1);
  if(lives<=0){state='over';player.visible=false;emit(pos.x,pos.y,pos.z,SPARK,90,12,1.4);
    if(dist>best){best=Math.floor(dist);try{localStorage.setItem('skydash-best',best)}catch(e){}}
    $('fs').textContent=Math.floor(dist)+' m';$('fb').textContent='Best: '+Math.floor(best)+' m';
    setTimeout(()=>$('over').classList.remove('hidden'),800)}
  hud()}

function update(dt){
  const goal=state==='play'?Math.min(24+level*2.6,58):16;speed+=(goal-speed)*Math.min(1,dt*1.5);
  if(state==='play'){
    t+=dt;dist+=dt*speed*.6;const L=1+Math.floor(t/12);
    if(L>level){level=L;toast('Level '+level+'!');emit(pos.x,pos.y,pos.z-1,SPARK,50,8,1.2)}
    if((spawnT-=dt)<=0){spawn();spawnT=Math.max(.38,1.25-level*.085)*R(.8,1.2)}
    const kx=(keys.ArrowRight||keys.KeyD?1:0)-(keys.ArrowLeft||keys.KeyA?1:0),ky=(keys.ArrowUp||keys.KeyW?1:0)-(keys.ArrowDown||keys.KeyS?1:0);
    target.x=clamp(target.x+kx*dt*16,-XR,XR);target.y=clamp(target.y+ky*dt*12,YMIN,YMAX);
    hud()}
  else if(state==='menu'){target.set(Math.sin(T*.8)*Math.min(3,XR*.5),2+Math.sin(T*1.3)*1.5,0)}
  /* spring physics: slightly underdamped for a floaty feel */
  if(state!=='over'){for(const a of['x','y']){const acc=(target[a]-pos[a])*20-vel[a]*7;vel[a]+=acc*dt;pos[a]+=vel[a]*dt}}
  pos.y=clamp(pos.y,YMIN-.6,YMAX+.6);
  invuln=Math.max(0,invuln-dt);
  player.position.set(pos.x,pos.y+Math.sin(T*3)*.08,0);
  player.rotation.z+=((-vel.x*.07)-player.rotation.z)*Math.min(1,dt*10);
  player.rotation.x+=((vel.y*.05)-player.rotation.x)*Math.min(1,dt*10);
  player.rotation.y=-vel.x*.02;
  flap(player,T,14+Math.max(0,vel.y)*1.5);
  if(state!=='over')player.visible=invuln<=0||Math.floor(T*16)%2===0;
  /* feather trail */
  if(state!=='over'&&(trailT-=dt)<=0){trailT=.03;emit(pos.x,pos.y,1.1,[0xfff4e0,0xffc23d,0xff8fb1],1,1.2,.7)}
  /* enemies */
  for(let i=enemies.length-1;i>=0;i--){const e=enemies[i];
    e.z+=(speed+e.sp)*dt;
    if(e.hom&&e.z<-25)e.y+=(pos.y-e.y)*Math.min(1,e.hom*dt);
    e.x=e.bx+Math.sin(T*e.f+e.ph)*e.amp;
    e.m.position.set(e.x,e.y,e.z);flap(e.m,T+e.ph,12);e.m.rotation.z=Math.cos(T*e.f+e.ph)*e.amp*.08;
    const dx=e.x-pos.x,dy=e.y-pos.y,dz=e.z;
    if(state==='play'){
      if(Math.abs(dz)<2.5)e.near=Math.min(e.near,Math.hypot(dx,dy));
      if(invuln<=0&&Math.abs(dz)<1.4&&dx*dx+dy*dy<1.6*1.6){hit(e);continue}
      if(!e.passed&&e.z>2.5){e.passed=true;if(e.near<3.3&&invuln<=0){dist+=25;toast('Close call! +25');emit(pos.x,pos.y,0,SPARK,26,7,.9)}}}
    if(e.z>14){scene.remove(e.m);enemies.splice(i,1)}}
  /* scenery */
  for(const c of clouds){c.position.z+=speed*.6*dt;if(c.position.z>25)placeCloud(c,-170)}
  for(const s of streaks){s.position.z+=speed*1.4*dt;if(s.position.z>12){s.position.set(R(-20,20),R(-6,12),-140);if(Math.abs(s.position.x)<3)s.position.x+=6}}
  updP(dt,speed);
  /* camera */
  shake*=Math.exp(-dt*5);
  cam.position.x+=(pos.x*.55-cam.position.x)*Math.min(1,dt*4);
  cam.position.y+=(pos.y*.7+1.2-cam.position.y)*Math.min(1,dt*4);
  cam.position.z=8;cam.position.x+=R(-1,1)*shake*.3;cam.position.y+=R(-1,1)*shake*.3;
  cam.lookAt(pos.x*.3,pos.y*.6+.5,-20);
  const fov=62+(speed-16)*.18;if(Math.abs(fov-cam.fov)>.05){cam.fov=fov;cam.updateProjectionMatrix()}
}
let last=performance.now();
(function loop(now){requestAnimationFrame(loop);const dt=Math.min((now-last)/1000,.05);last=now;T+=dt;update(dt);renderer.render(scene,cam)})(last);
</script>
</body></html>
