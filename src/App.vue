<template>
  <div>
    <h1>{{ percentage }}</h1>
  </div>
</template>

<script setup>
//import three.js and cannon-es
import * as THREE from "three";
import * as CANNON from "cannon-es";
import gsap from "gsap";
// orbit control component from three
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
// model loader
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader";
import { ref } from "vue";

//kicking force
let percentage = ref(30);
gsap.to(percentage, {
  duration: 1,
  value: 100,
  ease: "linear",
  repeat: -1,
  onUpdate: () => {
    percentage.value = Math.floor(percentage.value);
  },
});

//scene init
const scene = new THREE.Scene();
//camera init 75 degree angle , perspective ratio , closest and farthest observing distance
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  100
);
//camera positioning
camera.position.set(4, 2, 0);
camera.updateProjectionMatrix();
//renderer init
const renderer = new THREE.WebGLRenderer({
  // anti aliasing
  antialias: true,
  logarithmicDepthBuffer: true,
});
//set renderer size
renderer.setSize(window.innerWidth, window.innerHeight);
//toner
renderer.toneMapping = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 0.5;
//shadow
renderer.shadowMap.enable = true;
//shadow type
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
document.body.appendChild(renderer.domElement);

// controller init

const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.dampingFactor = 0.05;

//load texture

const textureLoader = new THREE.TextureLoader();
textureLoader.load("./texture/outdoor.png", (texture) => {
  texture.mapping = THREE.EquirectangularReflectionMapping;
  //environment texture
  scene.background = texture;
  scene.environment = texture;
  scene.backgroundBlurriness = 0.2;
});

let dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath("/draco/");
//load models

const gltfLoader = new GLTFLoader();
gltfLoader.setDRACOLoader(dracoLoader);
gltfLoader.load("./model/playground02.glb", (gltf) => {
  const model = gltf.scene;

  model.traverse((child) => {
    if (child.isMesh && child.name.search(/Solid/) == -1) {
      child.caseShadow = true;
      child.receiveShadow = true;

      // trimesh category
      const trimesh = new CANNON.Trimesh(
        child.geometry.attributes.position.array,
        child.geometry.index.array
      );
      // create body
      const trimeshBody = new CANNON.Body({
        mass: 0,
        shape: trimesh,
      });
      //get world position and quaternion to physical world
      trimeshBody.position.copy(child.getWorldPosition(new THREE.Vector3()));
      trimeshBody.quaternion.copy(
        child.getWorldQuaternion(new THREE.Quaternion())
      );
      // add it to the world
      world.addBody(trimeshBody);

      //goal
      if(child.name == "door") {
        child.material = new THREE.MeshBasicMaterial({
          color:0x000000,
          opacity: 0,
          transparent: true,
        })
      }

    }
    if (child.name == "Soccer_Ball") {
      ball = child;
      // create ball
      const ballShape = new CANNON.Sphere(0.15);
      // create ball's rigid body
      ballBody = new CANNON.Body({
        mass: 1,
        position: new CANNON.Vec3(0, 5, 0),
        shape: ballShape,
      });
      // add rigid body to the world
      world.addBody(ballBody);
    }
    setTimeout(() => {
      ballBody.position.set(0, 15, 0);
      ballBody.velocity.set(0, 0, 0);
      ballBody.angularVelocity.set(0, 0, 0);
    }, 2000);
  });
  scene.add(model);
});

// add lighting
const spotLight = new THREE.SpotLight(0xffffff);
spotLight.position.set(10, 50, 0);
spotLight.castShadow = true;
spotLight.shadow.mapSize.width = 2048;
spotLight.shadow.mapSize.height = 2048;
spotLight.shadow.camera.near = 0.5;
spotLight.shadow.camera.far = 500;
spotLight.shadow.camera.fov = 30;
spotLight.shadow.bias = -0.000008;
spotLight.intensity = 2.5;
scene.add(spotLight);

// physics init
const world = new CANNON.World();
// set gravity
world.gravity.set(0, -9.82, 0);
let ball, ballBody;

let clock = new THREE.Clock();

//render function
const render = () => {
  //time update
  let delta = clock.getDelta();
  world.step(delta);
  //making the ball move with the body
  if (ball && ballBody) {
    ball.position.copy(ballBody.position);
    ball.quaternion.copy(ballBody.quaternion);
  }

  renderer.render(scene, camera);
  controls.update();
  requestAnimationFrame(render);
};
render();

let isClick = false;
window.addEventListener("click", () => {
  if (isClick) return;
  isClick = true;
  ballBody.applyForce(
    new CANNON.Vec3(
      -10 * percentage.value,
      6 * percentage.value,
      (Math.random() - 0.5) * percentage.value
    ),
    ballBody.position
  );
  setTimeout(() => {
    isClick = false;
    ballBody.position.set(0, 15, 0);
    ballBody.velocity.set(0, 0, 0);
    ballBody.angularVelocity.set(0, 0, 0);
  }, 2000);
});
</script>

<style>
* {
  margin: 0;
  padding: 0;
}
canvas {
  position: fixed;
  left: 0;
  top: 0;
  width: 100vw;
  height: 100vh;
}
h1 {
  position: fixed;
  left: 0;
  top: 0;
  z-index: 10;
  color: white;
}
</style>
