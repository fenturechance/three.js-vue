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
camera.position.set(0, 0, 15);

// 實例化渲染器
const renderer = new THREE.WebGLRenderer();
// 渲染器負責投影畫面在螢幕上，會需要寬高
renderer.setSize(window.innerWidth, window.innerHeight);
new OrbitControls( camera, renderer.domElement );
// 渲染器會產生canvas物件，我們在html的body放置它
document.body.appendChild(renderer.domElement);


// 加一個光源，讓模型能被看到
const light = new THREE.DirectionalLight(0xffffff, 1);
light.position.set(0, 5, 0);
scene.add(light);

// 加上幫助器：長度為 1，可調整
const lightHelper = new THREE.DirectionalLightHelper(light, 1);
scene.add(lightHelper);

// 可加環境光讓陰影不太硬
scene.add(new THREE.AmbientLight(0xffffff, 0.5));


const loader = new GLTFLoader();
loader.load('/0405.glb', (gltf) => {
  const model = gltf.scene;
  scene.add(model);
});
function animate() {
  requestAnimationFrame(animate);
  renderer.render(scene, camera);
}
// 函式起始點
animate();
</script>
