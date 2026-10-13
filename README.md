# warship-game
 My 3D Warship Game

<!DOCTYPE html>
<html lang="th">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Warship 3D</title>
<style>
body{margin:0;overflow:hidden;background:#061b2b;color:white;font-family:Arial}
#ui{position:fixed;top:12px;left:12px;z-index:2;background:#001a2acc;padding:14px;border-radius:12px}
button{padding:12px 18px;background:#167cb5;color:white;border:0;border-radius:8px;font-size:16px}
#help{font-size:13px;line-height:1.7}
</style>
</head>
<body>
<div id="ui">
  <b>⚓ WARSHIP 3D</b>
  <div id="score">คะแนน: 0</div>
  <div id="help">WASD = เคลื่อนที่<br>คลิกหรือแตะ = ยิงปืน</div>
  <button id="fire">FIRE!</button>
</div>
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x70b9d6);
scene.fog = new THREE.Fog(0x70b9d6,80,250);

const camera = new THREE.PerspectiveCamera(
  65,innerWidth/innerHeight,0.1,500
);
camera.position.set(0,16,25);

const renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth,innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio,2));
document.body.appendChild(renderer.domElement);

scene.add(new THREE.HemisphereLight(0xffffff,0x35566b,2));
const sun = new THREE.DirectionalLight(0xffffff,2);
sun.position.set(10,30,10);
scene.add(sun);

const water = new THREE.Mesh(
  new THREE.PlaneGeometry(500,500),
  new THREE.MeshStandardMaterial({color:0x087eae,roughness:0.4})
);
water.rotation.x=-Math.PI/2;
water.position.y=-0.7;
scene.add(water);

function makeShip(color){
  const ship=new THREE.Group();
  const hull=new THREE.Mesh(
    new THREE.BoxGeometry(2.4,0.8,6),
    new THREE.MeshStandardMaterial({color})
  );
  hull.position.y=0;
  ship.add(hull);

  const deck=new THREE.Mesh(
    new THREE.BoxGeometry(1.5,0.7,2.8),
    new THREE.MeshStandardMaterial({color:0xc6c9c9})
  );
  deck.position.set(0,0.65,0.3);
  ship.add(deck);

  const turret=new THREE.Mesh(
    new THREE.CylinderGeometry(0.55,0.55,0.35,16),
    new THREE.MeshStandardMaterial({color:0x343c42})
  );
  turret.position.set(0,1.1,-0.8);
  ship.add(turret);

  const barrel=new THREE.Mesh(
    new THREE.BoxGeometry(0.18,0.18,1.6),
    new THREE.MeshStandardMaterial({color:0x20272b})
  );
  barrel.position.set(0,1.2,-1.6);
  ship.add(barrel);
  return ship;
}

const player=makeShip(0x28536b);
scene.add(player);
player.position.set(0,0,8);

const enemies=[];
for(let i=0;i<7;i++){
  const e=makeShip(0x9c3d36);
  e.position.set((Math.random()-0.5)*65,0,-20-Math.random()*100);
  e.rotation.y=Math.PI;
  scene.add(e);
  enemies.push(e);
}

const keys={};
let score=0;
let lastShot=0;

addEventListener('keydown',e=>keys[e.key.toLowerCase()]=true);
addEventListener('keyup',e=>keys[e.key.toLowerCase()]=false);

function shoot(){
  if(performance.now()-lastShot<350)return;
  lastShot=performance.now();

  const bullet=new THREE.Mesh(
    new THREE.SphereGeometry(0.22,8,8),
    new THREE.MeshBasicMaterial({color:0xffef91})
  );
  bullet.position.copy(player.position);
  bullet.position.y=1;
  scene.add(bullet);

  const direction=new THREE.Vector3(0,0,-1);
  let alive=true;

  function moveBullet(){
    if(!alive)return;
    bullet.position.addScaledVector(direction,1.8);

    for(let i=enemies.length-1;i>=0;i--){
      if(bullet.position.distanceTo(enemies[i].position)<4){
        scene.remove(enemies[i]);
        enemies.splice(i,1);
        scene.remove(bullet);
        alive=false;
        score+=100;
        document.getElementById('score').textContent='คะแนน: '+score;
        if(enemies.length===0)spawnEnemies();
        return;
      }
    }

    if(bullet.position.z < -160){
      scene.remove(bullet);
      alive=false;
      return;
    }
    requestAnimationFrame(moveBullet);
  }
  moveBullet();
}

function spawnEnemies(){
  for(let i=0;i<7;i++){
    const e=makeShip(0x9c3d36);
    e.position.set((Math.random()-0.5)*65,0,-30-Math.random()*100);
    e.rotation.y=Math.PI;
    scene.add(e);
    enemies.push(e);
  }
  alert('กำจัดเรือศัตรูครบแล้ว! เริ่มรอบใหม่');
}

document.getElementById('fire').addEventListener('click',shoot);
renderer.domElement.addEventListener('pointerdown',shoot);

function animate(){
  requestAnimationFrame(animate);

  if(keys['w']||keys['arrowup'])player.position.z-=0.35;
  if(keys['s']||keys['arrowdown'])player.position.z+=0.35;
  if(keys['a']||keys['arrowleft'])player.position.x-=0.35;
  if(keys['d']||keys['arrowright'])player.position.x+=0.35;

  player.position.x=Math.max(-35,Math.min(35,player.position.x));
  player.position.z=Math.max(-40,Math.min(35,player.position.z));

  camera.position.x+=(player.position.x-camera.position.x)*0.04;
  camera.position.z+=(player.position.z+20-camera.position.z)*0.04;
  camera.lookAt(player.position.x,0,player.position.z-12);

  water.position.z=player.position.z;
  renderer.render(scene,camera);
}
addEventListener('resize',()=>{
  camera.aspect=innerWidth/innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(innerWidth,innerHeight);
});
animate();
</script>
</body>
</html>
