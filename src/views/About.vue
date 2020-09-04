<template>
  <div class="about">
    <div class="three" id="world" @mousemove="handleMouseMove"></div>
  </div>
</template>

<script>
import "@/scss/About/index.scss";
import * as THREE from "three";
export default {
  data() {
    return {
      Colors: {
        red: "#f25346",
        white: "#d8d0d1",
        brown: "#59332e",
        pink: "#f5986e",
        brownDark: "#23190f",
        blue: "#68c3c0",
      },

      //场景
      scene: {},
      //摄像头
      camera: {},
      //视野
      fieldOfView: {},
      //屏幕高宽比
      aspectRatio: {},
      //飞机平面
      nearPlane: {},
      //云平面
      farPlane: {},
      //高
      HEIGHT: "",
      //宽
      WIDTH: "",
      //渲染器
      renderer: {},
      //容器
      container: {},

      hemisphereLight: {},
      shadowLight: {},

      sea: {},
      cloud: {},
      sky: {},
      airplane: {},

      mousePos: { x: 0, y: 0 },
    };
  },
  methods: {
    initThree() {
      //创建绘画容器
      // let canvas = this.$refs.three;

      // 创建场景，相机和渲染器

      this.createScene();

      // 添加光源

      this.createLights();

      // 添加对象

      this.createPlane();

      this.createSea();

      this.createSky();

      // 调用循环函数，在每帧更新对象的位置和渲染场景

      this.loop();
    },

    createScene() {
      //获取屏幕的宽度和高度，
      //使用它们来设置相机的宽高比
      //以及渲染器的大小。
      this.HEIGHT = window.innerHeight;
      this.WIDTH = window.innerWidth;

      // 创建场景
      this.scene = new THREE.Scene();

      //在场景中添加雾化效果； 与颜色相同
      //样式表中使用的背景色
      this.scene.fog = new THREE.Fog(0xf7d9aa, 100, 950);

      // Create the camera
      this.aspectRatio = this.WIDTH / this.HEIGHT;
      this.fieldOfView = 60;
      this.nearPlane = 1;
      this.farPlane = 10000;

      /**
       * PerspectiveCamera 透视相机
       * @param fieldOfView 视角
       * @param aspectRatio 纵横比
       * @param nearPlane 近平面
       * @param farPlane 远平面
       */
      this.camera = new THREE.PerspectiveCamera(
        this.fieldOfView,
        this.aspectRatio,
        this.nearPlane,
        this.farPlane
      );

      // Set the position of the camera
      this.camera.position.x = 0;
      this.camera.position.z = 200;
      this.camera.position.y = 100;

      // Create the renderer
      this.renderer = new THREE.WebGLRenderer({
        //允许透明度显示渐变背景
        //我们在CSS中定义
        alpha: true,

        //激活抗锯齿； 这表现不佳
        //但是，由于我们的项目是基于低聚的，因此应该没问题:)
        antialias: true,
      });

      //定义渲染器的大小； 在这种情况下，
      //它会填满整个屏幕
      this.renderer.setSize(this.WIDTH, this.HEIGHT);

      // 启动阴影渲染
      this.renderer.shadowMap.enabled = true;

      //将渲染的dom节点 添加到 world 标签中
      this.container = document.getElementById("world");
      this.container.appendChild(this.renderer.domElement);

      //当窗口发生改变时, 修改 渲染器节点大小和 摄像机的纵横比
      window.addEventListener("resize", this.handleWindowResize, false);
    },

    //当窗口变化时, 修改参数
    handleWindowResize() {
      // update height and width of the renderer and the camera
      this.HEIGHT = window.innerHeight;
      this.WIDTH = window.innerWidth;
      this.renderer.setSize(this.WIDTH, this.HEIGHT);
      this.camera.aspect = this.WIDTH / this.HEIGHT;
      //更新投影矩阵
      this.camera.updateProjectionMatrix();
    },

    //设置灯光和阴影
    createLights() {
      // A hemisphere light is a gradient colored light;
      // the first parameter is the sky color, the second parameter is the ground color,
      // the third parameter is the intensity of the light
      this.hemisphereLight = new THREE.HemisphereLight(0xaaaaaa, 0x000000, 0.9);

      // A directional light shines from a specific direction.
      // It acts like the sun, this means this all the rays produced are parallel.
      this.shadowLight = new THREE.DirectionalLight(0xffffff, 0.9);

      // Set the direction of the light
      this.shadowLight.position.set(150, 350, 350);

      // Allow shadow casting
      this.shadowLight.castShadow = true;

      // define the visible area of the projected shadow
      this.shadowLight.shadow.camera.left = -200;
      this.shadowLight.shadow.camera.right = 400;
      this.shadowLight.shadow.camera.top = 200;
      this.shadowLight.shadow.camera.bottom = -400;
      this.shadowLight.shadow.camera.near = 0;
      this.shadowLight.shadow.camera.far = 2000;

      // define the resolution of the shadow; the higher the better,
      // but also the more expensive and less performant
      this.shadowLight.shadow.mapSize.width = 2048;
      this.shadowLight.shadow.mapSize.height = 2048;

      // to activate the lights, just add them to the scene
      this.scene.add(this.hemisphereLight);
      this.scene.add(this.shadowLight);
    },

    Sea(that) {
      // create the geometry (shape) of the cylinder;
      // the parameters are:
      // radius top, radius bottom, height, number of segments on the radius, number of segments vertically
      var geom = new THREE.CylinderGeometry(600, 600, 800, 40, 10);

      // rotate the geometry on the x axis
      geom.applyMatrix(new THREE.Matrix4().makeRotationX(-Math.PI / 2));

      // create the material
      var mat = new THREE.MeshPhongMaterial({
        color: that.Colors.blue,
        transparent: true,
        opacity: 0.6,
        shading: THREE.FlatShading,
      });

      // To create an object in Three.js, we have to create a mesh
      // which is a combination of a geometry and some material
      var mesh = new THREE.Mesh(geom, mat);

      // Allow the sea to receive shadows
      mesh.receiveShadow = true;
      return {
        geom,
        mat,
        mesh,
      };
    },

    Cloud(that) {
      // Create an empty container this will hold the different parts of the cloud
      var mesh = new THREE.Object3D();

      // create a cube geometry;
      // that shape will be duplicated to create the cloud
      var geom = new THREE.BoxGeometry(20, 20, 20);

      // create a material; a simple white material will do the trick
      var mat = new THREE.MeshPhongMaterial({
        color: that.Colors.white,
      });

      // duplicate the geometry a random number of times
      var nBlocs = 3 + Math.floor(Math.random() * 3);
      for (var i = 0; i < nBlocs; i++) {
        // create the mesh by cloning the geometry
        var m = new THREE.Mesh(geom, mat);

        // set the position and the rotation of each cube randomly
        m.position.x = i * 15;
        m.position.y = Math.random() * 10;
        m.position.z = Math.random() * 10;
        m.rotation.z = Math.random() * Math.PI * 2;
        m.rotation.y = Math.random() * Math.PI * 2;

        // set the size of the cube randomly
        var s = 0.1 + Math.random() * 0.9;
        m.scale.set(s, s, s);

        // allow each cube to cast and to receive shadows
        m.castShadow = true;
        m.receiveShadow = true;

        // add the cube to the container we first created
        mesh.add(m);
      }

      return {
        mesh,
        geom,
        mat,
        nBlocs,
      };
    },

    Sky(that) {
      // Create an empty container
      var mesh = new THREE.Object3D();

      // choose a number of clouds to be scattered in the sky
      var nClouds = 20;

      // To distribute the clouds consistently,
      // we need to place them according to a uniform angle
      var stepAngle = (Math.PI * 2) / nClouds;

      // create the clouds
      for (var i = 0; i < nClouds; i++) {
        var c = new that.Cloud(that);

        // set the rotation and the position of each cloud;
        // for that we use a bit of trigonometry
        var a = stepAngle * i; // that is the final angle of the cloud
        var h = 750 + Math.random() * 200; // that is the distance between the center of the axis and the cloud itself

        // Trigonometry!!! I hope you remember what you've learned in Math :)
        // in case you don't:
        // we are simply converting polar coordinates (angle, distance) into Cartesian coordinates (x, y)
        c.mesh.position.y = Math.sin(a) * h;
        c.mesh.position.x = Math.cos(a) * h;

        // rotate the cloud according to its position
        c.mesh.rotation.z = a + Math.PI / 2;

        // for a better result, we position the clouds
        // at random depths inside of the scene
        c.mesh.position.z = -400 - Math.random() * 400;

        // we also set a random scale for each cloud
        var s = 1 + Math.random() * 2;
        c.mesh.scale.set(s, s, s);

        // do not forget to add the mesh of each cloud in the scene
        mesh.add(c.mesh);
      }

      return {
        mesh,
        nClouds,
        stepAngle,
      };
    },

    AirPlane(that) {
      var mesh = new THREE.Object3D();

      // Create the cabin
      var geomCockpit = new THREE.BoxGeometry(60, 50, 50, 1, 1, 1);
      var matCockpit = new THREE.MeshPhongMaterial({
        color: that.Colors.red,
        shading: THREE.FlatShading,
      });
      var cockpit = new THREE.Mesh(geomCockpit, matCockpit);
      cockpit.castShadow = true;
      cockpit.receiveShadow = true;
      mesh.add(cockpit);

      // Create the engine
      var geomEngine = new THREE.BoxGeometry(20, 50, 50, 1, 1, 1);
      var matEngine = new THREE.MeshPhongMaterial({
        color: that.Colors.white,
        shading: THREE.FlatShading,
      });
      var engine = new THREE.Mesh(geomEngine, matEngine);
      engine.position.x = 40;
      engine.castShadow = true;
      engine.receiveShadow = true;
      mesh.add(engine);

      // Create the tail
      var geomTailPlane = new THREE.BoxGeometry(15, 20, 5, 1, 1, 1);
      var matTailPlane = new THREE.MeshPhongMaterial({
        color: that.Colors.red,
        shading: THREE.FlatShading,
      });
      var tailPlane = new THREE.Mesh(geomTailPlane, matTailPlane);
      tailPlane.position.set(-35, 25, 0);
      tailPlane.castShadow = true;
      tailPlane.receiveShadow = true;
      mesh.add(tailPlane);

      // Create the wing
      var geomSideWing = new THREE.BoxGeometry(40, 8, 150, 1, 1, 1);
      var matSideWing = new THREE.MeshPhongMaterial({
        color: that.Colors.red,
        shading: THREE.FlatShading,
      });
      var sideWing = new THREE.Mesh(geomSideWing, matSideWing);
      sideWing.castShadow = true;
      sideWing.receiveShadow = true;
      mesh.add(sideWing);

      // propeller
      var geomPropeller = new THREE.BoxGeometry(20, 10, 10, 1, 1, 1);
      var matPropeller = new THREE.MeshPhongMaterial({
        color: that.Colors.brown,
        shading: THREE.FlatShading,
      });
      var propeller = new THREE.Mesh(geomPropeller, matPropeller);
      propeller.castShadow = true;
      propeller.receiveShadow = true;

      // blades
      var geomBlade = new THREE.BoxGeometry(1, 100, 20, 1, 1, 1);
      var matBlade = new THREE.MeshPhongMaterial({
        color: that.Colors.brownDark,
        shading: THREE.FlatShading,
      });

      var blade = new THREE.Mesh(geomBlade, matBlade);
      blade.position.set(8, 0, 0);
      blade.castShadow = true;
      blade.receiveShadow = true;
      propeller.add(blade);
      propeller.position.set(50, 0, 0);
      mesh.add(propeller);

      return {
        mesh,
        geomCockpit,
        matCockpit,
        cockpit,
        geomEngine,
        matEngine,
        engine,
        geomTailPlane,
        matTailPlane,
        tailPlane,
        geomSideWing,
        matSideWing,
        sideWing,
        geomPropeller,
        matPropeller,
        propeller,
        geomBlade,
        matBlade,
        blade,
      };
    },

    createSea() {
      this.sea = new this.Sea(this);

      // push it a little bit at the bottom of the scene
      this.sea.mesh.position.y = -600;

      // add the mesh of the sea to the scene
      this.scene.add(this.sea.mesh);
    },

    createSky() {
      this.sky = new this.Sky(this);
      this.sky.mesh.position.y = -600;
      this.scene.add(this.sky.mesh);
    },

    createPlane() {
      this.airplane = new this.AirPlane(this);
      console.log(this.airplane);
      this.airplane.mesh.scale.set(0.25, 0.25, 0.25);
      this.airplane.mesh.position.y = 100;
      this.scene.add(this.airplane.mesh);
    },

    loop() {
      this.sea.mesh.rotation.z += 0.005;
      this.sky.mesh.rotation.z += 0.01;

      // update the plane on each frame
      this.updatePlane();

      this.renderer.render(this.scene, this.camera);

      requestAnimationFrame(this.loop);
    },

    init() {
      this.createScene();
      this.createLights();
      this.createPlane();
      this.createSea();
      this.createSky();

      this.loop();
    },

    handleMouseMove(event) {
      // here we are converting the mouse position value received
      // to a normalized value varying between -1 and 1;
      // this is the formula for the horizontal axis:

      var tx = -1 + (event.clientX / this.WIDTH) * 2;

      // for the vertical axis, we need to inverse the formula
      // because the 2D y-axis goes the opposite direction of the 3D y-axis

      var ty = 1 - (event.clientY / this.HEIGHT) * 2;
      this.mousePos = { x: tx, y: ty };
    },

    updatePlane() {
      // let's move the airplane between -100 and 100 on the horizontal axis,
      // and between 25 and 175 on the vertical axis,
      // depending on the mouse position which ranges between -1 and 1 on both axes;
      // to achieve this we use a normalize function (see below)

      var targetX = this.normalize(this.mousePos.x, -1, 1, -100, 100);
      var targetY = this.normalize(this.mousePos.y, -1, 1, 25, 175);

      // update the airplane's position
      this.airplane.mesh.position.y = targetY;
      this.airplane.mesh.position.x = targetX;
      this.airplane.propeller.rotation.x += 0.3;
    },

    normalize(v, vmin, vmax, tmin, tmax) {
      var nv = Math.max(Math.min(v, vmax), vmin);
      var dv = vmax - vmin;
      var pc = (nv - vmin) / dv;
      var dt = tmax - tmin;
      var tv = tmin + pc * dt;
      return tv;
    },
  },
  mounted() {
    // this.initThree();
    this.init();
    //add the listener

    // this.renderer.render(this.scene, this.camera);
  },
};
</script>

<style lang="scss" scoped>
.three {
  position: absolute;
  width: 100%;
  height: 100%;
  overflow: hidden;
  background: linear-gradient(#e4e0ba, #f7d9aa);
}
</style>