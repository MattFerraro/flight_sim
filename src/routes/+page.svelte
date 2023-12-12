<script lang="ts">
	import { Canvas } from '@threlte/core';
	import Scene from './Scene.svelte';
	import { World, Debug } from '@threlte/rapier';

	import { throttle, aileron, elevator, rudder } from './stores';
</script>

Throttle: {$throttle}
Aileron: {$aileron}
Elevator: {$elevator}
Rudder: {$rudder}

<div style="width: 80vw; height:80vh; border:solid 1px">
	<Canvas>
		<World>
			<Debug />
			<Scene />
			<div slot="fallback">Uh oh, physics engine failed to load!</div>
		</World>
	</Canvas>
</div>
