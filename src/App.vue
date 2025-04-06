<template>

</template>
<script setup>
import * as THREE from "three";
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
const scene = new THREE.Scene();
// 加入座標軸輔助器（AxesHelper）
const axesHelper = new THREE.AxesHelper(5); // 5是座標軸長度，可自由調整
scene.add(axesHelper);
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
// Camera 身為鏡頭，有位置屬性，設定在Z軸即可。
camera.position.set(10, 10, 20);
camera.lookAt(0, 5, 0);

// 實例化渲染器
const renderer = new THREE.WebGLRenderer();
// 渲染器負責投影畫面在螢幕上，會需要寬高
renderer.setSize(window.innerWidth, window.innerHeight);
new OrbitControls( camera, renderer.domElement );
// 渲染器會產生canvas物件，我們在html的body放置它
document.body.appendChild(renderer.domElement);


// 加一個光源，讓模型能被看到
const light = new THREE.DirectionalLight(0xffffff, 1);
light.position.set(10, 20, 10);
scene.add(light);

// 加上幫助器：長度為 1，可調整
const lightHelper = new THREE.DirectionalLightHelper(light, 1);
scene.add(lightHelper);

// 可加環境光讓陰影不太硬
scene.add(new THREE.AmbientLight(0xffffff, 0.5));

const addGlb = () => {
  const loader = new GLTFLoader();
  loader.load('/0405.glb', (gltf) => {
    const model = gltf.scene;
    scene.add(model);
  });
}

const floorMeshes = []; // 儲存樓層

// Raycaster & mouse
let raycaster = new THREE.Raycaster();
let mouse = new THREE.Vector2();
const addBuilding = () => {
  // 建立 11 層樓
  const floorWidth = 5;
  const floorHeight = 1.2; // 原本可能是 1，調高一點

  for (let i = 0; i < 11; i++) {
    const geometry = new THREE.BoxGeometry(floorWidth, floorHeight, floorWidth);
    const material = new THREE.MeshLambertMaterial({ color: 0x00aaff }); // 預設藍色
    const floor = new THREE.Mesh(geometry, material);
    floor.position.y = i * floorHeight + floorHeight / 2; // 疊上去
    scene.add(floor);
    floorMeshes.push(floor);
    // 在這裡加邊線
    const edgeGeometry = new THREE.EdgesGeometry(geometry);
    const edgeMaterial = new THREE.LineBasicMaterial({ color: 0x000000 });
    const edgeLines = new THREE.LineSegments(edgeGeometry, edgeMaterial);
    edgeLines.position.copy(floor.position);
    scene.add(edgeLines);
  }

  // 點擊事件
  function onMouseClick(event) {
    // 將滑鼠座標轉換為 -1 到 1 的區間（NDC）
    const rect = renderer.domElement.getBoundingClientRect();
    mouse.x = ((event.clientX - rect.left) / rect.width) * 2 - 1;
    mouse.y = -((event.clientY - rect.top) / rect.height) * 2 + 1;

    raycaster.setFromCamera(mouse, camera);
    const intersects = raycaster.intersectObjects(floorMeshes, true);
    // debugger
    if (intersects.length > 0) {
      const selected = intersects[0].object;
      selected.material.color.set(0xff0000); // 變紅色
    }
  }

  // 滑鼠點擊處理
  window.addEventListener('click', onMouseClick);
}
addBuilding()

function animate() {
  requestAnimationFrame(animate);
  renderer.render(scene, camera);
}
// 函式起始點
animate();
</script>
