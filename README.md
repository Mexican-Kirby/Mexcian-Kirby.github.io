# Mexcian-Kirby.github.io
<!DOCTYPE html>
<html lang="en">
	<head>
		<title>three.js webgl - interactive cubes</title>
		<meta charset="utf-8">
		<meta name="viewport" content="width=device-width, user-scalable=no, minimum-scale=1.0, maximum-scale=1.0">
		<link type="text/css" rel="stylesheet" href="main.css">
		<style>
			body {
				background-color: #f0f0f0;
				color: #444;
			}
			a {
				color: #08f;
			}
		</style>
	</head>
	<body>

		<div id="info">
			<a href="https://threejs.org" target="_blank" rel="noopener">three.js</a> webgl - interactive cubes
		</div>

		<!-- Import maps polyfill -->
		<!-- Remove this when import maps will be widely supported -->
		<script async src="https://unpkg.com/es-module-shims@1.6.3/dist/es-module-shims.js"></script>

		<script type="importmap">
			{
				"imports": {
					"three": "../build/three.module.js",
					"three/addons/": "./jsm/"
				}
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
