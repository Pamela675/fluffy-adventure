<!DOCTYPE html>
<html lang="pt-br">
<head>
  <meta charset="UTF-8" />
  <title>Corrida de Moto 3D com Obstáculos</title>
  <style>
    body {
      margin: 0;
      overflow: hidden;
    }

    canvas {
      display: block;
    }

    .controls {
      position: absolute;
      bottom: 30px;
      left: 50%;
      transform: translateX(-50%);
      display: flex;
      flex-direction: column;
      gap: 10px;
      z-index: 1;
    }

    .row {
      display: flex;
      justify-content: center;
      gap: 10px;
    }

    .arrow {
      width: 60px;
      height: 60px;
      font-size: 24px;
      font-weight: bold;
      border: none;
      background: rgba(255, 255, 255, 0.8);
      border-radius: 10px;
      cursor: pointer;
    }

    .arrow:active {
      background: rgba(255, 255, 255, 1);
    }
  </style>
</head>
<body>
  <!-- Botões -->
  <div class="controls">
    <div class="row">
      <button class="arrow" id="up">▲</button>
    </div>
    <div class="row">
      <button class="arrow" id="left">◀</button>
      <button class="arrow" id="down">▼</button>
      <button class="arrow" id="right">▶</button>
    </div>
  </div>

  <script type="module">
    import * as THREE from 'https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js';

    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x87ceeb);

    const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
    camera.position.set(0, 5, 10);

    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    const light = new THREE.DirectionalLight(0xffffff, 1);
    light.position.set(5, 10, 10);
    scene.add(light);

    const pista = new THREE.Mesh(
      new THREE.PlaneGeometry(10, 2000),
      new THREE.MeshStandardMaterial({ color: 0x333333 })
    );
    pista.rotation.x = -Math.PI / 2;
    scene.add(pista);

    const moto = new THREE.Group();
    const corpo = new THREE.Mesh(
      new THREE.BoxGeometry(1, 0.5, 2),
      new THREE.MeshStandardMaterial({ color: 'yellow' })
    );
    corpo.position.y = 0.5;
    moto.add(corpo);

    function criarRoda(x, z) {
      const roda = new THREE.Mesh(
        new THREE.CylinderGeometry(0.3, 0.3, 0.2, 16),
        new THREE.MeshStandardMaterial({ color: 'black' })
      );
      roda.rotation.z = Math.PI / 2;
      roda.position.set(x, 0.2, z);
      return roda;
    }

    moto.add(criarRoda(-0.5, -0.8));
    moto.add(criarRoda(0.5, -0.8));
    moto.add(criarRoda(-0.5, 0.8));
    moto.add(criarRoda(0.5, 0.8));
    moto.position.set(0, 0.3, 0);
    scene.add(moto);

    // Obstáculos
    const obstaculos = [];
    const explosoes = [];
    for (let i = 20; i <= 2000; i += 40) {
      const obs = new THREE.Mesh(
        new THREE.BoxGeometry(1, 1, 1),
        new THREE.MeshStandardMaterial({ color: 'red' })
      );
      obs.position.set(Math.random() * 6 - 3, 0.5, -i);
      scene.add(obs);
      obstaculos.push(obs);
    }
    

    // Teclado
    const keys = {};
    document.addEventListener('keydown', (e) => keys[e.key] = true);
    document.addEventListener('keyup', (e) => keys[e.key] = false);

    // Botões visuais
    const held = { up: false, down: false, left: false, right: false };

    function configurarBotao(id, direcao) {
      const btn = document.getElementById(id);
      btn.addEventListener('pointerdown', () => held[direcao] = true);
      btn.addEventListener('pointerup', () => held[direcao] = false);
      btn.addEventListener('pointerleave', () => held[direcao] = false);
    }

    configurarBotao('up', 'up');
    configurarBotao('down', 'down');
    configurarBotao('left', 'left');
    configurarBotao('right', 'right');

    // Loop
    function animate() {
      requestAnimationFrame(animate);

      // Movimento contínuo para frente
      moto.position.z -= 0.3;

      // Controles manuais
      if (keys['ArrowLeft'] || held.left) moto.position.x -= 0.2;
      if (keys['ArrowRight'] || held.right) moto.position.x += 0.2;

      // Colisão com obstáculos
      for (const obs of obstaculos) {
        const dist = moto.position.distanceTo(obs.position);
        if (dist < 1.5) {
          obs.material.color.set(0x000000);
          moto.position.set(0, 0.3, 0); // reinicia posição
          break;
        }
      }

      // Câmera segue a moto
      camera.position.set(moto.position.x, 5, moto.position.z + 10);
      camera.lookAt(moto.position);

      renderer.render(scene, camera);
    }

    animate();

    // Ajuste de tela
    window.addEventListener('resize', () => {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });
  </script>
</body>
</html>