<script>
  import "./css/variables.css";
  import "./css/reset.css";

  import { onMount } from "svelte";
  import * as THREE from "three";
  import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";

  let animContainer;

  onMount(() => {
    let scene = new THREE.Scene();
    let camera = new THREE.PerspectiveCamera(
      70,
      window.innerWidth / window.innerHeight,
      0.01,
      1000
    );
    let renderer = new THREE.WebGLRenderer({
      canvas: animContainer,
    });
    renderer.setPixelRatio(window.devicePixelRatio);
    renderer.setSize(window.innerWidth, window.innerHeight);
    camera.position.setX(0);
    camera.position.setY(10);
    camera.position.setZ(25);

    let allowedGeometries = [new THREE.TorusGeometry(14, 3, 12, 24)];
    let material = new THREE.MeshBasicMaterial({
      color: "hsl(270, 70%, 50%)",
      wireframe: true,
    });
    let shape = new THREE.Mesh(allowedGeometries[0], material);
    scene.add(shape);

    let gs = 1000;
    let grid = new THREE.GridHelper(gs, gs / 25);
    scene.add(grid);

    const controls = new OrbitControls(camera, renderer.domElement);

    let backToStart = false;
    let nsLoc = shape.position.z;
    let shapeList = [{ item: shape, spd: 0.01 }];

    function animate() {
      requestAnimationFrame(animate);

      controls.update();
      shapeList.forEach((s) => {
        s.item.rotation.x += s.spd;
        s.item.rotation.y += s.spd;
      });

      if (camera.position.z > 45 * 6 + 45) {
        backToStart = true;
      }

      if (camera.position.z < 10) {
        backToStart = false;
      }

      if (!backToStart) {
        camera.position.z += 0.5;
      } else {
        camera.position.z -= 0.5;
      }

      if (shapeList.length < 6) {
        nsLoc += 45;
        genNewShape(nsLoc);
      }

      renderer.render(scene, camera);
    }

    function genNewShape(zpos) {
      let nShape = new THREE.Mesh(
        allowedGeometries[randBW(0, allowedGeometries.length - 1)],
        material
      );
      nShape.position.z = zpos;
      scene.add(nShape);
      shapeList.push({ item: nShape, spd: randBW(1, 5) * 0.01 });
    }

    function randBW(min, max) {
      return Math.floor(Math.random() * (max - min + 1) + min);
    }

    animate();
  });
</script>

<canvas class="animBg" bind:this={animContainer} />
<p class="ps">
  PS. You can hold and drag your mouse to change the camera angle
</p>

<style>
  :global(body) {
    background-color: var(--bg);
    color: var(--fg);
  }

  .animBg {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: -10;
  }

  .ps {
    position: fixed;
    top: 0;
    right: 0;
    width: 220px;
    font-size: 0.8rem;
    padding: 1rem;
  }
</style>
