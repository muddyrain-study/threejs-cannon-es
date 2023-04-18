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
boxMaterialCon.friction = 0.2
boxMaterialCon.restitution = 0.1

// 创建一个平面
const planeShape1 = new CANNON.Plane()
const planeBody1 = new CANNON.Body({
  // mass: 0,
  shape: planeShape1,
  material: boxMaterialCon,
  position: new CANNON.Vec3(0, 0, 0),
  type: CANNON.Body.STATIC,
  collisionFilterGroup: GROUP1,
  collisionFilterMask: GROUP1 | GROUP2 | GROUP3
})
planeBody1.quaternion.setFromAxisAngle(new CANNON.Vec3(1, 0, 0), -Math.PI / 2)
// 将刚体添加到物理世界
world.addBody(planeBody1)

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
// 将网格添加到3D场景
scene.add(planeMesh);

// 物理世界物体
let phyMeshes = []
// 渲染世界物体
let meshs = []

// 创建由 2个球 + 1个圆柱体组成的叫哦囊
// 创建一个
const capsuleBody = new CANNON.Body({
  mass: 1,
  material: boxMaterialCon,
  position: new CANNON.Vec3(0, 5, 0),
  collisionFilterGroup: GROUP2,
  collisionFilterMask: GROUP1 | GROUP3 | GROUP4
})
// 创建一个球体几何体
const sphereShape = new CANNON.Sphere(0.5)
// 创建一个圆柱体形状
const cylinderShape = new CANNON.Cylinder(0.5, 0.5, 1.5, 20)
// 将球和圆柱体添加到 body
capsuleBody.addShape(sphereShape, new CANNON.Vec3(0, 0.75, 0))
capsuleBody.addShape(cylinderShape, new CANNON.Vec3(0, 0, 0))
capsuleBody.addShape(sphereShape, new CANNON.Vec3(0, -0.75, 0))

capsuleBody.velocity.set(2, 0, 0);

world.addBody(capsuleBody)
phyMeshes.push(capsuleBody)

// 创建胶囊提网格
const capsuleGeometry = new THREE.CylinderGeometry(0.5, 0.5, 1.5, 20)
// 创建胶囊材质
const capsuleMaterial = new THREE.MeshBasicMaterial({ color: 0x00ff00 })
// 创建胶囊网格
const capsuleMesh = new THREE.Mesh(capsuleGeometry, capsuleMaterial)
// 将胶囊添加到3D场景
capsuleMesh.position.copy(capsuleBody.position)
capsuleMesh.quaternion.copy(capsuleBody.quaternion)
scene.add(capsuleMesh)
meshs.push(capsuleMesh)

// 创建一个球体
const sphereGeometry = new THREE.SphereGeometry(0.5, 20, 20);
const sphereMaterial = new THREE.MeshBasicMaterial({ color: 0x0000ff });
const sphereMesh = new THREE.Mesh(sphereGeometry, sphereMaterial);
sphereMesh.position.set(0, 0.75, 0);
capsuleMesh.add(sphereMesh);
const sphereMesh2 = new THREE.Mesh(sphereGeometry, sphereMaterial);
sphereMesh2.position.set(0, -0.75, 0);
capsuleMesh.add(sphereMesh2);

// 加载模型
const loader = new GLTFLoader()
loader.load('./model/lion.gltf', gltf => {
  let lion = gltf.scene.children[0]
  console.log(lion);
  lion.material = new THREE.MeshBasicMaterial({ color: 0xff0000 })
  scene.add(lion)
  meshs.push(lion)

  // 设置 trimeshbody
  const trimesShape = new CANNON.Trimesh(
    lion.geometry.attributes.position.array,
    lion.geometry.index.array
  )
  const trimeshBody = new CANNON.Body({
    mass: 1,
    shape: trimesShape,
    material: boxMaterialCon,
    position: new CANNON.Vec3(0, 1, 0),
    collisionFilterGroup: GROUP1,
    collisionFilterMask: GROUP1 | GROUP2 | GROUP3
  })
  // 添加到物理世界
  world.addBody(trimeshBody)
  phyMeshes.push(trimeshBody)
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
