<script lang="ts">

    import { onMount } from 'svelte';
    import { browser } from '$app/environment';

    let { videoElement = $bindable<HTMLVideoElement | undefined>() } = $props();
    let cameraError=$state<string | null>(null);

    onMount(async () => {
        if (!browser) return;

        try {
            // request access to user's video camera
            const stream = await navigator.mediaDevices.getUserMedia({ video: true });
            
            // if successful, assign stream to video element
            if (videoElement) {
                videoElement.srcObject = stream;
            }
        } catch (err: any) {
            cameraError = 'Could not access camera: ' + (err?.message ?? err);
        }
    });
</script>

<div class="camera-feed">
    {#if cameraError}
        <p class="camera-feed__error">{cameraError}</p>
    {:else}
        <video
            class="camera-feed__video"
            bind:this={videoElement}
            autoplay
            playsinline
            muted></video>
    {/if}
</div>

<style>
    .camera-feed { /* ... */ }
    .camera-feed__video { /* ... */ }
    .camera-feed__error { /* ... */ }
</style>