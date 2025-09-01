# Naksha
https://your-username.github.io/3d-naksha/naksha_3d_view.html<!doctype html>
<html lang="hi">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>3D Floorplan - 25x45 ft (Vastu)</title>
  <style>
    html,body{height:100%;margin:0}
    #app{height:100%;display:flex;flex-direction:row}
    #viewport{flex:1;background:#e8eef6}
    #panel{width:340px; background:#fff; box-shadow:0 0 12px rgba(0,0,0,0.12); padding:12px; font-family:system-ui,Segoe UI,Roboto,'Noto Sans',sans-serif}
    h2{margin:6px 0 12px;font-size:18px}
    .room-btn{display:block;width:100%;padding:8px;margin:6px 0;border:1px solid #ddd;border-radius:6px;background:#f7f9fc;cursor:pointer;text-align:left}
    .small{font-size:13px;color:#555}
    .legend{display:flex;gap:8px;flex-wrap:wrap;margin-top:8px}
    .chip{padding:6px 8px;border-radius:6px;background:#f1f5f9;border:1px solid #e2e8f0;font-size:13px}
  </style>
</head>
<body>
  <div id="app">
    <div id="viewport"></div>
    <div id="panel">
      <h2>3D Naksha — 25×45 ft (Vastu)</h2>
      <div class="small">Scale: <strong>1 ft = 10 units</strong>. Use mouse to orbit/zoom.</div>
      <div style="margin-top:12px">
        <button class="room-btn" data-room="parking">🔲 Parking (Front)</button>
        <button class="room-btn" data-room="living">🛋️ Living / Drawing</button>
        <button class="room-btn" data-room="kitchen">🍽️ Kitchen (SE)</button>
        <button class="room-btn" data-room="pooja">🕉️ Pooja (NE)</button>
        <button class="room-btn" data-room="master">🛏️ Master Bedroom (SW)</button>
        <button class="room-btn" data-room="bed2">🛏️ Bedroom 2</button>
        <button class="room-btn" data-room="guest">👤 Guest Room</button>
        <button class="room-btn" data-room="stairs">↕️ Stairs</button>
      </div>
      <div style="margin-top:14px">
        <button id="toggle-grid" class="room-btn">Toggle Grid</button>
        <button id="reset-camera" class="room-btn">Reset Camera</button>
      </div>
      <div style="margin-top:12px">
        <div class="small">Legend:</div>
        <div class="legend">
          <div class="chip">Walls</div>
          <div class="chip">Floor</div>
          <div class="chip">Doors</div>
          <div class="chip">Bathrooms</div>
        </div>
      </div>
      <div style="margin-top:12px;font-size:13px;color:#333">
        Notes: ये एक सामान्य 3D वास्तु-कम्प्लायंट विज़ुअलाइज़ेशन है। मेटरियल और फिटिंग को आप बदलवा सकते हैं।
      </div>
    </div>
  </div>

  <!-- Three.js CDN -->
  <script src="https://cdn.jsdelivr.net/npm/three@0.156.0/build/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.156.0/examples/js/controls/OrbitControls.js"></script>
  <script>
  (function(){
    const scale = 10;
    const VIEW_WIDTH_FT = 25;
    const VIEW_DEPTH_FT = 45;

    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0xe8eef6);

    const camera = new THREE.PerspectiveCamera(45, (window.innerWidth-340)/window.innerHeight, 1, 5000);
    camera.position.set(400, 400, 400);

    const renderer = new THREE.WebGLRenderer({antialias:true});
    renderer.setSize(window.innerWidth-340, window.innerHeight);
    document.getElementById('viewport').appendChild(renderer.domElement);

    window.addEventListener('resize', ()=>{
      renderer.setSize(window.innerWidth-340, window.innerHeight);
      camera.aspect = (window.innerWidth-340)/window.innerHeight;
      camera.updateProjectionMatrix();
    });

    // lights
    scene.add(new THREE.HemisphereLight(0xffffff, 0x888888, 0.9));
    const dir = new THREE.DirectionalLight(0xffffff,0.6);
    dir.position.set(200,400,200); scene.add(dir);

    // floor
    const floorGeo = new THREE.PlaneGeometry(VIEW_WIDTH_FT*scale, VIEW_DEPTH_FT*scale);
    const floorMat = new THREE.MeshStandardMaterial({color:0xf7fafc, side:THREE.DoubleSide});
    const floor = new THREE.Mesh(floorGeo, floorMat);
    floor.rotation.x = -Math.PI/2;
    scene.add(floor);

    // grid
    let grid = new THREE.GridHelper(500,50);
    grid.position.y=0.01;
    scene.add(grid);

    // helper room maker
    function makeRoom(x_ft,z_ft,w_ft,d_ft,h_ft,label,color){
      const geo=new THREE.BoxGeometry(w_ft*scale,h_ft*scale,d_ft*scale);
      const mat=new THREE.MeshStandardMaterial({color:color||0xbfd8ff});
      const mesh=new THREE.Mesh(geo,mat);
      mesh.position.set((x_ft-VIEW_WIDTH_FT/2+w_ft/2)*scale,h_ft*scale/2,(z_ft-VIEW_DEPTH_FT/2+d_ft/2)*scale);
      scene.add(mesh);

      const canvas=document.createElement('canvas');
      canvas.width=256;canvas.height=64;
      const ctx=canvas.getContext('2d');
      ctx.fillStyle='rgba(255,255,255,0.9)';ctx.fillRect(0,0,256,64);
      ctx.fillStyle='#111';ctx.font='22px sans-serif';ctx.textAlign='center';
      ctx.fillText(label,128,38);
      const tex=new THREE.CanvasTexture(canvas);
      const spriteMat=new THREE.SpriteMaterial({map:tex,depthTest:false});
      const sprite=new THREE.Sprite(spriteMat);
      sprite.scale.set(120,30,1);
      sprite.position.set(mesh.position.x,h_ft*scale+20,mesh.position.z);
      scene.add(sprite);

      return mesh;
    }

    const rooms = {
      parking: makeRoom(0,0,25,10,0.5,"Parking",0xa8d0a6),
      living: makeRoom(0,10,15,16,9,"Living",0xbfd8ff),
      kitchen: makeRoom(15,25,10,12,8,"Kitchen(SE)",0xffd6a5),
      pooja: makeRoom(19,6,6,6,8,"Pooja(NE)",0xffe5f2),
      master: makeRoom(0,26,12,12,9,"Master(SW)",0xd1c4e9),
      bed2: makeRoom(12,20,13,12,9,"Bedroom2",0xc8e6c9),
      guest: makeRoom(15,10,10,9,8,"Guest",0xfff59d),
      stairs: makeRoom(11,12,4,8,12,"Stairs",0xcccccc)
    };

    const controls=new THREE.OrbitControls(camera,renderer.domElement);
    controls.target.set(0,30,0); controls.update();

    function animate(){requestAnimationFrame(animate);renderer.render(scene,camera);} animate();

    document.querySelectorAll('.room-btn').forEach(btn=>{
      btn.addEventListener('click',()=>{
        const id=btn.getAttribute('data-room');
        if(!rooms[id]) return;
        const pos=rooms[id].position.clone().add(new THREE.Vector3(200,150,200));
        controls.target.copy(rooms[id].position);
        camera.position.copy(pos);
      });
    });

    document.getElementById('toggle-grid').addEventListener('click',()=>{grid.visible=!grid.visible;});
    document.getElementById('reset-camera').addEventListener('click',()=>{camera.position.set(400,400,400);controls.target.set(0,30,0);});
  })();
  </script>
</body>
</html>

