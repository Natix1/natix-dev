<script lang="ts">
	import { T, useTask } from "@threlte/core";
	import { Grid, interactivity } from "@threlte/extras";
	import { cubicIn, cubicOut } from "svelte/easing";
	import { Spring, Tween } from "svelte/motion";
	import { Object3D } from "three";
	import { DEG2RAD } from "three/src/math/MathUtils.js";
// @ts-expect-error just shut up please
	import Character from "./Character.svelte";

    interactivity();

    let characterPos: [number, number, number] = $state([0, 0, 0]);
    let gridDistance = new Tween(0, {
        duration: 1750,
        easing: cubicOut
    });
    gridDistance.target = 20;

    let characterSpinSpeed = new Tween(0, {
        duration: 3000,
        easing: cubicIn
    });

    let spotlightTarget: Object3D | undefined = $state();
    let characterRotation = $state(0);
    let outlineOpacity = new Spring(0, {
        stiffness: 0.3,
        damping: 1.0
    });
    let characterActions = $state();
    useTask((dt) => {
        characterRotation += characterSpinSpeed.current * dt;
    });

    $effect(() => {
        if (characterActions) {
            // @ts-expect-error just fuck off
            const unsubscribe = characterActions.subscribe((actionsMap) => {
                const waveAction = actionsMap?.Wave;
                if (waveAction) {
                    waveAction.setLoop(2200, 1);
                    waveAction.clampWhenFinished = true;
                    waveAction.play();

                    const durationMs = waveAction.getClip().duration * 1000;
                    const timer = setTimeout(() => {
                        characterSpinSpeed.target = 2;
                    }, durationMs);

                    return () => clearTimeout(timer);
                }
            });
            return unsubscribe;
        }
    });
</script>

<T.Object3D
    bind:ref={spotlightTarget}
    position={[characterPos[0], characterPos[1], characterPos[2]]}
/>

<Character 
    position={characterPos}
    rotation={[90 * DEG2RAD, 180 * DEG2RAD, characterRotation]}
    outlineOpacity={outlineOpacity.current}

    bind:actions={characterActions}
></Character>

<T.PerspectiveCamera
    makeDefault
    position={[characterPos[0], characterPos[1] + 5, characterPos[2] + 10]}
    oncreate={(camera) => {
        camera.lookAt(characterPos[0], characterPos[1] + 2, characterPos[2]);
    }}
/>

<T.SpotLight 
    position={[characterPos[0], characterPos[1] + 10, characterPos[2] + 4]}
    target={spotlightTarget}
    angle={0.5}
    penumbra={1}
    decay={1.5}
    distance={30}
    intensity={100}
    color={0xFFEDDE}
    castShadow
/>

<Grid 
    plane="xz"
    type="grid"
    infiniteGrid={true}
    fadeDistance={gridDistance.current}
    sectionThickness={0}
    sectionSize={5}
    cellColor={0xFFFFFF}
/>