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
    .camera-feed {
        display: flex;
        justify-content: center;
        align-items: center;
        background: #4a2c2a;
        border-radius: 1.2rem;
        overflow: hidden;
        width: 100%;
        max-width: 480px; 
        aspect-ratio: 4/3;
        margin: 0 auto;
        padding: 16px; 
        box-sizing: border-box;
        
    }

    .camera-feed__video {
        width: 100%;
        height: 100%;
        object-fit: cover;
        filter: sepia(90%) saturate(60%) hue-rotate(-10deg) brightness(90%) contrast(110%);
        border-radius: 0.7rem;
        background: grey;
        display: block;
    }

    .camera-feed__error {
        color: #c00;
        font-weight: bold;
        text-align: center;
        padding: 1rem;
    }
</style>