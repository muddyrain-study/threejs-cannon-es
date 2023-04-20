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
camera.position.set(0, 50, 50);

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

// 创建地面
const groundShape = new CANNON.Plane();
const groundBody = new CANNON.Body({
  mass: 0,
  shape: groundShape,
  material: new CANNON.Material(),
});
groundBody.quaternion.setFromAxisAngle(new CANNON.Vec3(1, 0, 0), -Math.PI / 2);
world.addBody(groundBody);
phyMeshes.push(groundBody);

const groundMesh = new THREE.Mesh(
  new THREE.PlaneGeometry(100000, 100000),
  new THREE.MeshBasicMaterial({
    color: 0x666666,
  })
);
groundMesh.rotation.x = -Math.PI / 2;
scene.add(groundMesh);
meshes.push(groundMesh);

// 创建车身子
const chassisShape = new CANNON.Box(new CANNON.Vec3(5, 0.5, 2));
const chassisBody = new CANNON.Body({
  mass: 100,
  shape: chassisShape,
  position: new CANNON.Vec3(0, 5, 0),
});
// world.addBody(chassisBody);
phyMeshes.push(chassisBody);

// threejs的车身
const chassisMesh = new THREE.Mesh(
  new THREE.BoxGeometry(10, 1, 4),
  new THREE.MeshBasicMaterial({
    color: 0x660066
  })
)
scene.add(chassisMesh);
meshes.push(chassisMesh);

// 创建钢性的车子
const vehicle = new CANNON.RigidVehicle({
  chassisBody: chassisBody,
})


// 创建轮子
// const wheelShape = new CANNON.Cylinder(0.5, 0.5, 1, 20)
const wheelShape = new CANNON.Sphere(1.5)

const wheelBody1 = new CANNON.Body({
  mass: 1,
  shape: wheelShape
})
vehicle.addWheel({
  body: wheelBody1,
  // 车轮方向
  position: new CANNON.Vec3(-4, -0.5, 3.5),
  // 旋转轴
  axis: new CANNON.Vec3(0, 0, -1),
  // 轮子方向
  direction: new CANNON.Vec3(0, -1, 0),
})
// world.addBody(wheelBody1)
phyMeshes.push(wheelBody1)

// threejs 的车轮
const wheelGeometry = new THREE.SphereGeometry(1.5, 8, 8)
const wheelMaterial = new THREE.MeshBasicMaterial({
  color: 0x660000,
  wireframe: true
})
const wheelMesh1 = new THREE.Mesh(
  wheelGeometry,
  wheelMaterial
)
scene.add(wheelMesh1)
meshes.push(wheelMesh1)

const wheelBody2 = new CANNON.Body({
  mass: 1,
  shape: wheelShape
})
vehicle.addWheel({
  body: wheelBody2,
  // 车轮方向
  position: new CANNON.Vec3(4, -0.5, 3.5),
  // 旋转轴
  axis: new CANNON.Vec3(0, 0, -1),
  // 轮子方向
  direction: new CANNON.Vec3(0, -1, 0),
})
// world.addBody(wheelBody2)
phyMeshes.push(wheelBody2)

// threejs 的车轮
const wheelMesh2 = new THREE.Mesh(
  wheelGeometry,
  wheelMaterial
)
scene.add(wheelMesh2)
meshes.push(wheelMesh2)

const wheelBody3 = new CANNON.Body({
  mass: 1,
  shape: wheelShape
})
vehicle.addWheel({
  body: wheelBody3,
  // 车轮方向
  position: new CANNON.Vec3(-4, -0.5, -3.5),
  // 旋转轴
  axis: new CANNON.Vec3(0, 0, -1),
  // 轮子方向
  direction: new CANNON.Vec3(0, -1, 0),
})
// world.addBody(wheelBody3)
phyMeshes.push(wheelBody3)

// threejs 的车轮
const wheelMesh3 = new THREE.Mesh(
  wheelGeometry,
  wheelMaterial
)
scene.add(wheelMesh3)
meshes.push(wheelMesh3)

const wheelBody4 = new CANNON.Body({
  mass: 1,
  shape: wheelShape
})
vehicle.addWheel({
  body: wheelBody4,
  // 车轮方向
  position: new CANNON.Vec3(4, -0.5, -3.5),
  // 旋转轴
  axis: new CANNON.Vec3(0, 0, -1),
  // 轮子方向
  direction: new CANNON.Vec3(0, -1, 0),
})
// world.addBody(wheelBody4)
phyMeshes.push(wheelBody4)

// threejs 的车轮
const wheelMesh4 = new THREE.Mesh(
  wheelGeometry,
  wheelMaterial
)
scene.add(wheelMesh4)
meshes.push(wheelMesh4)

// 设置完毕车身和所以车轮 将车子添加到世界中
vehicle.addToWorld(world)

// 轮胎压力
const wheelForce = 100
// 控制车子
window.addEventListener("keydown", (event) => {
  console.log(event.key);
  switch (event.key) {
    case "w": {
      // 设置轮子力量
      vehicle.setWheelForce(-wheelForce, 0)
      vehicle.setWheelForce(-wheelForce, 2)
      break;
    }
    case "s": {
      // 设置轮子力量
      vehicle.setWheelForce(wheelForce, 0)
      vehicle.setWheelForce(wheelForce, 2)
      break;
    }
    case "a": {
      vehicle.setSteeringValue(Math.PI / 4, 0)
      vehicle.setSteeringValue(Math.PI / 4, 2)
      break;
    }
    case "d": {
      vehicle.setSteeringValue(-Math.PI / 4, 0)
      vehicle.setSteeringValue(-Math.PI / 4, 2)
      break;
    }

  }
})

window.addEventListener("keyup", (event) => {
  console.log(event.key);
  switch (event.key) {
    case "w": {
      // 设置轮子力量
      vehicle.setWheelForce(0, 0)
      vehicle.setWheelForce(0, 2);
      break;
    }
    case "s": {
      // 设置轮子力量
      vehicle.setWheelForce(0, 0)
      vehicle.setWheelForce(0, 2);
      break;
    }
    case "a": {
      vehicle.setSteeringValue(0, 0)
      vehicle.setSteeringValue(0, 2)
      break;

    }
    case "d": {
      vehicle.setSteeringValue(0, 0)
      vehicle.setSteeringValue(0, 2)
      break;
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
