<template>
  <div id="info">
			<!-- <a href="https://threejs.org" target="_blank" rel="noopener">three.js</a> - level of detail -->
  </div>
</template>

<script>
import * as THREE from '../js/three.module.js';

import { GLTFLoader } from '../js/GLTFLoader.js';   

import { FlyControls } from '../js/FlyControls.js';

// import * as THREE from 'three';

// import { Capsule } from './jsm/math/Capsule.js';

export default {
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      scene: new THREE.Scene(),
      ambientlight: null,
      directionalLight: new THREE.DirectionalLight( 0xffffaa, 1.2 ),
      fillLight1: null,
      fillLight2: null,
      camera: null, 
      renderer: null,
      container: null,
      map: null,
      plane: null,
      flyControl: null,
      clock: new THREE.Clock(),
      loader: null,
      loader2: null
    }
  },
  created() {

			// const scene = new THREE.Scene();
			this.scene.background = new THREE.Color( 0x88ccff );


			//	环境光
		  this.ambientlight = new THREE.AmbientLight( 0x6688cc );
			this.scene.add( this.ambientlight );

			//	平行光
			this.fillLight1 = new THREE.DirectionalLight( 0xff9999, 0.5 );
			this.fillLight1.position.set( - 1, 1, 2 );
			this.scene.add( this.fillLight1 );

			this.fillLight2 = new THREE.DirectionalLight( 0x8888ff, 0.2 );
			this.fillLight2.position.set( 0, - 1, 0 );
			this.scene.add( this.fillLight2 );

			// const directionalLight = new THREE.DirectionalLight( 0xffffaa, 1.2 );
			this.directionalLight.position.set( - 5, 25, - 1 );
			this.directionalLight.castShadow = true;
			this.directionalLight.shadow.camera.near = 0.01;
			this.directionalLight.shadow.camera.far = 500;
			this.directionalLight.shadow.camera.right = 30;
			this.directionalLight.shadow.camera.left = - 30;
			this.directionalLight.shadow.camera.top	= 30;
			this.directionalLight.shadow.camera.bottom = - 30;
			this.directionalLight.shadow.mapSize.width = 1024;
			this.directionalLight.shadow.mapSize.height = 1024;
			this.directionalLight.shadow.radius = 4;
			this.directionalLight.shadow.bias = - 0.00006;
			this.scene.add( this.directionalLight );

            // const playerCollider = new Capsule( new THREE.Vector3( 0, 0.35, -4 ), new THREE.Vector3( 0, 1, -8 ), 0.35 );
            // var playerVelocity = new THREE.Vector3();
            //------------------------------------------------------------------------------------
            

//------------------------------------------------------------------------------------

			this.init();
      this.Load();
      // this.$nextTick(() => {
        this.animate()
      // })
			// this.animate();
  },
  methods: {
    Load() {
      let that = this;
      this.loader = new GLTFLoader()//.setPath( './models/gltf/' );
      // console.log(this.loader)
      // loader.load( './models/gltf/collision-world.glb', function( gltf ) {//issum_the_town_on_capital_isle
      this.loader.load( '../../static/gltf/issum_the_town_on_capital_isle/scene.gltf', function( gltf ) {
          that.map = gltf.scene;

          gltf.scene.scale.setScalar( 1 );
          gltf.scene.position.set(30, -10, 60)
          that.scene.add( gltf.scene );
          // worldOctree.fromGraphNode( gltf.scene );

          gltf.scene.traverse( child => {

              if ( child.isMesh ) {

                  child.castShadow = true;
                  child.receiveShadow = true;

                  if ( child.material.map ) {

                      child.material.map.anisotropy = 8;

                  }

              }

          } );

          // animate();

      } );

      this.loader2 = new GLTFLoader().setPath( '../../static/gltf/low_poly_plane/' );
      this.loader2.load('scene.gltf', function(gltf) {
          that.plane = gltf.scene;
          gltf.cameras.push(that.camera)
          console.log(gltf)
          // plane.rotateY(Math.PI);
          // console.log(that.plane)

          gltf.scene.name = 'plane';
          gltf.scene.position.set(0, 0, 0)    // x, x, 前
          // gltf.scene.position.x = gltf.scene.position.z = 0;
          gltf.scene.scale.setScalar( 0.5 );
          that.scene.add( gltf.scene );

          that.flyControl = new FlyControls( that.plane, that.camera, that.renderer.domElement );
          that.flyControl.movementSpeed = -9;
          that.flyControl.rollSpeed = Math.PI / 8;


      })
    },

    init() {
      this.container = document.createElement( 'div' );
      document.body.appendChild( this.container );

      this.camera = new THREE.PerspectiveCamera( 45, window.innerWidth / window.innerHeight, 1, 1500 );
      // camera.position.z = 0;//1000;
      this.camera.position.set(0, 1, -9);
      this.camera.rotation.order = 'YXZ';
      // camera.lookAt(new THREE.Vector3(0,1,0));//相机朝向的位置
      this.camera.rotateY(Math.PI);
              // camera.pointion.x = camera.position.y = 0;
      //	雾
      this.scene.fog = new THREE.Fog( 0x000000, 1, 15000 );

      const pointLight = new THREE.PointLight( 0xff2200 );
      pointLight.position.set( 0, 0, 0 );
      this.scene.add( pointLight );

      const dirLight = new THREE.DirectionalLight( 0xffffff );
      dirLight.position.set( 0, 0, 1 ).normalize();
      this.scene.add( dirLight );

      this.renderer = new THREE.WebGLRenderer( { antialias: true } );
      this.renderer.setPixelRatio( window.devicePixelRatio );
      this.renderer.setSize( window.innerWidth, window.innerHeight );
      this.container.appendChild( this.renderer.domElement );

      window.addEventListener( 'resize', this.onWindowResize );
		},

    updatePlayer( deltaTime ) {
      if(this.plane) {
        this.camera.lookAt(this.plane.position);
        this.camera.position.set(this.plane.position.x, this.plane.position.y, this.plane.position.z - 13)
      }
    },

    onWindowResize() {
      this.camera.aspect = window.innerWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();

      this.renderer.setSize( window.innerWidth, window.innerHeight );
    },

    animate() {
      requestAnimationFrame( this.animate );
      this.render();
    },

    render() {
      // const STEPS_PER_FRAME = 5;
      // const deltaTime = Math.min( 0.05, clock.getDelta() ) / STEPS_PER_FRAME;
      
      if(this.flyControl)
          this.flyControl.update( this.clock.getDelta() );
      // controls.update( clock.getDelta() );
      this.updatePlayer();
			this.renderer.render( this.scene, this.camera );
		}
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

body {
	margin: 0;
	background-color: #000;
	color: #fff;
	font-family: Monospace;
	font-size: 13px;
	line-height: 24px;
	overscroll-behavior: none;
}

a {
	color: #ff0;
	text-decoration: none;
}

a:hover {
	text-decoration: underline;
}

button {
	cursor: pointer;
	text-transform: uppercase;
}

#info {
	position: absolute;
	top: 0px;
	width: 100%;
	padding: 10px;
	box-sizing: border-box;
	text-align: center;
	-moz-user-select: none;
	-webkit-user-select: none;
	-ms-user-select: none;
	user-select: none;
	pointer-events: none;
	z-index: 1; /* TODO Solve this in HTML */
}

a, button, input, select {
	pointer-events: auto;
}

.dg.ac {
	-moz-user-select: none;
	-webkit-user-select: none;
	-ms-user-select: none;
	user-select: none;
	z-index: 2 !important; /* TODO Solve this in HTML */
}

#overlay {
	position: absolute;
	font-size: 16px;
	z-index: 2;
	top: 0;
	left: 0;
	width: 100%;
	height: 100%;
	display: flex;
	align-items: center;
	justify-content: center;
	flex-direction: column;
	background: rgba(0,0,0,0.7);
}

	#overlay button {
		background: transparent;
		border: 0;
		border: 1px solid rgb(255, 255, 255);
		border-radius: 4px;
		color: #ffffff;
		padding: 12px 18px;
		text-transform: uppercase;
		cursor: pointer;
	}

#notSupported {
	width: 50%;
	margin: auto;
	background-color: #f00;
	margin-top: 20px;
	padding: 10px;
}
</style>
