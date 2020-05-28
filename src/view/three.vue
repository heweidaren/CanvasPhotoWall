<template>
  <div ref="demo1" @click="fly"></div>
</template>

<script>
import * as THREE from "three";
let action;
export default {
  data: () => ({
    controls: {
      scene: null,
      camera: null,
      renderer: null,
      rotationSpeed: 0.02,
    },
    dots: [],
  }),
  created() {
    this.$nextTick(() => {
      this.init();
    });
  },
  methods: {
    init() {
      this.initMesh();
      // 根据输入的内容填充图片
      this.dots = this.getimgData("FHC");
    },
    initMesh() {
      this.scene = new THREE.Scene(); // 场景
      this.camera = new THREE.PerspectiveCamera(
        45,
        window.innerWidth / window.innerHeight,
        0.1,
        10000
      ); // 相机.视场，长宽比，近面，远面
      this.camera.position.x = 0;
      this.camera.position.y = 0;
      this.camera.position.z = 100;
      this.camera.lookAt(this.scene.position);

      this.renderer = new THREE.WebGLRenderer({ antialias: true }); // 渲染器
      this.renderer.setSize(window.innerWidth, window.innerHeight - 100);
      this.renderer.shadowMapEnabled = true; // 开启阴影

      // let axes = new THREE.AxisHelper(20) // 坐标轴

      // let planeGeometry = new THREE.PlaneGeometry(60, 20, 10, 10) // 生成平面
      // let planeMaterial = new THREE.MeshLambertMaterial({color: 0xffffff}) // 材质
      // let plane = new THREE.Mesh(planeGeometry, planeMaterial)
      // plane.rotation.x = -0.5 * Math.PI
      // plane.position.x = 0
      // plane.position.y = 0
      // plane.position.z = 0
      // plane.receiveShadow = true

      // let cubeGeometry = new THREE.CubeGeometry(10, 10, 10)
      // let cubeMaterial = new THREE.MeshLambertMaterial({color: 0xff0000})
      // this.cube = new THREE.Mesh(cubeGeometry, cubeMaterial)
      // this.cube.position.x = -4
      // this.cube.position.y = 3
      // this.cube.position.z = 0
      // this.cube.castShadow = true

      // var geometry = new THREE.SphereBufferGeometry( 500, 60, 40 );
      // // invert the geometry on the x-axis so that all of the faces point inward
      // geometry.scale( - 1, 1, 1 );

      // var texture = new THREE.TextureLoader().load( require('../assets/o.jpg') );
      // var material = new THREE.MeshBasicMaterial( { map: texture } );

      // var box = new THREE.Mesh( geometry, material );

      let spotLight = new THREE.SpotLight(0xffffff);
      spotLight.position.set(0, 60, 10);
      spotLight.castShadow = true;

      // this.scene.add(axes) // 场景添加坐标轴
      // this.scene.add(plane) // 向该场景中添加物体
      // this.scene.add(this.cube)
      // this.scene.add(box)
      this.scene.add(spotLight);

      this.$refs.demo1.append(this.renderer.domElement);
      this.renderScene();
    },
    getDot(x, y) {
      //
      // console.log(x, y);
      const dx = (x - window.innerWidth / 2) / 4;
      const dy = (window.innerHeight / 2 - y) / 4;
      var texture = new THREE.TextureLoader().load(
        require(`../assets/${Math.ceil(Math.random() * 6)}.jpg`)
      );
      var dotgeometry = new THREE.PlaneGeometry(6, 6, 1);
      var dotmaterial = new THREE.MeshBasicMaterial({
        map: texture,
        side: THREE.DoubleSide,
      });
      var dotplane = new THREE.Mesh(dotgeometry, dotmaterial);
      // dotplane.position.x = x;
      // dotplane.position.y = y;
      // dotplane.position.set(0, 3, 20)
      let ran = Math.random() - 0.5;
      dotplane.origin = {
        x: dx,
        y: dy,
        z: ran * 3,
      };
      dotplane.target = {
        x: (dx * Math.random() * window.innerWidth) / 200,
        y: (dy * Math.random() * window.innerHeight) / 200, //
        z: ran * 600,
      };
      dotplane.position.set(dx, dy, 0);
      this.scene.add(dotplane);
      return dotplane;
    },
    fly() {
      action = action == "out" ? "in" : "out";
    },
    getimgData(text) {
      const c = document.createElement("canvas");
      const ctx = c.getContext("2d");
      c.width = window.innerWidth;
      c.height = window.innerHeight;

      ctx.font = "280px 微软雅黑 bold";
      ctx.fillStyle = "rgba(168,168,168,1)";
      ctx.textAlign = "center";
      ctx.textBaseline = "middle";
      ctx.fillText(text, c.width / 2, c.height / 2);

      var imgData = ctx.getImageData(0, 0, c.width, c.height);

      var dots = [];
      for (var x = 0; x < imgData.width; x += 14) {
        for (var y = 0; y < imgData.height; y += 14) {
          var i = (y * imgData.width + x) * 4;
          if (imgData.data[i] >= 128) {
            // var dot = new Dot(x - 3, y - 3, 0, 3);
            const dot = this.getDot(x, y);
            dots.push(dot);
          }
        }
      }
      console.log(dots);
      return dots;
    },
    renderScene() {
      let { controls, cube, scene, camera, renderer } = this;
      // cube.rotation.x += controls.rotationSpeed
      // cube.rotation.y += controls.rotationSpeed
      // cube.rotation.z += controls.rotationSpeed

      this.dots.forEach((dot) => {
        if (action == "out") {
          if (
            Math.abs(dot.target.x - dot.position.x) < 0.1 &&
            Math.abs(dot.target.y - dot.position.y) < 0.1 &&
            Math.abs(dot.target.z - dot.position.z) < 0.1
          ) {
            // dot.position.x = dot.target.x;
            // dot.position.y = dot.target.y;
            dot.position.z = dot.target.z;
          } else {
            // dot.position.x =
            //   dot.position.x + (dot.target.x - dot.position.x) * 0.02;
            // dot.position.y =
            //   dot.position.y + (dot.target.y - dot.position.y) * 0.02;
            dot.position.z =
              dot.position.z + (dot.target.z - dot.position.z) * 0.02;
          }
        }
        if (action == "in") {
          if (
            Math.abs(dot.origin.x - dot.position.x) < 1.2 &&
            // Math.abs(dot.origin.y - dot.position.y) < 1.2 &&
            Math.abs(dot.origin.z - dot.position.z) < 1.2
          ) {
            // 合成结束
            // dot.position.x = dot.origin.x
            // dot.position.y = dot.origin.y
            // dot.position.z = dot.origin.z
            // console.log(dot.position.y)
          } else {
            // dot.position.x =
            //   dot.position.x + (dot.origin.x - dot.position.x) * 0.05;
            // dot.position.y =
            //   dot.position.y + (dot.origin.y - dot.position.y) * 0.05;
            dot.position.z =
              dot.position.z + (dot.origin.z - dot.position.z) * 0.05;
          }

          // dot.position.y+=(Math.random() * 2 - 1) *0.01
          let speed = 1;

          if (dot.position.y > dot.origin.y + 0.1) {
            speed = 1;
          }

          if (dot.position.y > dot.origin.y - 0.1) {
            speed = -1;
          }
          dot.position.y += speed * Math.random() * 0.007;
        }
      });

      requestAnimationFrame(this.renderScene);
      renderer.render(scene, camera);
    },
  },
};
</script>
