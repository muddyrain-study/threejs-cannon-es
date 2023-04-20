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
camera.position.set(0, 20, 20);

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
const groundShape = new CANNON.Box(new CANNON.Vec3(50, 0.5, 50))
let groundBody = new CANNON.Body({
  mass: 0,
  shape: groundShape,
})
world.addBody(groundBody)

// threejs 地面
const groundMesh = new THREE.Mesh(
  new THREE.BoxGeometry(100, 0.5, 100),
  new THREE.MeshBasicMaterial({
    color: 0x888888,
  })
)
scene.add(groundMesh)

// 创建车身
let chassisShape = new CANNON.Box(new CANNON.Vec3(2, 0.5, 1))
let chassisBody = new CANNON.Body({
  mass: 150,
  shape: chassisShape,
})
chassisBody.position.set(0, 5, 0)
world.addBody(chassisBody)
phyMeshes.push(chassisBody)

// 创建threejs车身
let chassisMesh = new THREE.Mesh(
  new THREE.BoxGeometry(4, 1, 2),
  new THREE.MeshBasicMaterial({
    color: 0xffffff,
  })
)
scene.add(chassisMesh)
meshes.push(chassisMesh)

// 创建 拥有悬架的车辆
const vehicle = new CANNON.RaycastVehicle({
  chassisBody
})

// 设置车轮配置
const wheelOptions = {
  // 车轮半径
  radius: 0.5,
  // 车轮宽度
  directionLocal: new CANNON.Vec3(0, -1, 0),
  // 设置悬架的刚度
  suspensionStiffness: 30,
  // 设置悬架的休息长度
  suspensionRestLength: 0.3,
  // 设置车轮的滑动摩擦力
  frictionSlip: 1.4,
  // 设置拉伸的阻尼
  dampingRestitution: 2.3,
  // 设置压缩的阻尼
  dampingCompression: 4.4,
  // 最大的悬架力
  maxSuspensionForce: 100000,
  // 设置最大的悬架行程
  maxSuspensionTravel: 0.2,
  // 设置车轮的转向轴
  axleLocal: new CANNON.Vec3(0, 0, 1),
}

// 添加车轮子
vehicle.addWheel(
  {
    // 配置设置项
    ...wheelOptions,
    // 设置车轮的位置
    chassisConnectionPointLocal: new CANNON.Vec3(-1, 0, 1),
  }
)
// 添加车轮子
vehicle.addWheel(
  {
    // 配置设置项
    ...wheelOptions,
    // 设置车轮的位置
    chassisConnectionPointLocal: new CANNON.Vec3(-1, 0, -1),
  }
)
// 添加车轮子
vehicle.addWheel(
  {
    // 配置设置项
    ...wheelOptions,
    // 设置车轮的位置
    chassisConnectionPointLocal: new CANNON.Vec3(1, 0, 1),
  }
)
// 添加车轮子
vehicle.addWheel(
  {
    // 配置设置项
    ...wheelOptions,
    // 设置车轮的位置
    chassisConnectionPointLocal: new CANNON.Vec3(1, 0, -1),
  }
)

vehicle.addToWorld(world)

// 创建 threejs 车轮
const wheelBodies = []

// 车辆形状
const wheelShape = new CANNON.Cylinder(0.5, 0.5, 0.2, 20)
const wheelGeometry = new THREE.CylinderGeometry(0.5, 0.5, 0.2, 20)
console.log(vehicle.wheelInfos);
for (let i = 0; i < vehicle.wheelInfos.length; i++) {
  const wheel = vehicle.wheelInfos[i]
  const cylinderBody = new CANNON.Body({
    mass: 0,
    shape: wheelShape
  })
  cylinderBody.addShape(wheelShape);
  // world.addBody(cylinderBody)
  phyMeshes.push(cylinderBody)
  wheelBodies.push(cylinderBody)
  const wheelMaterial = new THREE.MeshBasicMaterial({ color: i === 2 || i === 3 ? 0xff0000 : 0x88ff88, wireframe: true })
  const cylinderMesh = new THREE.Mesh(
    wheelGeometry,
    wheelMaterial
  )
  cylinderMesh.rotation.x = -Math.PI / 2
  const wheelObj = new THREE.Object3D()
  wheelObj.add(cylinderMesh)
  scene.add(wheelObj)
  meshes.push(wheelObj)
}

world.addEventListener("postStep", () => {
  for (let i = 0; i < vehicle.wheelInfos.length; i++) {
    vehicle.updateWheelTransform(i);
    const t = vehicle.wheelInfos[i].worldTransform;
    const wheelBody = wheelBodies[i];
    wheelBody.position.copy(t.position);
    wheelBody.quaternion.copy(t.quaternion);
  }
});

const wheelForce = 2000
window.addEventListener('keydown', event => {
  if (event.key == "w") {
    vehicle.applyEngineForce(wheelForce, 2);
    vehicle.applyEngineForce(wheelForce, 3);
  }
  if (event.key == "s") {
    vehicle.applyEngineForce(-wheelForce, 2);
    vehicle.applyEngineForce(-wheelForce, 3);
  }
  if (event.key == "a") {
    vehicle.setSteeringValue(Math.PI / 4, 2);
    vehicle.setSteeringValue(Math.PI / 4, 3);
  }
  if (event.key == "d") {
    vehicle.setSteeringValue(-Math.PI / 4, 2);
    vehicle.setSteeringValue(-Math.PI / 4, 3);
  }
  // 按下r重置车辆
  if (event.key == "r") {
    chassisBody.velocity.set(0, 0, 0);
    chassisBody.angularVelocity.set(0, 0, 0);
    chassisBody.position.set(0, 10, 0);
  }
  // 空格刹车
  if (event.key == " ") {
    vehicle.setBrake(100, 0);
    vehicle.setBrake(100, 1);
  }
})
window.addEventListener("keyup", (event) => {
  if (event.key == "w" || event.key == "s") {
    vehicle.applyEngineForce(0, 2);
    vehicle.applyEngineForce(0, 3);
  }
  if (event.key == "a" || event.key == "d") {
    vehicle.setSteeringValue(0, 2);
    vehicle.setSteeringValue(0, 3);
  }
  if (event.key == " ") {
    vehicle.setBrake(0, 0);
    vehicle.setBrake(0, 1);
  }
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
