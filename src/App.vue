<template>
  <div id="app">
    <div id="control">
      <button @click="addObject('Cube','laozaihao')">add cube</button>
      <button @click="addObject('Sphere','laozaihao1')">add sphere</button>


      <div class="control-btns">
        <button @click="loadModel">
          添加示例几何体
        </button>
        <button @click="loadGltf">
          添加机器人
        </button>
        <button @click="addSphere">
          添加球体
        </button>
        <button @click="addBarriers">
          添加墙壁
        </button>
        <button @click="initEnemys">
          添加敌人
        </button>
        
        <hr>

        <button @click="player = null">
          初始化player
        </button>
        <button @click="initEnemys">
          初始化敌人 
        </button>
        <button @click="initScene"> 
          初始化场景
        </button>
      </div>
    </div>
    <div id="container">
      
    </div>
    <div id="manager-panel">
      PLAYERHP:{{playerHp}}

      <hr>


      SCORE:{{score}}
    </div>
  </div>
</template>

<script>
import * as THREE from 'three'
import {OrbitControls} from 'three/examples/jsm/controls/OrbitControls'

import { MTLLoader, OBJLoader } from 'three-obj-mtl-loader'

import {GLTFLoader} from 'three/examples/jsm/loaders/GLTFLoader'
//import { Object3D } from 'three'

const PI = Math.PI;
const RotationUnit = (PI / 180);
var mixers = null;
var clock = new THREE.Clock();
//var AnimationAction = null;


export default {
  name: 'App',
  data() {
    return {
      camera:null,
      scene:null,
      renderer:null,
      controls:null,

      step:0,
      cameraRotateStep:1,

      apple:null,

      player:null,
      playerAngle:0 * RotationUnit,
      playerHp:5,

      barriers:[],
      enemys:[],
      traps:[],

      objects:[],
      objectsName:[],

      spotLight:null,


      score:0


    }
  },

  methods: {
    init(){
      var container = document.getElementById('container'); 
      
      this.scene = new THREE.Scene();
      this.addLight();
      this.addAxes();

      this.camera = new THREE.PerspectiveCamera(75, window.innerWidth/ window.innerHeight,0.1,2000);
      this.camera.position.x = 0;
      this.camera.position.y = 400;
      this.camera.position.z = -600;
      this.camera.lookAt(new THREE.Vector3(0,0,0));
      //console.log(this.camera)

      this.renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
      this.renderer.setClearColor(0xEEEEEE); //背景色
      this.renderer.setSize(window.innerWidth * 0.75 ,window.innerHeight * 0.75);
      this.renderer.shadowMap.enabled = true; //启用阴影
      this.renderer.toneMapping = THREE.LinearToneMapping;
      this.renderer.toneMappingExposure = 1; // 曝光系数

      container.append(this.renderer.domElement);

      this.render();

      this.controls = new OrbitControls(this.camera,this.renderer.domElement);
      this.controls.addEventListener('change',this.render)

      this.score = 0;//初始化得分
    },

    initScene(){
      window.location.reload();
    },

    addPlayerControl(){
      window.addEventListener('keypress',(event)=>{
        let keyCode = event.keyCode;
        this.checkCrash()//碰撞监测
        if(this.player.position.x < 350 && this.player.position.x > -350 
        && this.player.position.z < 350 && this.player.position.z > -350){//碰撞监测
          switch(keyCode){
            case 119://W 向前
              this.player.position.z += (2 * Math.cos(this.playerAngle));
              this.player.position.x += (2 * Math.sin(this.playerAngle));
              break;
            case 97://A 向左
              this.player.rotation.y += (1 * RotationUnit);
              this.playerAngle = this.player.rotation.y;
              //this.syncCameraAndPlayer();
              break;
            case 100://D 向右
              this.player.rotation.y += (-1 * RotationUnit);
              this.playerAngle = this.player.rotation.y;
              //this.syncCameraAndPlayer();
              break;
            case 115://S 向后
              this.player.position.z += (-2 * Math.cos(this.playerAngle));
              this.player.position.x +=(-2 * Math.sin(this.playerAngle));
              //this.syncCameraAndPlayer();
              break;
            default:
              console.log('我不认识喔');
              break;
          }
        }else{//碰撞边界处理
          if(this.player.position.z >= 350){
            let diff = this.player.position.z - 350;
            this.player.position.z -= (diff + 10);
          }
          
          
          if(this.player.position.z <= -350){
            let diff = -1 * (this.player.position.z + 350);
            this.player.position.z += (diff + 10);
          }

          if(this.player.position.x >= 350){
            let diff = this.player.position.x - 350;
            this.player.position.x -= (diff + 10);
          }
          
          if(this.player.position.x <= -350){
            let diff = -1 * (this.player.position.x + 350);
            this.player.position.x += (diff + 10);
          }
        }
        //console.log(event)

        console.log(this.playerAngle)
      })
    },

    addSphere(){
      let obj = new THREE.Mesh(new THREE.SphereGeometry(40,20,20), new THREE.MeshNormalMaterial());
      // obj.scale.set(8,8,8)
      obj.position.x = -20;
      obj.position.y = 30;
      obj.position.z = 20;

      obj.geometry

      this.scene.add(obj);

      this.player = obj;
      console.log(obj)

      this.addPlayerControlForSphere();
      this.render();
    },

    addEnemy(position){
      let obj = new THREE.Mesh(new THREE.BoxGeometry(40,40,40),new THREE.MeshBasicMaterial({color:0x00000}));

      obj.position.x = position.x;
      obj.position.y = position.y;
      obj.position.z = position.z;

      this.scene.add(obj);
      this.enemys.push(obj);
    },

    resetEnemys(){
      this.enemys = [];
    },

    initEnemys(){
      this.addEnemy({x:200,y:20,z:260})
      this.addEnemy({x:200,y:20,z:0})
      this.addEnemy({x:200,y:20,z:-260})

      console.log(this.enemys)
    },

    destroyed(obj) {//实际上放置到视界之外
      obj.position.y = 5000;
    },

    addPlayerControlForSphere(){
      window.addEventListener('keypress',(event)=>{
        let keyCode = event.keyCode;
        if(this.player.position.x < 350 && this.player.position.x > -350 
        && this.player.position.z < 350 && this.player.position.z > -350){//碰撞监测
          switch(keyCode){
            case 119://W 向前
              this.player.position.z += 2;//位移

              this.player.rotation.x += PI / 6; //旋转
              //this.player.position.x += (2 * Math.sin(this.playerAngle));
              break;
            case 97://A 向左
              // this.player.rotation.y += (1 * RotationUnit);
              // this.playerAngle = this.player.rotation.y;
              //this.syncCameraAndPlayer();
              this.player.position.x += 2;
              this.player.rotation.z +=  PI / 6
              break;
            case 100://D 向右
              // this.player.rotation.y += (-1 * RotationUnit);
              // this.playerAngle = this.player.rotation.y;
              //this.syncCameraAndPlayer();
              this.player.position.x += -2;
              this.player.rotation.z += -1 * (PI / 6);
              break;
            case 115://S 向后
              this.player.position.z += -2;

              this.player.rotation.x +=  - 1 * (PI / 6); //旋转
              //this.player.position.x +=(-2 * Math.sin(this.playerAngle));
              //this.syncCameraAndPlayer();
              break;
            default:
              console.log('我不认识喔');
              break;
          }
        }else{//碰撞处理
          if(this.player.position.z >= 350){
            let diff = this.player.position.z - 350;
            this.player.position.z -= (diff + 10);
          }
          
          
          if(this.player.position.z <= -350){
            let diff = -1 * (this.player.position.z + 350);
            this.player.position.z += (diff + 10);
          }

          if(this.player.position.x >= 350){
            let diff = this.player.position.x - 350;
            this.player.position.x -= (diff + 10);
          }
          
          if(this.player.position.x <= -350){
            let diff = -1 * (this.player.position.x + 350);
            this.player.position.x += (diff + 10);
          }
        }
        //console.log(event)

        //onsole.log(this.playerAngle)
      })
    },

    render(){
      this.renderer.render(this.scene, this.camera);
    },

    addLight(){
      this.spotLight = new THREE.SpotLight(0xffffff,2,200);
      this.spotLight.position.set(200,200,200);
      this.spotLight.castShadow = true;
      this.spotLight.shadow.mapSize.width = 1024;
      this.spotLight.shadow.mapSize.height = 1024;

      this.spotLight.shadow.camera.near = 500;
      this.spotLight.shadow.camera.far = 4000;
      this.spotLight.shadow.camera.fov = 30;
      this.scene.add(this.spotLight);
      
      let pointColor = "#000000";
      let directionalLight = new THREE.DirectionalLight(pointColor);
      directionalLight.position.set(-40, 60, -10);
      directionalLight.castShadow = true;
      directionalLight.shadowCameraNear = 2;
      directionalLight.shadowCameraFar = 200;
      directionalLight.shadowCameraLeft = -50;
      directionalLight.shadowCameraRight = 50;
      directionalLight.shadowCameraTop = 50;
      directionalLight.shadowCameraBottom = -50;

      directionalLight.distance = 0;
      directionalLight.intensity = 0.5;        // 光强度，默认为1
      directionalLight.shadowMapHeight = 1024; // 阴影图尺寸
      directionalLight.shadowMapWidth = 1024;
      this.scene.add(directionalLight);



      let ambientLight = new THREE.AmbientLight(0xbbbbbb);
      this.scene.add(ambientLight);

      const hemiLight = new THREE.HemisphereLight(0xffffff, 0x000000, 1);
      hemiLight.position.set(0, 100, 0);
      this.scene.add(hemiLight);


      // let directionalLight = new THREE.DirectionalLight( 0xffffff, 0.5 );
      // console.log(directionalLight)
      // this.scene.add( directionalLight );
    },

    addAxes(){
        //添加坐标轴
        let axes = new THREE.AxesHelper(50);
        this.scene.add(axes);

        //平面
        let planeGeometry = new THREE.PlaneGeometry(1000,1000)
        let planeMaterial = new THREE.MeshStandardMaterial({color: 0x777999});
        let plane = new THREE.Mesh(planeGeometry, planeMaterial);
        //plane.receiveShadow = true;
        plane.rotation.x = -0.5 * Math.PI;
        this.scene.add(plane);
    },

    addObject(_type = 'Cube',name){
      console.log(name)
      if(_type === 'Cube'){
        let obj = new THREE.Mesh(new THREE.BoxGeometry(10,10,10), new THREE.MeshNormalMaterial({color:0xff0000}))
        obj.position.x = 20;
        obj.position.y = 20;
        obj.position.z = 20;

        this.objects.push(obj);
        this.objectsName.push(name)
        this.scene.add(obj);
      }else if(_type === "Sphere"){
        let obj = new THREE.Mesh(new THREE.SphereGeometry(5,20,20), new THREE.MeshLambertMaterial({color:0x00ff00}));
        obj.position.x = -20;
        obj.position.y = 20;
        obj.position.z = 20;

        this.objects.push(obj);
        this.objectsName.push(name)
        this.scene.add(obj);
      }else{
        console.log("abnf")
      }
    },

    addBarriers(){//添加边界以及陷阱
      let obj1 = new THREE.Mesh(new THREE.BoxGeometry(5,200,800), new THREE.MeshNormalMaterial({color:0xff0000}));
      obj1.position.x = -400;
      obj1.position.y = 0;
      obj1.position.z = 0;

      let obj2 = new THREE.Mesh(new THREE.BoxGeometry(800,200,5), new THREE.MeshNormalMaterial({color:0xff0000}));
      obj2.position.z = -400;
      obj2.position.y = 0;
      obj2.position.x = 0;

      let obj3 = new THREE.Mesh(new THREE.BoxGeometry(5,200,800), new THREE.MeshNormalMaterial({color:0xff0000}));
      obj3.position.x = 400;
      obj3.position.y = 0;
      obj3.position.z = 0;

      let obj4 = new THREE.Mesh(new THREE.BoxGeometry(800,200,5), new THREE.MeshNormalMaterial({color:0xff0000}));
      obj4.position.z = 400;
      obj4.position.y = 0;
      obj4.position.x = 0;


      let objA = new THREE.Mesh(new THREE.BoxGeometry(80,200,80), new THREE.MeshLambertMaterial({color:0xff0000}));
      objA.position.z = 200;
      objA.position.y = 40;
      objA.position.x = 0;


      let objB = new THREE.Mesh(new THREE.BoxGeometry(80,200,80), new THREE.MeshLambertMaterial({color:0xff0000}));
      objB.position.z = 200;
      objB.position.y = 40;
      objB.position.x = -150;

      this.scene.add(objA,objB);
      this.traps.push(objA,objB);

      this.barriers.push(obj1,obj2,obj3,obj4);
      this.barriers.forEach(item => {
        this.scene.add(item);
      });
    },


    animate(){
      //render loop
      requestAnimationFrame(this.animate);

      //this.camera.rotation.z += 0.02;
      //this.camera.lookAt(new THREE.Vector3(0,0,0));
      //this.rotationCamera();

      if(this.objects.length > 0){
        this.objects[0].rotation.x += 0.02;
        this.objects[0].rotation.y += 0.02;

        if(this.objects[1]){
          this.step += 0.03
          this.objects[1].position.y = 20 + 18 * (Math.cos(this.step))
        }
      }

      var delta = clock.getDelta();
      if(mixers !== null){
        mixers.update(delta)
      }


      //this.checkCrash()//碰撞监测


      let target = this.player === null ? new THREE.Vector3(0,0,0):this.player.position;
      this.camera.lookAt(target);

      this.renderer.render(this.scene,this.camera);
      //mixers.update(clock.getDelta())
    },

    loadModel(){
      let mtlloader = new MTLLoader();
      let objloader = new OBJLoader();

      mtlloader.load('/static/model/APPLE/APPLE.mtl', (materials) => {
          materials.preload();
          objloader.setMaterials(materials);
          objloader.load('static/model/APPLE/APPLE.obj', (object) => {
            object.scale.set(1, 1, 1);
            object.position.x = -10;
            object.position.y = 10;
            object.position.z = -10;
            this.apple = object;
            console.log(this.apple)
            this.scene.add(object);
          })
        })
    },

    loadFbx(){

    },

    loadGltf(){
      var gltfLoader = new GLTFLoader();
      gltfLoader.load('/sci-fi_robot/scene.gltf',(gltf) => {
        let mesh = gltf.scene;
        mesh.traverse( function ( child ) {
          
          if ( child.isMesh ) {
            //console.log(child)
            //console.log('a')
            child.material.emissive =  child.material.color;
            child.material.emissiveMap = child.material.map ;
          }
        });


        mesh.scale.set(20,20,20);
        mesh.position.set(0,0,0);

        console.log(gltf.scene)

        this.scene.add(mesh);

        this.player = mesh;

        mixers = new THREE.AnimationMixer(mesh);
        mixers.clipAction(gltf.animations[0]).play();

        this.spotLight.target = mesh;

        console.log(gltf)
        this.addPlayerControl();
        this.render();
      },()=>{},(e)=>{console.log(e)});
      //this.scene.add(gltfLoader);    
    },

    checkCrash(){
      if(this.player === null){
        return;
      }else if(this.enemys.length === 0){
        return;
      }else{
        let {x,z} = this.player.position
        let playerPos = {x,z};
        this.enemysPos.forEach((item,index) => {
          if(Math.abs(item.x - playerPos.x) < 20 && Math.abs(item.z - playerPos.z) < 20){
            this.enemys[index].position.y = 5000;
            this.enemys[index].position.x = 5000;
            this.enemys[index].position.z = 5000;//将小方块移出视界
            this.score += 1; //分数 +1
            console.log(this.score);


            let diffZ = playerPos.z - item.z;
            let diffX = playerPos.x - item.x;
            


            this.player.position.x += (diffX / Math.abs(diffX)) * 30;
            this.player.position.z += (diffZ / Math.abs(diffZ)) * 30;
 
          }
        });

        if(this.traps.length > 0){
          this.trapsPos.forEach((item) => { //监测范围应该是障碍的一半边长
            if(Math.abs(item.x - playerPos.x) < 40 && Math.abs(item.z - playerPos.z) < 40){
              this.playerHp -= 1; //扣减player的血量
              let diffZ = playerPos.z - item.z;
              let diffX = playerPos.x - item.x;
              
              this.player.position.x += (diffX / Math.abs(diffZ)) * 30;
              this.player.position.z += (diffZ / Math.abs(diffZ)) * 30;
            }
          })
        }
      }


      this.judegWinOrFail(); //每次碰撞监测游戏是否胜利
    },

    judegWinOrFail(){
      if(this.playerHp <= 0){
        alert('You waste');
        this.initScene();
      }

      if(this.score === 3){
        alert('You win');
        this.initScene();
      }
    }
  },

  computed:{
    enemysPos(){
      return this.enemys.map(item => {
        return {
          x:item.position.x,
          z:item.position.z
        }
      })
    },

    trapsPos(){
      return this.traps.map(item => {
        return {
          x:item.position.x,
          z:item.position.z
        }
      })
    }
  },

  mounted() {
    this.init();
    this.animate();
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  
  color: #2c3e50;
}

#container{
  margin: 20px 0;
  padding: 0 50px;
  display: inline-block; 
}

#control{
  text-align: center;
  vertical-align: middle;
  width: 100%;
  border: 1px solid orange;
  height: 100px;
}

#manager-panel{
  position: absolute;
  right: 20px;
  margin: 20px 0;
  padding: 0 20px;
  width: 180px;
  line-height: 1;
  border: 1px solid black;
  height: 545px;
  background: rgb(247, 245, 245);
  display: inline-block;
}

ul{
  list-style: none;
  padding: 0;
  width: 100%;
  color:#ffffff;
}

.control-btns{
  position: absolute;
}

</style>
