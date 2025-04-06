<template>

</template>
<script setup>
// Three.js Energy Flow Monitoring Demo (Box-based)
import Decimal from 'decimal.js';
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';

let scene, camera, renderer, raycaster, mouse;
let generator, battery, consumer, batteryLevel, flowLine
let level = 0.5;
let gapSize = new Decimal(0.2); // 初始值

init();
animate();

function init() {
  scene = new THREE.Scene();
  // scene.background = new THREE.Color('#f0f0f0');

  camera = new THREE.PerspectiveCamera(70, window.innerWidth / window.innerHeight, 0.1, 1000);
  camera.position.set(10, 10, 20);
  camera.lookAt(0, 0, 0);

  renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.body.appendChild(renderer.domElement);

  new OrbitControls(camera, renderer.domElement);

  raycaster = new THREE.Raycaster();
  mouse = new THREE.Vector2();
  window.addEventListener('click', onMouseClick);

  scene.add(new THREE.AxesHelper(10));
  const light = new THREE.DirectionalLight(0xffffff, 1);
  light.position.set(10, 20, 10);
  scene.add(light);

  // Generator Box (yellow)
  generator = createBox('#ffcc00', 'Generator');
  generator.position.set(-6, 0.5, 0);
  scene.add(generator);

  // Battery Box (blue)
  // battery = createBox('#3399ff', 'Battery');
  // battery.position.set(0, 0.5, 0);
  // battery.userData.levelBar = createBatteryLevelBar();
  // battery.add(battery.userData.levelBar);
  // scene.add(battery);
  const batteryShell = new THREE.Mesh(
    new THREE.CylinderGeometry(1, 1, 5, 32),
    new THREE.MeshPhongMaterial({ color: '#cccccc', transparent: true, opacity: 0.4 })
  );
  scene.add(batteryShell);
  batteryLevel = new THREE.Mesh(
    new THREE.CylinderGeometry(0.9, 0.9, 4 * level, 32),
    new THREE.MeshPhongMaterial({ color: '#00ff00' })
  );
  batteryLevel.position.y = -(2 - 2 * level); // 液面高度調整
  scene.add(batteryLevel);


  // Consumer Box (white)
  consumer = createBox('#eeeeee', 'Consumer');
  consumer.position.set(6, 0.5, 0);
  scene.add(consumer);

  // Energy Flow Line
  // const flowMaterial = new THREE.LineBasicMaterial({ color: 0x00ff00 });
  // const flowPoints = [
  //   new THREE.Vector3(-5, 1, 0),
  //   new THREE.Vector3(0, 1, 0),
  //   new THREE.Vector3(5, 1, 0)
  // ];
  // const flowGeometry = new THREE.BufferGeometry().setFromPoints(flowPoints);
  // const flowLine = new THREE.Line(flowGeometry, flowMaterial);
  // scene.add(flowLine);
  const flowMat = new THREE.LineDashedMaterial({
    color: 0x00ff00,
    dashSize: 0.3,
    gapSize: 0.2,
  });
  const flowGeo = new THREE.BufferGeometry().setFromPoints([
    new THREE.Vector3(-5, 1, 0),
    new THREE.Vector3(0, 1, 0),
    new THREE.Vector3(5, 1, 0),
  ]);
  // 手動設置 line distances
  flowLine = new THREE.Line(flowGeo, flowMat);
  flowLine.computeLineDistances(); // 需要這行才能生效
  scene.add(flowLine);
}

function createBox(color, name) {
  const geometry = new THREE.BoxGeometry(2, 1, 2);
  const material = new THREE.MeshLambertMaterial({ color });
  const mesh = new THREE.Mesh(geometry, material);
  mesh.name = name;
  return mesh;
}

function createBatteryLevelBar() {
  const levelHeight = 0.8;
  const geometry = new THREE.BoxGeometry(0.3, levelHeight, 0.3);
  const material = new THREE.MeshBasicMaterial({ color: '#00ff00' });
  const bar = new THREE.Mesh(geometry, material);
  bar.position.set(0, levelHeight / 2, 1);
  return bar;
}

function onMouseClick(event) {
  const rect = renderer.domElement.getBoundingClientRect();
  mouse.x = ((event.clientX - rect.left) / rect.width) * 2 - 1;
  mouse.y = -((event.clientY - rect.top) / rect.height) * 2 + 1;

  raycaster.setFromCamera(mouse, camera);
  const intersects = raycaster.intersectObjects([generator, battery, consumer]);

  if (intersects.length > 0) {
    const clicked = intersects[0].object;
    alert(`${clicked.name} clicked!`);
  }
}
function updateBattery(level) {
  batteryLevel.scale.y = level;
  batteryLevel.position.y = -2 + 2 * level;
}
console.log(flowLine.material);
function animate() {
  requestAnimationFrame(animate);

  // Animate battery level (simulate charge/discharge)
  // const bar = battery.userData.levelBar;
  // level += 0.01 * (Math.sin(Date.now() * 0.001));
  if(level < 1.5) {
    level += 0.01;
  } else {
    level = 0;
  }
  
  updateBattery(level);
  
  if (gapSize.greaterThan(new Decimal(0.2))) {
    gapSize = gapSize.minus(new Decimal(0.001)); // 使用減法，精確到小數點
  } else {
    gapSize = new Decimal(0.4); // 重置為 0.2
  }

  // 設定給材質
  flowLine.material.gapSize = gapSize.toNumber(); // 轉換為原生的數字
  flowLine.material.needsUpdate = true;


  // batteryLevel = THREE.MathUtils.clamp(batteryLevel, 0.1, 1);
  // bar.scale.y = batteryLevel;
  // bar.position.y = batteryLevel / 2;

  renderer.render(scene, camera);
}

</script>
