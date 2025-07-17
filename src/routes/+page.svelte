<script lang="ts">
    
	import CameraFeed from "$lib/components/CameraFeed.svelte";
        
    let videoElement = $state<HTMLVideoElement | undefined>(undefined);
    let photoCanvas: HTMLCanvasElement | undefined = undefined;
    let currentView = $state<'camera' | 'photo'>('camera');
    let isSequenceActive = $state(false);
    let timer = $state<number | null>(null);
    let photos: string[] = $state([]); // stores 3 photos
    let sequenceIndex = $state(0);

    function takePhotoForSequence(){
        if (!videoElement || !photoCanvas) return;

        // set canvas dimensions to match video element's
        photoCanvas.width = videoElement.videoWidth;
        photoCanvas.height = videoElement.videoHeight;
        
        const ctx = photoCanvas.getContext('2d');
        if (!ctx) return;

        // apply mirror effect to the captured image to match the live selfie preview
        ctx.translate(photoCanvas.width, 0); // move origin to top-right
        ctx.scale(-1, 1); // flip horizontally

        // draw the current video frame onto the canvas
        ctx.drawImage(videoElement, 0, 0, photoCanvas.width, photoCanvas.height);
        
        // reset transformation for future draws
        ctx.setTransform(1, 0, 0, 1, 0, 0); 

        // get the image data URL from the canvas
        const dataUrl = photoCanvas.toDataURL('image/jpeg', 0.9); // JPEG for smaller file size

        photos = [...photos, dataUrl];

    }

    async function startPhotoSequence() {
            isSequenceActive = true;
            photos = [];
            sequenceIndex = 0;

            for (let i = 0; i < 3; i++) {
                timer = 3;
                // countdown
                while (timer > 0) {
                    await new Promise(r => setTimeout(r, 1000));
                    timer--;
                }
                await new Promise(r => setTimeout(r, 200));
                takePhotoForSequence();
                sequenceIndex = i + 1;
                await new Promise(r => setTimeout(r, 500));
            }
        timer = null;
        isSequenceActive = false;
        currentView = 'photo';
    }

</script>

<main class="photobooth">
    <h1 class="photobooth__title">Capture</h1>   
    <h2 class="photbooth__subtitle">Vintage Moments, Modern Memories.</h2>
    {#if currentView === 'camera'}
        <div class="photobooth__camera-area">
            <CameraFeed bind:videoElement/>
            {#if isSequenceActive && timer !== null}
                <div class="photobooth__timer-overlay">{timer}</div>
            {/if}
        </div>
        <button 
            class="photobooth__start-btn"
            onclick={startPhotoSequence}
            disabled={isSequenceActive}>
            {isSequenceActive ? 'Taking Photos...' : 'Take a Photo!'}
        </button>  
    {:else if currentView === 'photo'}
        <div class="photobooth__result-area">
            {#if photos.length > 0}
                <div class="photobooth__photo-strip">
                    {#each photos as photo, i}
                        <div class="photobooth__photo-item">
                            <img class="photobooth_photo-img" src={photo} alt="Photo {i+1}" />
                            <a class="photobooth_photo-download" href={photo} download={`photo${i+1}.jpg`}>Download</a>
                        </div>
                    {/each}
                </div>
            {:else}
                <p class="photobooth__no-photo">No photo captured yet.</p>
            {/if}
            <button 
                class="photobooth__retake-btn"
                onclick={() => { currentView = 'camera'; }}>Retake</button>
        </div>
    {/if}
    <p class="photobooth__count">Photos taken: {photos.length}</p>
</main>

<canvas bind:this={photoCanvas} style="display: none;"></canvas>


<style>

    .photobooth { /* ... */ }
    .photobooth__title { /* ... */ }
    .photobooth__camera-area { /* ... */ }
    .photobooth__timer-overlay { /* ... */ }
    .photobooth__photo-strip { /* ... */ }
    .photobooth__photo-item { /* ... */ }
    .photobooth__photo-img { /* ... */ }
    .photobooth__photo-download { /* ... */ }
    .photobooth__start-btn { /* ... */ }
    .photobooth__retake-btn { /* ... */ }
    .photobooth__count { /* ... */ }
    
</style>