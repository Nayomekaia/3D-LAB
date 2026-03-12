<script>
import { onMount } from 'svelte';
// Hiermee kun je code pas laten lopen als de component in de browser geladen is
// Zonder dit zou je Three.js code te vroeg uitvoeren en kan het fouten geven

import * as THREE from 'three';

// Hiermee kun je met je muis rondkijken en je object draaien
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';



	let canvas;

	onMount(() => {
		// Scene: De lege 3D wereld waar alles gebeurt: objecten, lichten en camera zitten hier.
		const scene = new THREE.Scene();

		// Camera: Het oogpunt in de wereld. Bepaalt hoe en vanuit welke hoek je alles ziet.
		const camera = new THREE.PerspectiveCamera(
			75,
			window.innerWidth / window.innerHeight,
			0.1,
			1000
		);
		camera.position.z = 40; // Zoomt uit zodat je het object goed kan zien

		// Renderer: Laat de 3D wereld op het scherm zien. Tekent objecten en kleuren.
		const renderer = new THREE.WebGLRenderer({ canvas, antialias: true });
		renderer.setSize(window.innerWidth, window.innerHeight); // Past canvas aan scherm

		// Geometry De vorm van het object (hier een torus knot three.js term voor object)
		const geometry = new THREE.TorusKnotGeometry(5.529, 3.7917, 55, 20, 5, 16);

		// Shader Material: Kleuren en effecten voor het object. Maakt de bloem look met zachte kleuren en randen.
        const material = new THREE.ShaderMaterial({
  // --- Uniforms: kleuren die we gebruiken ---
  uniforms: {
    colorInside: { value: new THREE.Color(0xff4d6d) },  // Binnenkant van de bloem (licht roze)
    colorOutside: { value: new THREE.Color(0xffc0cb) }, // Buitenkant (pastel roze)
    outlineColor: { value: new THREE.Color(0x8b0000) } // Randkleur (diep rood)
  },

  // --- Vertex Shader ---
  // Bepaalt de positie van elk punt van het object en de normale richting
  // Normaal is belangrijk voor hoe licht en schaduwen vallen
  vertexShader: `
    varying vec3 vPos;     // Bewaart positie van elk punt
    varying vec3 vNormal;  // Bewaart normale richting van elk punt
    void main() {
      vPos = position;                                 // Sla positie op
      vNormal = normalize(normalMatrix * normal);     // Sla normale op
      gl_Position = projectionMatrix * modelViewMatrix * vec4(position,1.0); // Zet punt op scherm
    }
  `,

  // Fragment Shader: Bepaalt de kleur van elk stukje oppervlak
  // Hier maken we het bloem effect met kleurverloop, lobes en zachte randen
  fragmentShader: `
    uniform vec3 colorInside;
    uniform vec3 colorOutside;
    uniform vec3 outlineColor;
    varying vec3 vPos;
    varying vec3 vNormal;

    void main() {
      float d = length(vPos.xy); // Afstand van centrum, voor kleurverloop
      float angle = atan(vPos.y, vPos.x); // Hoek voor bloem lobes
      float petal = 0.5 + 0.5 * cos(angle * 6.0 + d * 2.0); // Subtiele bloemvorm

      vec3 baseColor = mix(colorInside, colorOutside, d / 10.0); // Mix binnen en buitenkleur
      baseColor = mix(baseColor, vec3(1.0,0.1,0.3), petal*0.2);  // Voeg accentkleur toe voor lobes

      // Fresnel effect voor zachte randen (licht aan de randen)
      float fresnel = 1.0 - dot(vNormal, normalize(vPos));
      fresnel = smoothstep(0.2, 0.7, fresnel);

      vec3 finalColor = mix(baseColor, outlineColor, fresnel*0.5); // Combineer basis kleur en rand

      gl_FragColor = vec4(finalColor, 1.0); // Eindkleur van het oppervlak
    }
  `
});

// --- Samengevat ---
// Wat dit doet: maakt een speciaal materiaal voor het object met kleuren en effecten
// Waarom belangrijk: zonder dit ziet het object er vlak en saai uit, met deze shader krijgt het de bloem-look met mooie kleuren en zachte randen
// Kort te zeggen: "ShaderMaterial laat me elk stukje van het object kleuren en effecten geven, zodat het lijkt op een echte bloem"

		// Mesh: Combineert de vorm en het materiaal tot een zichtbaar object
		const torusKnot = new THREE.Mesh(geometry, material);
		scene.add(torusKnot); // Zet het object in de scene

		// Lights: Lichten geven diepte en kleur. Keylight voor highlights, filllight voor zachte schaduwen, ambient voor algemene verlichting. zonder dit zie je het object niet
		const keyLight = new THREE.PointLight(0xffffff, 12);
		keyLight.position.set(10, 10, 10);
		scene.add(keyLight);

		const fillLight = new THREE.PointLight(0xff4d6d, 5);
		fillLight.position.set(-10, -5, 5);
		scene.add(fillLight);

		const ambient = new THREE.AmbientLight(0xffffff, 1.2);
		scene.add(ambient);

		// Controls Laat je met de muis rondkijken en object draaien
		const controls = new OrbitControls(camera, renderer.domElement);
		controls.enableDamping = true;

		// Animatie Loop: Laat het object langzaam draaien zodat je de bloem van alle kanten kan zien
		function animate() {
			requestAnimationFrame(animate);

			torusKnot.rotation.y += 0.002;
			torusKnot.rotation.x += 0.001;

			controls.update(); // Update de muis bewegingen
			renderer.render(scene, camera); // Laat alles zien op het scherm
		}
		animate();

		// Resize Handler: Past alles automatisch aan als het scherm groter of kleiner wordt
		function onResize() {
			camera.aspect = window.innerWidth / window.innerHeight;
			camera.updateProjectionMatrix();
			renderer.setSize(window.innerWidth, window.innerHeight);
		}
		window.addEventListener('resize', onResize);

		// Cleanup: Ruimt alles netjes op als je de pagina sluit zodat lading tijd goed blijft als je object niet gebruikt
		return () => {
			window.removeEventListener('resize', onResize);
			geometry.dispose();
			material.dispose();
			renderer.dispose();
			controls.dispose();
		};
	});
</script>

<section>
	<canvas bind:this={canvas}></canvas>
</section>

<style>
	canvas {
		display: block;
		width: 100vw;
		height: 100vh;
	}
	section {
		margin: 0;
		padding: 0;
		overflow: hidden;
	}
</style>
