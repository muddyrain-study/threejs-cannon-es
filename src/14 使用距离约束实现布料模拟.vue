<template>
  <div></div>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import * as CANNON from "cannon-es";
import Stats from 'three/addons/libs/stats.module.js';
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
// 创建物理世界
const world = new CANNON.World();
// 设置物理世界允许休眠
world.allowSleeping = true;
// 设置物理世界的重力
world.gravity.set(0, -9.8, 0)
// 初始化3D场景
const scene = new THREE.Scene();
// 初始化相机 
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.set(0, 10, 20);

// 初始化渲染器
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// 初始化控制器
const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;


// 设置碰撞组 数值 要用 2的幂
const GROUP1 = 1
const GROUP2 = 2
const GROUP3 = 4
const GROUP4 = 8
let phyMeshes = []
let meshes = []

const boxMaterialCon = new CANNON.Material("boxMaterial")
boxMaterialCon.friction = 0
boxMaterialCon.restitution = 0.1


const rows = 15
const cols = 15
const bodies = {
}

const particleShape = new CANNON.Particle()

for (let i = 0; i < rows; i++) {
  for (let j = 0; j < cols; j++) {
    // const boxShape = new CANNON.Box(new CANNON.Vec3(0.5, 0.5, 0.5))
    const boxBody = new CANNON.Body({
      mass: 0.5,
      shape: particleShape,
      // material: boxMaterialCon,
      position: new CANNON.Vec3(
        i - cols * 0.5, 10, j - rows * 0.5
      ),
    })
    world.addBody(boxBody)
    phyMeshes.push(boxBody)
    bodies[`${[i]}-${[j]}`] = boxBody

    const boxMesh = new THREE.Mesh(
      new THREE.SphereGeometry(0.3, 8, 8),
      new THREE.MeshBasicMaterial({ color: 0xff0000 })
    )
    boxMesh.position.set(i - cols * 0.5, 10, j - rows * 0.5)
    meshes.push(boxMesh)
    scene.add(boxMesh)
  }
}


for (let i = 0; i < rows; i++) {
  for (let j = 0; j < cols; j++) {
    const body = bodies[`${[i]}-${[j]}`]
    if (i > 0) {
      const body2 = bodies[`${[i - 1]}-${[j]}`]
      const constraint = new CANNON.DistanceConstraint(body, body2, 1)
      world.addConstraint(constraint)
    }
    if (j > 0) {
      const body2 = bodies[`${[i]}-${[j - 1]}`]
      const constraint = new CANNON.DistanceConstraint(body, body2, 1)
      world.addConstraint(constraint)
    }
  }
}

// 创建物理球
const sphereShape = new CANNON.Sphere(5)
const sphereBody = new CANNON.Body({
  mass: 0,
  shape: sphereShape,
  material: boxMaterialCon,
  position: new CANNON.Vec3(
    0, 0, 0
  ),
})
world.addBody(sphereBody)
phyMeshes.push(sphereBody)

const sphereGeometry = new THREE.SphereGeometry(5, 32, 32)
const sphereMaterial = new THREE.MeshBasicMaterial({ color: 0xffff00 })
const sphereMesh = new THREE.Mesh(sphereGeometry, sphereMaterial)
sphereMesh.position.set(0, 0, 0)
scene.add(sphereMesh)
meshes.push(sphereMesh)

// 渲染
let clock = new THREE.Clock();
// 添加帧率检测器
const stats = new Stats();
document.body.appendChild(stats.dom);

const axesHelper = new THREE.AxesHelper(5)
scene.add(axesHelper)
function animate() {
  let delta = clock.getDelta();
  world.step(1 / 60, delta);

  for (let i = 0; i < phyMeshes.length; i++) {
    meshes[i].position.copy(phyMeshes[i].position)
    meshes[i].quaternion.copy(phyMeshes[i].quaternion)
  }

  stats.update();
  controls.update();
  renderer.render(scene, camera);
  requestAnimationFrame(animate);
}

animate();

</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

canvas {
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
}
</style>
