<script setup lang="ts">
import { onMounted, onUnmounted, ref } from "vue";
import * as THREE from "three";
import kamalaImage from "@/assets/images/kamala.png";

const canvasRef = ref<HTMLCanvasElement | null>(null);

let scene: THREE.Scene;
let camera: THREE.PerspectiveCamera;
let renderer: THREE.WebGLRenderer;
let material: THREE.MeshBasicMaterial;
let geometry: THREE.BoxGeometry;
let cube: THREE.Mesh;
let texture: THREE.Texture;
let time = 0;

function initThree() {
  scene = new THREE.Scene();
  scene.background = new THREE.Color(0x800080); // Set background color (dark gray)

  camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  );

  renderer = new THREE.WebGLRenderer({
    canvas: canvasRef.value as HTMLCanvasElement,
  });
  renderer.setSize(window.innerWidth, window.innerHeight);

  const textureLoader = new THREE.TextureLoader();
  texture = textureLoader.load(kamalaImage);

  // Make texture sharper
  texture.minFilter = THREE.NearestFilter;
  texture.magFilter = THREE.NearestFilter;
  texture.anisotropy = renderer.capabilities.getMaxAnisotropy();

  geometry = new THREE.BoxGeometry();
  material = new THREE.MeshBasicMaterial({ map: texture });
  cube = new THREE.Mesh(geometry, material);
  cube.scale.set(0.5, 0.5, 0.5); // Make cube smaller (50% of original size)
  scene.add(cube);

  camera.position.z = 1;
}

function animate() {
  requestAnimationFrame(animate);

  // Increment time for color animation
  time += 0.01;

  // Create pulsing colors using sine waves
  const r = Math.sin(time * 0.5) * 0.5 + 0.5; // Red channel
  const g = Math.sin(time * 0.3 + 2) * 0.5 + 0.5; // Green channel
  const b = Math.sin(time * 0.7 + 4) * 0.5 + 0.5; // Blue channel

  scene.background = new THREE.Color(r, g, b);

  cube.rotation.x += 0.01;
  cube.rotation.y += 0.01;
  renderer.render(scene, camera);
}

onMounted(() => {
  initThree();
  animate();

  // Add window resize event listener
  window.addEventListener("resize", onWindowResize);
});

function onWindowResize() {
  renderer.setSize(window.innerWidth, window.innerHeight);
}

onUnmounted(() => {
  // Clean up Three.js resources
  scene.remove(cube);
  geometry.dispose();
  material.dispose();
  renderer.dispose();

  // Remove window resize event listener
  window.removeEventListener("resize", onWindowResize);
});
</script>

<template>
  <canvas ref="canvasRef"></canvas>
</template>

<style scoped>
canvas {
  display: block;
  width: 100%;
  height: 100vh;
}
</style>
