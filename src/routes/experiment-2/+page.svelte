
<script>
	import { onMount, onDestroy } from 'svelte';
	import * as THREE from 'three';
	import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
	import earth from '$lib/assets/texture.jpg';
	import starsTexture from '$lib/assets/stars.jpg';

	let container;

	let renderer, scene, camera, controls, globe, stars;

	onMount(() => {
		// SCENE
		scene = new THREE.Scene();

		// CAMERA
		camera = new THREE.PerspectiveCamera(
			75,
			container.clientWidth / container.clientHeight,
			0.1,
			1000
		);
		camera.position.z = 20;

	
		renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
		renderer.setSize(container.clientWidth, container.clientHeight);
		renderer.setPixelRatio(window.devicePixelRatio);
		container.appendChild(renderer.domElement);

		controls = new OrbitControls(camera, renderer.domElement);
		controls.enableDamping = true;
		controls.enableZoom = true;
		controls.enablePan = false;

	
		const light = new THREE.DirectionalLight(0xffffff, 2);
		light.position.set(20, 20, 20);
		scene.add(light);

		scene.add(new THREE.AmbientLight(0xffffff, 0.3));

		
		const loader = new THREE.TextureLoader();

	
		const geometry = new THREE.SphereGeometry(8, 64, 64);
		const material = new THREE.MeshStandardMaterial({
			map: loader.load(earth)
		});

		globe = new THREE.Mesh(geometry, material);
		scene.add(globe);


		const starsGeo = new THREE.SphereGeometry(500, 64, 64);
		const starsMat = new THREE.MeshBasicMaterial({
			map: loader.load(starsTexture),
			side: THREE.BackSide
		});

		stars = new THREE.Mesh(starsGeo, starsMat);
		scene.add(stars);

		
		const animate = () => {
			requestAnimationFrame(animate);
			globe.rotation.y += 0.002;
			controls.update();
			renderer.render(scene, camera);
		};

		animate();

	
		const onResize = () => {
			camera.aspect = container.clientWidth / container.clientHeight;
			camera.updateProjectionMatrix();
			renderer.setSize(container.clientWidth, container.clientHeight);
		};
		window.addEventListener('resize', onResize);

	
		onDestroy(() => {
			window.removeEventListener('resize', onResize);
			geometry.dispose();
			material.dispose();
			starsGeo.dispose();
			starsMat.dispose();
			renderer.dispose();
			controls.dispose();
		});
	});
</script>

<section bind:this={container} class="container"> </section>

<style>
	.container {
		width: 100%;
		height: 100vh;
		overflow: hidden;
	}
</style>