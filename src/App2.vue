<template>

</template>
<script setup>
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.1, 1000 );
camera.position.z = 5;
scene.background = new THREE.Color(0x000000);
const renderer = new THREE.WebGLRenderer();
renderer.setSize( window.innerWidth, window.innerHeight );

// 加一個光源，讓模型能被看到
const light = new THREE.DirectionalLight(0xffffff, 1);
light.position.set(5, 5, 5);
scene.add(light);

// 可加環境光讓陰影不太硬
scene.add(new THREE.AmbientLight(0xffffff, 0.5));

// const loader = new GLTFLoader();
// loader.load('/0405.glb', (gltf) => {
//   const model = gltf.scene;

//   // 遍歷模型的每個 Mesh，強制設定白色材質（可選）
//   model.traverse((child) => {
//     if (child.isMesh) {
//       child.material = new THREE.MeshStandardMaterial({ color: 0xffffff });
//     }
//   });

//   scene.add(model);
// });
// 很像setInterval的函式。每一幀都會執行這個函式
function animate() {
  // 每一幀物體都會自轉
  cube.rotation.x += 0.01;
  cube.rotation.y += 0.01;
  // 它每一幀執行animate()
  requestAnimationFrame(animate);
  // 每一幀，場景物件都會被鏡頭捕捉
  renderer.render(scene, camera);
}
// 函式起始點
animate();
</script>
