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

// 创建15个距离约束的小球
for (let i = 0; i < 15; i++) {
  const sphereShape = new CANNON.Sphere(0.45);
  const sphereBody = new CANNON.Body({
    mass: i == 0 ? 0 : 1,
    shape: sphereShape,
    position: new CANNON.Vec3(0, 15 - i * 0.9, 0),
  });
  world.addBody(sphereBody);
  phyMeshes.push(sphereBody);

  const sphereGeometry = new THREE.SphereGeometry(0.45, 32, 32);
  const sphereMaterial = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
  const sphereMesh = new THREE.Mesh(sphereGeometry, sphereMaterial);
  sphereMesh.position.copy(sphereBody.position);
  meshes.push(sphereMesh);
  scene.add(sphereMesh);

  // 创建距离约束
  if (i > 0) {
    const constraint = new CANNON.DistanceConstraint(
      sphereBody,
      phyMeshes[i - 1],
      0.9
    );
    world.addConstraint(constraint);
  }
}

window.addEventListener("click", () => {
  // 创建一个球体
  const sphereShape = new CANNON.Sphere(0.45);
  const sphereBody = new CANNON.Body({
    mass: 1,
    shape: sphereShape,
    position: new CANNON.Vec3(5, 10, 0),
  });
  world.addBody(sphereBody);
  phyMeshes.push(sphereBody);
  sphereBody.velocity.set(-Math.random() * 50, 0, 0);

  const sphereGeometry = new THREE.SphereGeometry(0.45, 32, 32);
  const sphereMaterial = new THREE.MeshBasicMaterial({ color: 0xff0000 });
  const sphereMesh = new THREE.Mesh(sphereGeometry, sphereMaterial);
  sphereMesh.position.copy(sphereBody.position);
  meshes.push(sphereMesh);
  scene.add(sphereMesh);
});
// 在每次世界模拟完一个时间步之后，判断约束力度的绝对值大小，如果大于1000，就删除约束
world.addEventListener("postStep", () => {
  // 循环判断世界约束
  for (let i = 0; i < world.constraints.length; i++) {
    const constraint = world.constraints[i];
    // 获取约束力度的绝对值大小
    let multiplier = Math.abs(constraint.equations[0].multiplier);
    if (multiplier > 1000) {
      // 删除约束
      world.removeConstraint(constraint);
    }
  }
})

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
