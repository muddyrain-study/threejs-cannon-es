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

// 创建1个固定的静态球体
const sphereShape = new CANNON.Sphere(0.2);
const sphereBody = new CANNON.Body({
  mass: 0,
  shape: sphereShape,
  position: new CANNON.Vec3(0, 10, 0),
  type: CANNON.Body.STATIC,
});
world.addBody(sphereBody);
phyMeshes.push(sphereBody);

const sphereGeometry = new THREE.SphereGeometry(0.2, 32, 32);
const sphereMaterial = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
const sphereMesh = new THREE.Mesh(sphereGeometry, sphereMaterial);
sphereMesh.position.copy(sphereBody.position);
meshes.push(sphereMesh);
scene.add(sphereMesh);

// 创建1个立方体
const boxShape = new CANNON.Box(new CANNON.Vec3(1, 1, 0.3));
const boxBody = new CANNON.Body({
  mass: 1,
  shape: boxShape,
  position: new CANNON.Vec3(0, 6, 0),
});
world.addBody(boxBody);
phyMeshes.push(boxBody);

const boxGeometry = new THREE.BoxGeometry(2, 2, 0.6);
const boxMaterial = new THREE.MeshBasicMaterial({ color: 0x0000ff });
const boxMesh = new THREE.Mesh(boxGeometry, boxMaterial);
boxMesh.position.copy(boxBody.position);
meshes.push(boxMesh);
scene.add(boxMesh);

// 创建1个弹簧拉住立方体
const spring = new CANNON.Spring(boxBody, sphereBody, {
  // 弹簧的长度
  restLength: 2,
  // 弹簧的刚度
  stiffness: 100,
  // 弹簧的阻尼
  damping: 1,
  // 弹簧的锚点
  localAnchorA: new CANNON.Vec3(0, 0, 0),
  localAnchorB: new CANNON.Vec3(-1, 1, 0),
});
// 通过计算每一step之前获取弹簧的作用力，并且应用弹簧的作用力
world.addEventListener("preStep", () => {
  // 应用弹簧的作用力
  spring.applyForce();
});

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
