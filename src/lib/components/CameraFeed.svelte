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

<div class="camera-container">
    {#if cameraError}
        <p class="error-message">{cameraError}</p>
    {:else}
        <video bind:this={videoElement} autoplay playsinline muted></video>
    {/if}
</div>

<style>
  .camera-container {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    max-width: 640px;
    margin: 20px auto;
    background-color: #f0f0f0;
    border-radius: 8px;
    overflow: hidden; /* ensures video stays within bounds */
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
    min-height: 200px; /* to show loading/error message before video loads */
  }
  video {
    width: 100%;
    height: auto;
    display: block; /* remove extra space below video */
    transform: scaleX(-1); /* mirror the video horizontally for a selfie-like experience */
    filter: sepia(80%) saturate(120%) hue-rotate(5deg);
  }
  .error-message {
    color: red;
    font-weight: bold;
    text-align: center;
    padding: 20px;
  }
  p {
    text-align: center;
    color: #555;
  }
</style>