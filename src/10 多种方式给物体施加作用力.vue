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
camera.position.set(0, 5, 10);

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

const boxMaterialCon = new CANNON.Material("boxMaterial")
boxMaterialCon.friction = 0
boxMaterialCon.restitution = 0.1

// 创建一个物理世界的平面
const planeShape = new CANNON.Box(new CANNON.Vec3(5, 0.1, 5))
// 创建一个刚体
const planeBody = new CANNON.Body({
  // mass: 0,
  shape: planeShape,
  material: boxMaterialCon,
  position: new CANNON.Vec3(0, -0.1, 0),
  type: CANNON.Body.STATIC,
  collisionFilterGroup: GROUP1,
  collisionFilterMask: GROUP1 | GROUP2 | GROUP3
})
// planeBody.quaternion.setFromAxisAngle(new CANNON.Vec3(1, 0, 0), 0.1)
// 将刚体添加到物理世界
world.addBody(planeBody)


// 创建一个平面几何体
// const planeGeometry = new THREE.PlaneGeometry(10, 10);
const planeGeometry = new THREE.BoxGeometry(10, 0.2, 10,);
// 创建一个平面材质
const planeMaterial = new THREE.MeshBasicMaterial({ color: 0xffff00 });
// 创建一个平面网格
const planeMesh = new THREE.Mesh(planeGeometry, planeMaterial);
// x轴旋转90度
// planeMesh.rotation.x = 0.1;
planeMesh.position.y = -0.1
// 将网格添加到3D场景
scene.add(planeMesh);

// 物理世界物体
let phyMeshes = []
// 渲染世界物体
let meshs = []

// 创建物理球
const sphereShape = new CANNON.Sphere(0.5)
// 创建物理刚体
const sphereBody = new CANNON.Body({
  mass: 1,
  position: new CANNON.Vec3(0, 5, 0),
  shape: sphereShape,
  material: boxMaterialCon,
  collisionFilterGroup: GROUP2,
  collisionFilterMask: GROUP1 | GROUP2 | GROUP3
})
sphereBody.quaternion.setFromAxisAngle(new CANNON.Vec3(0, 0, 1), Math.PI)
// 将刚体添加到物理世界
world.addBody(sphereBody)
phyMeshes.push(sphereBody)

// 创建 3d球
const sphereGeometry = new THREE.SphereGeometry(0.5, 8, 8);
const sphereMaterial = new THREE.MeshBasicMaterial({ color: 0xff0000, wireframe: true });
const sphereMesh = new THREE.Mesh(sphereGeometry, sphereMaterial);
sphereMesh.position.y = 5
scene.add(sphereMesh);
meshs.push(sphereMesh)

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
    meshs[i].position.copy(phyMeshes[i].position)
    meshs[i].quaternion.copy(phyMeshes[i].quaternion)
  }

  stats.update();
  controls.update();
  renderer.render(scene, camera);
  requestAnimationFrame(animate);
}

animate();

// 监听点击事件
window.addEventListener("click", () => {
  console.log(sphereBody.position);
  // 添加 压力
  // sphereBody.applyForce(new CANNON.Vec3(10, 0, 0), new CANNON.Vec3(0, -0.5, 0))
  // sphereBody.applyLocalForce(new CANNON.Vec3(10, 0, 0), new CANNON.Vec3(0, -0.5, 0))
  // 添加 脉冲
  // sphereBody.applyImpulse(
  //   new CANNON.Vec3(10 * (1 / 60), 0, 0),
  //   new CANNON.Vec3(0, -0.5, 0)
  // )
  // sphereBody.applyLocalImpulse(
  //   new CANNON.Vec3(10 * (1 / 60), 0, 0),
  //   new CANNON.Vec3(0, -0.5, 0)
  // )
  // 扭矩
  sphereBody.applyTorque(new CANNON.Vec3(0, 0, 10))
})
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
