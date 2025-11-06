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

// DVD bouncing variables
let velocityX = 0.02;
let velocityY = 0.015;
let cubeSize = 1; // Size of the cube for collision detection
const BOUNDARY_OFFSET = 0.001; // Small offset to prevent cube from sticking at boundaries

// Cached boundary values (recalculated on resize)
let maxX = 0;
let maxY = 0;
let time = 0;

// DVD bouncing variables
let velocityX = 0.02;
let velocityY = 0.015;
let cubeSize = 1; // Size of the cube for collision detection
const BOUNDARY_OFFSET = 0.001; // Small offset to prevent cube from sticking at boundaries

// Cached boundary values (recalculated on resize)
let maxX = 0;
let maxY = 0;

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

  // Calculate initial boundaries
  updateBoundaries();
}

function updateBoundaries() {
  const aspect = window.innerWidth / window.innerHeight;
  const vFOV = THREE.MathUtils.degToRad(camera.fov);
  // Use cube's actual z-position for accurate boundary calculation
  const distance = Math.abs(camera.position.z - cube.position.z);
  const height = 2 * Math.tan(vFOV / 2) * distance;
  const width = height * aspect;

  maxX = width / 2 - cubeSize / 2;
  maxY = height / 2 - cubeSize / 2;

  // Calculate initial boundaries
  updateBoundaries();
}

function updateBoundaries() {
  const aspect = window.innerWidth / window.innerHeight;
  const vFOV = THREE.MathUtils.degToRad(camera.fov);
  // Use cube's actual z-position for accurate boundary calculation
  const distance = Math.abs(camera.position.z - cube.position.z);
  const height = 2 * Math.tan(vFOV / 2) * distance;
  const width = height * aspect;

  maxX = width / 2 - cubeSize / 2;
  maxY = height / 2 - cubeSize / 2;
}

function animate() {
  requestAnimationFrame(animate);

  // Keep rotation for visual effect

  // Increment time for color animation
  time += 0.01;

  // Create pulsing colors using sine waves
  const r = Math.sin(time * 0.5) * 0.5 + 0.5; // Red channel
  const g = Math.sin(time * 0.3 + 2) * 0.5 + 0.5; // Green channel
  const b = Math.sin(time * 0.7 + 4) * 0.5 + 0.5; // Blue channel

  scene.background = new THREE.Color(r, g, b);

  // Keep rotation for visual effect
  cube.rotation.x += 0.01;
  cube.rotation.y += 0.01;

  // Update position for DVD bouncing effect
  cube.position.x += velocityX;
  cube.position.y += velocityY;

  // Calculate visible boundaries based on camera frustum
  const aspect = window.innerWidth / window.innerHeight;
  const vFOV = THREE.MathUtils.degToRad(camera.fov);
  const height =
    2 * Math.tan(vFOV / 2) * Math.abs(camera.position.z - cube.position.z);
  const width = height * aspect;

  const maxX = width / 2 - cubeSize / 2;
  const maxY = height / 2 - cubeSize / 2;

  // Check boundaries and reverse direction if needed
  if (cube.position.x >= maxX) {
    velocityX = -Math.abs(velocityX); // Ensure negative velocity
    cube.position.x = maxX - 0.001; // Position just inside boundary
  } else if (cube.position.x <= -maxX) {
    velocityX = Math.abs(velocityX); // Ensure positive velocity
    cube.position.x = -maxX + 0.001; // Position just inside boundary
  }

  if (cube.position.y >= maxY) {
    velocityY = -Math.abs(velocityY); // Ensure negative velocity
    cube.position.y = maxY - 0.001; // Position just inside boundary
  } else if (cube.position.y <= -maxY) {
    velocityY = Math.abs(velocityY); // Ensure positive velocity
    cube.position.y = -maxY + 0.001; // Position just inside boundary
  }

  // Update position for DVD bouncing effect
  cube.position.x += velocityX;
  cube.position.y += velocityY;

  // Check boundaries and reverse direction if needed
  if (cube.position.x >= maxX) {
    velocityX = -Math.abs(velocityX); // Ensure negative velocity
    cube.position.x = maxX - BOUNDARY_OFFSET; // Position just inside boundary
  } else if (cube.position.x <= -maxX) {
    velocityX = Math.abs(velocityX); // Ensure positive velocity
    cube.position.x = -maxX + BOUNDARY_OFFSET; // Position just inside boundary
  }

  if (cube.position.y >= maxY) {
    velocityY = -Math.abs(velocityY); // Ensure negative velocity
    cube.position.y = maxY - BOUNDARY_OFFSET; // Position just inside boundary
  } else if (cube.position.y <= -maxY) {
    velocityY = Math.abs(velocityY); // Ensure positive velocity
    cube.position.y = -maxY + BOUNDARY_OFFSET; // Position just inside boundary
  }

  renderer.render(scene, camera);
}

onMounted(() => {
  initThree();
  animate();

  // Add window resize event listener
  window.addEventListener("resize", onWindowResize);
});

function onWindowResize() {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
  updateBoundaries(); // Recalculate boundaries on resize
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
