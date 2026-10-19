# kamal-sir-chemistry-lab
Chemistry 10th Class Virtual Lab

<!DOCTYPE html>
<html lang="hi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Kamal Sir Chemistry Virtual Lab</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#101c2b;color:white;font-family:Arial,sans-serif}
header{padding:14px;background:#123d59;text-align:center}
h2{margin:4px;font-size:21px}
p{margin:6px;font-size:13px}
#lab{height:58vh;min-height:330px;position:relative}
canvas{display:block}
#panel{padding:12px;background:#17283a}
button{padding:10px;margin:4px;border:0;border-radius:8px;
background:#159bba;color:white;font-weight:bold}
.item{background:#263e54}
#status{padding:10px;background:#0c1825;border-radius:8px}
small{color:#b8d9e8}
</style>
</head>
<body>
<header>
<h2>🧪 Kamal Sir Chemistry</h2>
<p>CLASS 9–10 • INTERACTIVE VIRTUAL LAB</p>
</header>

<div id="lab"></div>

<div id="panel">
<h3>🔬 प्रयोग की मेज</h3>
<p>पहले कोई पदार्थ चुनो, फिर बीकर पर टैप करो।</p>
<div>
<button class="item" data-item="water">💧 पानी</button>
<button class="item" data-item="salt">🧂 नमक</button>
<button class="item" data-item="sugar">🍚 चीनी</button>
<button class="item" data-item="sand">🏖️ रेत</button>
</div>
<div>
<button id="reset">नया प्रयोग</button>
<button id="dissolve">मिश्रण देखें</button>
</div>
<div id="status" aria-live="polite">
लैब तैयार है। 3D दृश्य में किसी वस्तु को चुनकर देखें।
</div>
<p><small>वर्चुअल मॉडल — वास्तविक प्रयोगों के लिए शिक्षक के निर्देश मानें।</small></p>
</div>

<script type="module">
import * as THREE from
'https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js';

const root = document.getElementById('lab');
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x9ccbdc);

const camera = new THREE.PerspectiveCamera(
55, root.clientWidth/root.clientHeight, 0.1, 100
);
camera.position.set(0,5,10);
camera.lookAt(0,1,0);

const renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setSize(root.clientWidth,root.clientHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio,2));
renderer.shadowMap.enabled=true;
root.appendChild(renderer.domElement);

scene.add(new THREE.HemisphereLight(0xffffff,0x536477,2));
const light = new THREE.DirectionalLight(0xffffff,2.5);
light.position.set(-4,8,5);
light.castShadow=true;
scene.add(light);

function box(w,h,d,color,x,y,z){
 const mesh=new THREE.Mesh(
  new THREE.BoxGeometry(w,h,d),
  new THREE.MeshStandardMaterial({color,roughness:.8})
 );
 mesh.position.set(x,y,z);
 mesh.castShadow=true;
 mesh.receiveShadow=true;
 scene.add(mesh);
 return mesh;
}

// कमरे का फर्श और दीवारें
box(14,.2,12,0xd7dce2,0,-.12,0);
box(14,5,.15,0xeaf2f5,0,2.4,-5.8);
box(.15,5,12,0xc4dbe5,-7,2.4,0);
box(.15,5,12,0xc4dbe5,7,2.4,0);

// खिड़कियाँ
box(2,1.5,.08,0x5cc9ee,-3,3.2,-5.68);
box(2,1.5,.08,0x5cc9ee,3,3.2,-5.68);

// लैब की मेज
box(8,.25,2,0x805536,0,1,0);
for(const x of [-3.5,3.5])
 for(const z of [-.7,.7])
  box(.18,1,.18,0x465362,x,.4,z);

// स्टोरेज कैबिनेट
box(2,2,.7,0x47758a,-4,1, -3.8);

// वस्तुओं के लिए लेबल
function label(text,x,y,z){
 const c=document.createElement('canvas');
 c.width=512;c.height=128;
 const ctx=c.getContext('2d');
 ctx.fillStyle='#102d42';
 ctx.fillRect(0,0,512,128);
 ctx.fillStyle='white';
 ctx.font='bold 42px Arial';
 ctx.textAlign='center';
 ctx.fillText(text,256,78);
 const tex=new THREE.CanvasTexture(c);
 const s=new THREE.Sprite(
  new THREE.SpriteMaterial({map:tex,transparent:true})
 );
 s.position.set(x,y,z);s.scale.set(1.8,.45,1);
 scene.add(s);
 return s;
}

const objects=[];
function vessel(name,x,z,color){
 const group=new THREE.Group();
 group.position.set(x,1.15,z);
 scene.add(group);

 const glass=new THREE.MeshStandardMaterial({
  color:0xb9edf4,transparent:true,opacity:.45,
  roughness:.15,side:THREE.DoubleSide
 });
 const body=new THREE.Mesh(
  new THREE.CylinderGeometry(.34,.29,.8,32,1,true),glass
 );
 group.add(body);

 const rim=new THREE.Mesh(
  new THREE.TorusGeometry(.34,.035,8,32),
  new THREE.MeshStandardMaterial({color:0xe6ffff})
 );
 rim.rotation.x=Math.PI/2;
 rim.position.y=.4;
 group.add(rim);

 const liquid=new THREE.Mesh(
  new THREE.CylinderGeometry(.27,.25,.08,32),
  new THREE.MeshStandardMaterial({
   color,transparent:true,opacity:.8
  })
 );
 liquid.position.y=-.32;
 group.add(liquid);

 group.userData={name,liquid,contents:[]};
 objects.push(group);
 label(name,x,2.05,z);
 return group;
}

const beaker=vessel('BEAKER',0,0,0x42bde8);
vessel('TEST TUBE',-2,0,0x76d8a5);
vessel('FLASK',2,0,0xe4b6ff);

// नाम लिखे पदार्थ के डिब्बे
const materials={
 water:{name:'पानी',color:0x42bde8},
 salt:{name:'नमक',color:0xffffff},
 sugar:{name:'चीनी',color:0xf3d68b},
 sand:{name:'रेत',color:0xd5a45c}
};
const containers=[];
const positions=[[-2,-1.8],[0,-1.8],[2,-1.8],[4,-1.8]];

Object.entries(materials).forEach(([key,item],i)=>{
 const [x,z]=positions[i];
 const jar=new THREE.Mesh(
  new THREE.CylinderGeometry(.28,.28,.65,24),
  new THREE.MeshStandardMaterial({
   color:item.color,transparent:true,opacity:.88
  })
 );
 jar.position.set(x,1.48,z);
 jar.userData={type:'material',key};
 scene.add(jar);
 containers.push(jar);
 label(item.name,x,2.02,z);
});

let selected=null;
const status=document.getElementById('status');
document.querySelectorAll('[data-item]').forEach(btn=>{
 btn.onclick=()=>{
  selected=btn.dataset.item;
  status.textContent='चुना गया: '+materials[selected].name+
  '। अब 3D दृश्य में BEAKER पर टैप करो।';
 };
});

const raycaster=new THREE.Raycaster();
const pointer=new THREE.Vector2();

renderer.domElement.addEventListener('pointerdown',event=>{
 const rect=renderer.domElement.getBoundingClientRect();
 pointer.x=((event.clientX-rect.left)/rect.width)*2-1;
 pointer.y=-((event.clientY-rect.top)/rect.height)*2+1;
 raycaster.setFromCamera(pointer,camera);
 const hits=raycaster.intersectObjects(scene.children,true);
 if(!hits.length)return;

 let obj=hits[0].object;
 while(obj.parent && !obj.userData.type &&
       !obj.userData.name)obj=obj.parent;

 if(obj.userData.type==='material'){
  selected=obj.userData.key;
  status.textContent='चुना गया: '+materials[selected].name+
   '। अब BEAKER पर टैप करो।';
  return;
 }

 if(obj.userData.name==='BEAKER' && selected){
  const item=materials[selected];
  beaker.userData.contents.push(selected);
  beaker.userData.liquid.material.color.set(item.color);
  beaker.userData.liquid.scale.y=Math.min(
   1+beaker.userData.contents.length*.25,3
  );
  status.textContent=item.name+' बीकर में जोड़ा गया। कुल पदार्थ: '+
   beaker.userData.contents.map(k=>materials[k].name).join(', ');
  selected=null;
 }
});

document.getElementById('dissolve').onclick=()=>{
 const c=beaker.userData.contents;
 if(c.includes('water') && c.includes('salt')){
  status.textContent='परिणाम: मॉडल में नमक-पानी का मिश्रण दिखाया गया है। नमक पानी में घुल सकता है।';
 }else if(c.includes('water') && c.includes('sugar')){
  status.textContent='परिणाम: चीनी पानी में घुल सकती है।';
 }else if(c.length===0){
  status.textContent='पहले बीकर में पदार्थ डालो।';
 }else{
  status.textContent='मिश्रण का परिणाम पदार्थों पर निर्भर करता है।';
 }
};

document.getElementById('reset').onclick=()=>{
 beaker.userData.contents=[];
 beaker.userData.liquid.scale.y=1;
 beaker.userData.liquid.material.color.set(0x42bde8);
 status.textContent='बीकर साफ है। नया प्रयोग शुरू करो।';
 selected=null;
};

function animate(){
 requestAnimationFrame(animate);
 renderer.render(scene,camera);
}
animate();

window.addEventListener('resize',()=>{
 camera.aspect=root.clientWidth/root.clientHeight;
 camera.updateProjectionMatrix();
 renderer.setSize(root.clientWidth,root.clientHeight);
});
</script>
</body>
</html>
