<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Cultivator's Cove Mini</title>
<style>
  body { margin: 0; overflow: hidden; }
  #ui {
    position: absolute;
    top: 10px; left: 10px;
    color: white; font-size: 20px; font-family: sans-serif;
    background: rgba(0,0,0,0.4); padding: 5px 10px; border-radius: 5px;
  }
</style>
</head>
<body>
<div id="ui">Crops: 0</div>
<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/examples/js/controls/PointerLockControls.js"></script>
<script>
let scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb); // sky blue

let camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);

let renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// Ground
let groundGeo = new THREE.PlaneGeometry(100, 100);
let groundMat = new THREE.MeshPhongMaterial({color: 0x228B22});
let ground = new THREE.Mesh(groundGeo, groundMat);
ground.rotation.x = -Math.PI/2;
ground.receiveShadow = true;
scene.add(ground);

// Light
let light = new THREE.DirectionalLight(0xffffff, 1);
light.position.set(5, 10, 5);
scene.add(light);
scene.add(new THREE.AmbientLight(0xffffff, 0.3));

// Controls
let controls = new THREE.PointerLockControls(camera, document.body);
document.body.addEventListener('click', () => controls.lock());
camera.position.set(0, 1.6, 5);

// Movement
let velocity = new THREE.Vector3();
let keys = {};
document.addEventListener('keydown', e => keys[e.code] = true);
document.addEventListener('keyup', e => keys[e.code] = false);

// Crops
let crops = [];
let cropGeo = new THREE.ConeGeometry(0.3, 1, 8);
let cropMat = new THREE.MeshPhongMaterial({color: 0xffd700});
for (let i=0; i<10; i++) {
  let crop = new THREE.Mesh(cropGeo, cropMat);
  crop.position.set(Math.random()*50-25, 0.5, Math.random()*50-25);
  scene.add(crop);
  crops.push(crop);
}

let cropCount = 0;
let ui = document.getElementById('ui');

// Harvest on click
document.addEventListener('mousedown', () => {
  let raycaster = new THREE.Raycaster();
  raycaster.setFromCamera(new THREE.Vector2(0,0), camera);
  let intersects = raycaster.intersectObjects(crops);
  if (intersects.length > 0) {
    let crop = intersects[0].object;
    scene.remove(crop);
    crops = crops.filter(c => c !== crop);
    cropCount++;
    ui.textContent = `Crops: ${cropCount}`;
  }
});

// Animation loop
let clock = new THREE.Clock();
function animate() {
  requestAnimationFrame(animate);
  let delta = clock.getDelta();

  if (controls.isLocked) {
    velocity.set(0,0,0);
    let speed = 5;
    if (keys['KeyW'] || keys['ArrowUp']) velocity.z -= speed * delta;
    if (keys['KeyS'] || keys['ArrowDown']) velocity.z += speed * delta;
    if (keys['KeyA'] || keys['ArrowLeft']) velocity.x -= speed * delta;
    if (keys['KeyD'] || keys['ArrowRight']) velocity.x += speed * delta;
    controls.moveRight(velocity.x);
    controls.moveForward(velocity.z);
  }

  renderer.render(scene, camera);
}
animate();

window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth/window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});
</script>
</body>
</html>
