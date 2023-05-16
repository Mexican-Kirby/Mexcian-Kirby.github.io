# Mexcian-Kirby.github.io
import * as THREE from 'three';

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.1, 1000 );

const renderer = new THREE.WebGLRenderer();
renderer.setSize( window.innerWidth, window.innerHeight );
document.body.appendChild( renderer.domElement );

const geometry = new THREE.BoxGeometry( 1, 1, 1 );
const material = new THREE.MeshBasicMaterial( { color: 0x00ff00 } );
const cube = new THREE.Mesh( geometry, material );
scene.add( cube );

camera.position.z = 5;

function animate() {

	requestAnimationFrame( animate );

	cube.rotation.x += 0.01;
	cube.rotation.y += 0.01;

	renderer.render( scene, camera );

}

animate();
			}
		</script>

		<script type="module">

			import * as THREE from 'three';

			import Stats from 'three/addons/libs/stats.module.js';

			import { TrackballControls } from 'three/addons/controls/TrackballControls.js';
			import * as BufferGeometryUtils from 'three/addons/utils/BufferGeometryUtils.js';

			let container, stats;
			let camera, controls, scene, renderer;
			let pickingTexture, pickingScene;
			let highlightBox;

			const pickingData = [];

			const pointer = new THREE.Vector2();
			const offset = new THREE.Vector3( 10, 10, 10 );
			const clearColor = new THREE.Color();

			init();
			animate();

			function init() {

				container = document.getElementById( 'container' );

				camera = new THREE.PerspectiveCamera( 70, window.innerWidth / window.innerHeight, 1, 10000 );
				camera.position.z = 1000;

				scene = new THREE.Scene();
				scene.background = new THREE.Color( 0xffffff );

				scene.add( new THREE.AmbientLight( 0x555555 ) );

				const light = new THREE.SpotLight( 0xffffff, 1.5 );
				light.position.set( 0, 500, 2000 );
				scene.add( light );

				const defaultMaterial = new THREE.MeshPhongMaterial( {
					color: 0xffffff,
					flatShading: true,
					vertexColors: true,
					shininess: 0
				} );

				// set up the picking texture to use a 32 bit integer so we can write and read integer ids from it
				pickingScene = new THREE.Scene();
				pickingTexture = new THREE.WebGLRenderTarget( 1, 1, {

					type: THREE.IntType,
					format: THREE.RGBAIntegerFormat,
					internalFormat: 'RGBA32I',

				} );
				const pickingMaterial = new THREE.ShaderMaterial( {

					glslVersion: THREE.GLSL3,

					vertexShader: /* glsl */`
						attribute int id;
						flat varying int vid;
						void main() {

							vid = id;
							gl_Position = projectionMatrix * modelViewMatrix * vec4( position, 1.0 );

						}
					`,

					fragmentShader: /* glsl */`
						layout(location = 0) out int out_id;
						flat varying int vid;

						void main() {

							out_id = vid;

						}
					`,

				} );

				function applyId( geometry, id ) {

					const position = geometry.attributes.position;
					const array = new Int16Array( position.count );
					array.fill( id );

					const bufferAttribute = new THREE.Int16BufferAttribute( array, 1, false );
					bufferAttribute.gpuType = THREE.IntType;
					geometry.setAttribute( 'id', bufferAttribute );

				}

				function applyVertexColors( geometry, color ) {

					const position = geometry.attributes.position;
					const colors = [];

					for ( let i = 0; i < position.count; i ++ ) {

						colors.push( color.r, color.g, color.b );

					}

					geometry.setAttribute( 'color', new THREE.Float32BufferAttribute( colors, 3 ) );

				}

				const geometries = [];
				const matrix = new THREE.Matrix4();
				const quaternion = new THREE.Quaternion();
				const color = new THREE.Color();

				for ( let i = 0; i < 5000; i ++ ) {

					const geometry = new THREE.BoxGeometry();

					const position = new THREE.Vector3();
					position.x = Math.random() * 10000 - 5000;
					position.y = Math.random() * 6000 - 3000;
					position.z = Math.random() * 8000 - 4000;

					const rotation = new THREE.Euler();
					rotation.x = Math.random() * 2 * Math.PI;
					rotation.y = Math.random() * 2 * Math.PI;
					rotation.z = Math.random() * 2 * Math.PI;

					const scale = new THREE.Vector3();
					scale.x = Math.random() * 200 + 100;
					scale.y = Math.random() * 200 + 100;
					scale.z = Math.random() * 200 + 100;

					quaternion.setFromEuler( rotation );
					matrix.compose( position, quaternion, scale );

					geometry.applyMatrix4( matrix );

					// give the geometry's vertices a random color to be displayed and an integer
					// identifier as a vertex attribute so boxes can be identified after being merged.
					applyVertexColors( geometry, color.setHex( Math.random() * 0xffffff ) );
					applyId( geometry, i );

					geometries.push( geometry );

					pickingData[ i ] = {

						position: position,
						rotation: rotation,
						scale: scale

					};

				}

				const mergedGeometry = BufferGeometryUtils.mergeGeometries( geometries );
				scene.add( new THREE.Mesh( mergedGeometry, defaultMaterial ) );
				pickingScene.add( new THREE.Mesh( mergedGeometry, pickingMaterial ) );

				highlightBox = new THREE.Mesh(
					new THREE.BoxGeometry(),
					new THREE.MeshLambertMaterial( { color: 0xffff00 } )
				);
				scene.add( highlightBox );

				renderer = new THREE.WebGLRenderer( { antialias: true } );
				renderer.setPixelRatio( window.devicePixelRatio );
				renderer.setSize( window.innerWidth, window.innerHeight );
				container.appendChild( renderer.domElement );

				controls = new TrackballControls( camera, renderer.domElement );
				controls.rotateSpeed = 1.0;
				controls.zoomSpeed = 1.2;
				controls.panSpeed = 0.8;
				controls.noZoom = false;
				controls.noPan = false;
				controls.staticMoving = true;
				controls.dynamicDampingFactor = 0.3;

				stats = new Stats();
				container.appendChild( stats.dom );

				renderer.domElement.addEventListener( 'pointermove', onPointerMove );

			}

			//

			function onPointerMove( e ) {

				pointer.x = e.clientX;
				pointer.y = e.clientY;

			}

			function animate() {

				requestAnimationFrame( animate );

				render();
				stats.update();

			}

			function pick() {

				// render the picking scene off-screen
				// set the view offset to represent just a single pixel under the mouse
				const dpr = window.devicePixelRatio;
				camera.setViewOffset(
					renderer.domElement.width, renderer.domElement.height,
					Math.floor( pointer.x * dpr ), Math.floor( pointer.y * dpr ),
					1, 1
				);

				// render the scene
				renderer.setRenderTarget( pickingTexture );

				// clear the background to - 1 meaning no item was hit
				clearColor.setRGB( - 1, - 1, - 1 );
				renderer.setClearColor( clearColor );
				renderer.render( pickingScene, camera );

				// clear the view offset so rendering returns to normal
				camera.clearViewOffset();

				// create buffer for reading single pixel
				const pixelBuffer = new Int32Array( 4 );

				// read the pixel
				renderer.readRenderTargetPixels( pickingTexture, 0, 0, 1, 1, pixelBuffer );

				const id = pixelBuffer[ 0 ];
				if ( id !== - 1 ) {

					// move our highlightBox so that it surrounds the picked object
					const data = pickingData[ id ];
					highlightBox.position.copy( data.position );
					highlightBox.rotation.copy( data.rotation );
					highlightBox.scale.copy( data.scale ).add( offset );
					highlightBox.visible = true;

				} else {

					highlightBox.visible = false;

				}

			}

			function render() {

				controls.update();

				pick();

				renderer.setRenderTarget( null );
				renderer.render( scene, camera );

			}

		</script>

	</body>
</html>
			}
		</script>

		<script type="module">

			import * as THREE from 'three';

			import Stats from 'three/addons/libs/stats.module.js';

			let container, stats;
			let camera, scene, raycaster, renderer;

			let INTERSECTED;
			let theta = 0;

			const pointer = new THREE.Vector2();
			const radius = 100;

			init();
			animate();

			function init() {

				container = document.createElement( 'div' );
				document.body.appendChild( container );

				camera = new THREE.PerspectiveCamera( 70, window.innerWidth / window.innerHeight, 1, 10000 );

				scene = new THREE.Scene();
				scene.background = new THREE.Color( 0xf0f0f0 );

				const light = new THREE.DirectionalLight( 0xffffff, 1 );
				light.position.set( 1, 1, 1 ).normalize();
				scene.add( light );

				const geometry = new THREE.BoxGeometry( 20, 20, 20 );

				for ( let i = 0; i < 2000; i ++ ) {

					const object = new THREE.Mesh( geometry, new THREE.MeshLambertMaterial( { color: Math.random() * 0xffffff } ) );

					object.position.x = Math.random() * 800 - 400;
					object.position.y = Math.random() * 800 - 400;
					object.position.z = Math.random() * 800 - 400;

					object.rotation.x = Math.random() * 2 * Math.PI;
					object.rotation.y = Math.random() * 2 * Math.PI;
					object.rotation.z = Math.random() * 2 * Math.PI;

					object.scale.x = Math.random() + 0.5;
					object.scale.y = Math.random() + 0.5;
					object.scale.z = Math.random() + 0.5;

					scene.add( object );

				}

				raycaster = new THREE.Raycaster();

				renderer = new THREE.WebGLRenderer();
				renderer.setPixelRatio( window.devicePixelRatio );
				renderer.setSize( window.innerWidth, window.innerHeight );
				container.appendChild( renderer.domElement );

				stats = new Stats();
				container.appendChild( stats.dom );

				document.addEventListener( 'mousemove', onPointerMove );

				//

				window.addEventListener( 'resize', onWindowResize );

			}

			function onWindowResize() {

				camera.aspect = window.innerWidth / window.innerHeight;
				camera.updateProjectionMatrix();

				renderer.setSize( window.innerWidth, window.innerHeight );

			}

			function onPointerMove( event ) {

				pointer.x = ( event.clientX / window.innerWidth ) * 2 - 1;
				pointer.y = - ( event.clientY / window.innerHeight ) * 2 + 1;

			}

			//

			function animate() {

				requestAnimationFrame( animate );

				render();
				stats.update();

			}

			function render() {

				theta += 0.1;

				camera.position.x = radius * Math.sin( THREE.MathUtils.degToRad( theta ) );
				camera.position.y = radius * Math.sin( THREE.MathUtils.degToRad( theta ) );
				camera.position.z = radius * Math.cos( THREE.MathUtils.degToRad( theta ) );
				camera.lookAt( scene.position );

				camera.updateMatrixWorld();

				// find intersections

				raycaster.setFromCamera( pointer, camera );

				const intersects = raycaster.intersectObjects( scene.children, false );

				if ( intersects.length > 0 ) {

					if ( INTERSECTED != intersects[ 0 ].object ) {

						if ( INTERSECTED ) INTERSECTED.material.emissive.setHex( INTERSECTED.currentHex );

						INTERSECTED = intersects[ 0 ].object;
						INTERSECTED.currentHex = INTERSECTED.material.emissive.getHex();
						INTERSECTED.material.emissive.setHex( 0xff0000 );

					}

				} else {

					if ( INTERSECTED ) INTERSECTED.material.emissive.setHex( INTERSECTED.currentHex );

					INTERSECTED = null;

				}

				renderer.render( scene, camera );

			}

		</script>

	</body>
</html>
