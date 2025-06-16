# physics-simulator-
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Universe Simulator</title>
  <style>
    body { margin: 0; overflow: hidden; background: #000; }
    #toolbar, #inspector, #log {
      position: absolute; color: #eee; font-family: sans-serif; padding: 10px;
      background: rgba(0,0,0,0.5); border-radius: 4px;
    }
    #toolbar { top: 10px; left: 10px; }
    #inspector { top: 10px; right: 10px; width: 200px; }
    #log { bottom: 10px; left: 10px; max-height: 150px; overflow: auto; width: 300px; }
    button, label { margin: 4px; color: #eee; background: #333; border: none; padding: 4px; border-radius: 2px; }
    input { width: 100px; }
  </style>
</head>
<body>
  <div id="toolbar">
    <button id="spawnPlanet">Spawn Planet</button>
    <button id="spawnStar">Spawn Star</button><br>
    <label>Gravity: <input id="gravSlider" type="range" min="0.01" max="1" step="0.01" value="0.1"></label><br>
    <label>Time Scale: <input id="timeSlider" type="range" min="0.1" max="5" step="0.1" value="1"></label>
  </div>
  <div id="inspector">Select an object to inspect</div>
  <div id="log"></div>

  <script type="importmap">
  {
    "imports": {
      "three": "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js",
      "three/controls/OrbitControls": "https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/controls/OrbitControls.js"
    }
  }
  </script>

  <script type="module">
    import * as THREE from 'three';
    import { OrbitControls } from 'three/controls/OrbitControls';

    let scene = new THREE.Scene();
    let camera = new THREE.PerspectiveCamera(75, innerWidth / innerHeight, 0.1, 1e7);
    camera.position.z = 500;
    let renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(innerWidth, innerHeight);
    document.body.appendChild(renderer.domElement);

    let controls = new OrbitControls(camera, renderer.domElement);

    // Starfield
    const starGeo = new THREE.BufferGeometry();
    const starCount = 1500;
    const pos = new Float32Array(starCount * 3);
    for (let i = 0; i < pos.length; i++) pos[i] = (Math.random() - 0.5) * 4000;
    starGeo.setAttribute('position', new THREE.BufferAttribute(pos, 3));
    scene.add(new THREE.Points(starGeo, new THREE.PointsMaterial({ color: 0xffffff })));

    // Data
    const bodies = [];
    const G_slider = document.getElementById('gravSlider');
    const t_slider = document.getElementById('timeSlider');
    let selected = null;
    let logEl = document.getElementById('log');

    function log(msg) {
      const div = document.createElement('div'); div.textContent = msg;
      logEl.prepend(div);
      if (logEl.children.length > 50) logEl.removeChild(logEl.lastChild);
    }

    function spawn(mass, size, color) {
      const geom = new THREE.SphereGeometry(size, 24, 24);
      const mat = new THREE.MeshPhongMaterial({ color, emissive: color * 0.4 });
      const mesh = new THREE.Mesh(geom, mat);
      mesh.position.set((Math.random() - 0.5) * 800, (Math.random() - 0.5) * 800, (Math.random() - 0.5) * 800);
      scene.add(mesh);
      bodies.push({
        mesh,
        mass,
        velocity: new THREE.Vector3(
          (Math.random() - 0.5) * 20,
          (Math.random() - 0.5) * 20,
          (Math.random() - 0.5) * 20
        )
      });
      log(`${mass > 50 ? 'Star' : 'Planet'} spawned (mass=${mass})`);
    }

    document.getElementById('spawnPlanet').onclick = () => spawn(20, 10, 0x3366ff);
    document.getElementById('spawnStar').onclick = () => spawn(200, 20, 0xffcc33);

    window.addEventListener('resize', () => {
      camera.aspect = innerWidth / innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(innerWidth, innerHeight);
    });

    // Lighting
    const hemi = new THREE.HemisphereLight(0xaaaaaa, 0x222222, 0.4);
    scene.add(hemi);
    const sunLight = new THREE.PointLight(0xffffff, 1);
    scene.add(sunLight);

    // Raycaster
    const ray = new THREE.Raycaster();
    const mouse = new THREE.Vector2();
    renderer.domElement.addEventListener('pointerdown', e => {
      mouse.x = (e.clientX / innerWidth) * 2 - 1;
      mouse.y = -(e.clientY / innerHeight) * 2 + 1;
      ray.setFromCamera(mouse, camera);
      const hit = ray.intersectObjects(bodies.map(b => b.mesh));
      if (hit.length) {
        selected = bodies.find(b => b.mesh === hit[0].object);
        log(`Selected mass=${selected.mass.toFixed(1)}`);
      }
    });

    // Inspector
    const insp = document.getElementById('inspector');

    function animate() {
      requestAnimationFrame(animate);

      const dt = 0.016 * t_slider.value;
      const G = parseFloat(G_slider.value);

      bodies.forEach(b1 => {
        bodies.forEach(b2 => {
          if (b1 === b2) return;
          const r = new THREE.Vector3().subVectors(b2.mesh.position, b1.mesh.position);
          const distSQ = r.lengthSq() + 25;
          const f = G * b1.mass * b2.mass / distSQ;
          b1.velocity.add(r.normalize().multiplyScalar(f / b1.mass * dt));
        });
      });

      bodies.forEach(b => {
        b.mesh.position.add(b.velocity.clone().multiplyScalar(dt));
      });

      if (selected) {
        const v = selected.velocity.length();
        const ke = 0.5 * selected.mass * selected.velocity.lengthSq();
        const p = selected.mesh.position;
        insp.innerHTML = `
          <b>Selected Object</b><br>
          Mass: ${selected.mass.toFixed(1)}<br>
          Velocity: ${v.toFixed(2)}<br>
          Kinetic Energy: ${ke.toFixed(2)}<br>
          Position: ${p.x.toFixed(1)}, ${p.y.toFixed(1)}, ${p.z.toFixed(1)}
        `;
      }

      sunLight.position.copy(camera.position);
      renderer.render(scene, camera);
    }

    animate();
  </script>
</body>
</html>
