<template>
  <div></div>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import * as CANNON from "cannon-es";
import Stats from 'three/addons/libs/stats.module.js';

// 创建物理世界
const world = new CANNON.World();
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
camera.position.set(0, 3, 5);

// 初始化渲染器
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// 初始化控制器
const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;


const boxMaterialCon = new CANNON.Material("boxMaterial")
boxMaterialCon.friction = 0.7
boxMaterialCon.restitution = 1

// 创建一个物理世界的平面
const planeShape = new CANNON.Box(new CANNON.Vec3(5, 0.1, 5))
// 创建一个刚体
const planeBody = new CANNON.Body({
  mass: 0,
  shape: planeShape,
  material: boxMaterialCon,
  position: new CANNON.Vec3(0, 0, 0),
  type: CANNON.Body.STATIC

})
planeBody.quaternion.setFromAxisAngle(new CANNON.Vec3(1, 0, 0), 0.1)
// 将刚体添加到物理世界
world.addBody(planeBody)


// 创建一个平面几何体
// const planeGeometry = new THREE.PlaneGeometry(10, 10);
const planeGeometry = new THREE.BoxGeometry(10, 0.2, 10);
// 创建一个平面材质
const planeMaterial = new THREE.MeshBasicMaterial({ color: 0xffff00 });
// 创建一个平面网格
const planeMesh = new THREE.Mesh(planeGeometry, planeMaterial);
// x轴旋转90度
planeMesh.rotation.x = 0.1;
// 将网格添加到3D场景
scene.add(planeMesh);

// 物理世界物体
let phyMeshes = []
// 渲染世界物体
let meshs = []

// 创建物理立方体
const boxShape = new CANNON.Box(new CANNON.Vec3(0.5, 0.5, 0.5))

// 创建一个刚体
const boxBody = new CANNON.Body({
  mass: 1,
  shape: boxShape,
  position: new CANNON.Vec3(0, 5, 0),
  material: boxMaterialCon
})
// 将刚体添加到物理世界
world.addBody(boxBody)
phyMeshes.push(boxBody)


// 创建立方体几何体
const boxGeometry = new THREE.BoxGeometry(1, 1, 1)
// 几何立方体材质
const boxMaterial = new THREE.MeshBasicMaterial({ color: 0x00ff00 })
// 立方体物体
const boxMesh = new THREE.Mesh(boxGeometry, boxMaterial)
scene.add(boxMesh)
meshs.push(boxMesh)

// 创建第二个物理立方体
const boxMaterialCon2 = new CANNON.Material("boxSlipperyMaterial")
// 设置材质摩擦系数 
boxMaterialCon2.friction = 0
// 创建一个刚体
const boxBody2 = new CANNON.Body({
  mass: 1,
  shape: boxShape,
  position: new CANNON.Vec3(2, 5, 0),
  material: boxMaterialCon2
})
// 将刚体添加到物理世界
world.addBody(boxBody2)
phyMeshes.push(boxBody2)

// 立方体物体
const boxMesh2 = new THREE.Mesh(boxGeometry, boxMaterial)
boxMesh2.position.set(2, 0, 0)
scene.add(boxMesh2)
meshs.push(boxMesh2)

// 设置立方体材质
const boxBouncyMaterial = new CANNON.Material("boxBouncyMaterial")
// boxBouncyMaterial.friction = 0.1
// boxBouncyMaterial.restitution = 1

const boxBody3 = new CANNON.Body({
  mass: 1,
  shape: boxShape,
  position: new CANNON.Vec3(3, 5, 0),
  material: boxBouncyMaterial
})
// 将刚体添加到物理世界
world.addBody(boxBody3)
phyMeshes.push(boxBody3)


// 立方体物体
const boxMesh3 = new THREE.Mesh(boxGeometry, boxMaterial)
boxMesh2.position.set(2, 0, 0)
scene.add(boxMesh3)
meshs.push(boxMesh3)

// 定义接触材质
const material3toPlane = new CANNON.ContactMaterial(boxMaterialCon, boxBouncyMaterial, {
  friction: 0,
  restitution: 0.1
})
// 将接触材质添加到物理世界
world.addContactMaterial(material3toPlane)


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
