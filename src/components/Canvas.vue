<template>
  <div>
    <canvas id="three"></canvas>
  </div>
</template>

<script>
import { defineProps, reactive } from "vue";
import * as THREE from "three";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
// import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer';
// import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass';
// import { GlitchPass } from 'three/examples/jsm/postprocessing/GlitchPass';

export default {
  mounted() {
    this.init();
  },
  methods: {
    init() {
      const scene = new THREE.Scene();
      scene.background = new THREE.Color("#eee");
      scene.fog = new THREE.Fog("#eee", 500, 2500);
      const canvas = document.querySelector("#three");
      const renderer = new THREE.WebGLRenderer({ canvas, antialias: true });
      renderer.shadowMap.enabled = true;
    
      const camera = new THREE.PerspectiveCamera(
        10,
        window.innerWidth / window.innerHeight,
        1,
        1000
      );
      camera.position.set(-60, 120, 400);
      camera.lookAt(new THREE.Vector3(0,0,0));

      const gltfLoader = new GLTFLoader();
      gltfLoader.load("/models/kiana_honkaie/scene.gltf", (gltf) => {
        let model = gltf.scene.children[0];
        // let texture = new THREE.TextureLoader().load(
        //   "/models/kiana_honkaie/textures/kianaModel001_Material001_baseColor.png"
        // );
        // texture.flipY = false;
        // const material = new THREE.MeshStandardMaterial({
        //   map: texture,
        // });
        // model.marterial = material;

        model.traverse((o) => {
          if (o.isMesh) {
            o.castShadow = true;
            o.receiveShadow = true;
          }
        });

        scene.add(model);
      });

      const hemLight = new THREE.HemisphereLight(0xffffff, 0xffffff, 1);
      hemLight.position.set(0, 48, 0);
      scene.add(hemLight);

      const dirLight = new THREE.DirectionalLight(0xffffff, 0.6);
      dirLight.position.set(20, 8, 5);
      dirLight.castShadow = true;
      dirLight.shadow.mapSize = new THREE.Vector2(1024, 1024);
      scene.add(dirLight);

      let floorGeometry = new THREE.PlaneGeometry(8000, 8000);
      let floorMaterial = new THREE.MeshPhongMaterial({
        color: 0x857ebb,
        shininess: 0,
      });

      let floor = new THREE.Mesh(floorGeometry, floorMaterial);
      floor.rotation.x = -0.5 * Math.PI;
      floor.receiveShadow = true;
      floor.position.y = -9.5;
      scene.add(floor);

      const controls = new OrbitControls(camera, renderer.domElement);
      controls.enableDamping = true;
      // controls.target.copy(camera.position);
      controls.target.set(0,50,0);

      function resizeRendererToDisplaySize(renderer) {
        const canvas = renderer.domElement;
        var width = window.innerWidth;
        var height = window.innerHeight;
        var canvasPixelWidth = canvas.width / window.devicePixelRatio;
        var canvasPixelHeight = canvas.height / window.devicePixelRatio;

        const needResize =
          canvasPixelWidth !== width || canvasPixelHeight !== height;
        if (needResize) {
          renderer.setSize(width, height, false);
        }
        return needResize;
      }

      function animate() {
        controls.update();
        renderer.render(scene, camera);
        requestAnimationFrame(animate);
        // composer.render();

        if (resizeRendererToDisplaySize(renderer)) {
          const canvas = renderer.domElement;
          camera.aspect = canvas.clientWidth / canvas.clientHeight;
          camera.updateProjectionMatrix();
        }
      }
      animate();
    },
  },
};
</script>

<style scoped>
#three {
  width: 100%;
  height: 100%;
  position: fixed;
  left: 0;
  top: 0;
}

#text {
  display: block;
  position: absolute;
  left: 50%;
  top: 50%;
  color: white;
}
</style>
