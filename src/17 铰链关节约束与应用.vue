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


// 创建固定的body
const fixedBody = new CANNON.Body({
  mass: 0,
  position: new CANNON.Vec3(0, 10, 0),
});
// 设置box形状
const fixedShape = new CANNON.Box(new CANNON.Vec3(2.5, 2.5, 0.25));
fixedBody.addShape(fixedShape);
// 设置body类型为静态
fixedBody.type = CANNON.Body.STATIC;
// 将body添加到物理世界
world.addBody(fixedBody);
phyMeshes.push(fixedBody);

//threejs的box
const fixedMesh = new THREE.Mesh(
  new THREE.BoxGeometry(5, 5, 0.5),
  new THREE.MeshBasicMaterial({ color: 0x00ff00 })
);
fixedMesh.position.copy(fixedBody.position);
fixedMesh.quaternion.copy(fixedBody.quaternion);
meshes.push(fixedMesh);
scene.add(fixedMesh);

// 创建移动的body
const moveBody = new CANNON.Body({
  mass: 1,
  position: new CANNON.Vec3(0, 4, 0),
});
// 设置box形状
moveBody.addShape(fixedShape);
// 将body添加到物理世界
world.addBody(moveBody);
phyMeshes.push(moveBody);

//threejs的box
const moveMesh = new THREE.Mesh(
  new THREE.BoxGeometry(5, 5, 0.5),
  new THREE.MeshBasicMaterial({ color: 0x00ff00 })
);
moveMesh.position.copy(moveBody.position);
moveMesh.quaternion.copy(moveBody.quaternion);
meshes.push(moveMesh);
scene.add(moveMesh);

// 创建铰链约束
const hingeConstraint = new CANNON.HingeConstraint(fixedBody, moveBody, {
  pivotA: new CANNON.Vec3(0, -3, 0),
  pivotB: new CANNON.Vec3(0, 3, 0),
  axisA: new CANNON.Vec3(1, 0, 0),
  axisB: new CANNON.Vec3(1, 0, 0),
  // collideConnected: true,
});
world.addConstraint(hingeConstraint);

window.addEventListener("click", () => {
  // 创建一个力
  // const force = new CANNON.Vec3(0, 0, -100);
  // // 应用力
  // moveBody.applyForce(force, moveBody.position);
  // 启用马达
  hingeConstraint.enableMotor();
  // 设置马达速度
  hingeConstraint.setMotorSpeed(1000);
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
