<script lang="ts">
	import { T, useTask, useLoader } from '@threlte/core';
	import { interactivity, useSuspense, OrbitControls } from '@threlte/extras';
	import { useGamepad } from '@threlte/extras';
	import { Collider, RigidBody, useRapier, useRigidBody } from '@threlte/rapier';

	import { spring } from 'svelte/motion';
	import { STLLoader } from 'three/examples/jsm/loaders/STLLoader';
	import { throttle, aileron, elevator, rudder } from './stores';
	import { DEG2RAD } from 'three/src/math/MathUtils';
	import Ground from './Ground.svelte';
	import { degToRad } from 'three/src/math/MathUtils.js';

	const gamepad = useGamepad({ index: 0 });

	const loader = useLoader(STLLoader);
	const suspend = useSuspense();

	interactivity();
	const scale = spring(0.1);

	let rotation = 0;
	let rigid: any;

	gamepad.clusterBottom.on('down', () => {
		if ($throttle <= 0.9) {
			throttle.update((v) => v + 0.1);
		}
	});
	gamepad.clusterLeft.on('down', () => {
		if ($throttle >= 0.1) {
			throttle.update((v) => v - 0.1);
		}
	});

	throttle.subscribe((v) => {
		if (rigid) {
			console.log(v);
			// rigid.resetForces(true);
			rigid.addForce({ x: 0.0, y: 0.0, z: $throttle * 400.0 }, true);
			// rigid.addForce({ x: v * 0.001, y: 0, z: 0 }, true);
		}
	});

	gamepad.leftStick.on('change', (event) => {
		aileron.set(event.value.x);
		elevator.set(event.value.y);
	});

	gamepad.rightStick.on('change', (event) => {
		rudder.set(event.value.x);
	});

	function handleKeydown(event) {
		if (event.key === 'ArrowUp') {
			throttle.update((v) => v + 0.1);
		}
		if (event.key === 'ArrowDown') {
			throttle.update((v) => v - 0.1);
		}
	}
</script>

<T.PerspectiveCamera
	makeDefault
	position={[10, 10, 10]}
	on:create={({ ref }) => {
		ref.lookAt(0, 1, 0);
	}}
>
	<OrbitControls />
</T.PerspectiveCamera>

<T.DirectionalLight position={[0, 10, 10]} castShadow />

{#await suspend(loader.load('/200-mig17.stl')) then geometry}
	<T.Group position={[0, 3, 0]} rotation={[-90 * DEG2RAD, 0, 0]}>
		<RigidBody
			on:create={({ ref }) => {
				rigid = ref;
			}}
		>
			<T.Mesh scale={0.15} castShadow>
				<T is={geometry} />
				<T.MeshStandardMaterial color="#FFFFFF" />
			</T.Mesh>
			<Collider shape={'cuboid'} args={[1, 3.5, 1]}></Collider>
		</RigidBody>
	</T.Group>
{/await}

<T.GridHelper args={[100]} />
<Ground />

<svelte:window on:keydown={handleKeydown} />
