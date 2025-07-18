<script lang="ts">
	import CameraFeed from "$lib/components/CameraFeed.svelte";
    
    // reactive values
    let videoElement = $state<HTMLVideoElement | undefined>(undefined); // points to the video element showing the camera feed
    let currentView = $state<'camera' | 'photo'>('camera'); // controls whether in camera mode or photostrip viewing mode
    let isSequenceActive = $state(false); // true while a photo sequence is running to disable the button
    let timer = $state<number | null>(null); // countdown value (3 -> 2 -> 1)
    let photos: string[] = $state([]); // stores 3 photos
    let sequenceIndex = $state(0); // tracks which photo is on (0-2)
    let photostripUrl = $state<string | null>(null);


    // non-reactive values
    let photoCanvas: HTMLCanvasElement | undefined = undefined;
    let photoStripCanvas: HTMLCanvasElement | undefined = undefined;

    function takePhoto() {
        if (!videoElement || !photoCanvas) return;

        photoCanvas.width = videoElement.videoWidth;
        photoCanvas.height = videoElement.videoHeight;

        const ctx = photoCanvas.getContext('2d');
        if (!ctx) return;

        // Apply mirror + vintage filter BEFORE drawing
        ctx.save();
        ctx.setTransform(-1, 0, 0, 1, photoCanvas.width, 0);
        ctx.filter = 'sepia(90%) saturate(60%) hue-rotate(-10deg) brightness(90%) contrast(110%)';
        ctx.drawImage(videoElement, 0, 0, photoCanvas.width, photoCanvas.height);
        ctx.restore();

        const dataUrl = photoCanvas.toDataURL('image/jpeg', 0.9);
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
                takePhoto();
                sequenceIndex++;
                await new Promise(r => setTimeout(r, 500));
            }
        timer = null;
        isSequenceActive = false;

        await createPhotostrip();
        currentView = 'photo';
    }

    // combine the three photos into a photostrip
    async function createPhotostrip() {
        if (photos.length !== 3 || !photoStripCanvas) {
            photostripUrl = null;
            return;
        }

        const images = await Promise.all(
            photos.map(
                (src) =>
                    new Promise<HTMLImageElement>((resolve) => {
                        const img = new Image();
                        img.onload = () => resolve(img);
                        img.src = src;
                    })
            )
        );

        const frameMargin = 40;
        const photoMargin = 20;
        const dateHeight = 60;
        const borderRadius = 20;

        const photoWidth = images[0].width;
        const photoHeight = images[0].height;

        const stripWidth = photoWidth + frameMargin * 2;
        const stripHeight = frameMargin * 2 + photoHeight * 3 + photoMargin * 2 + dateHeight;

        photoStripCanvas.width = stripWidth;
        photoStripCanvas.height = stripHeight;

        const ctx = photoStripCanvas.getContext("2d");
        if (!ctx) return;

        // background
        ctx.fillStyle = "#000"; // matte black background
        ctx.fillRect(0, 0, stripWidth, stripHeight);

        // draw each image centered horizontally
        ctx.filter = 'sepia(90%) saturate(60%) hue-rotate(-10deg) brightness(90%) contrast(110%)';
        images.forEach((img, i) => {
            const x = frameMargin;
            const y = frameMargin + i * (photoHeight + photoMargin);
            ctx.fillStyle = "#000"; // inner photo frame
            ctx.fillRect(x - 8, y - 8, photoWidth + 16, photoHeight + 16); // matte frame
            ctx.drawImage(img, x, y, photoWidth, photoHeight);
        });
        ctx.filter = "none";

        // draw date at the bottom
        const dateStr = new Date().toLocaleDateString(undefined, {
            year: "numeric",
            month: "long",
            day: "numeric"
        });

        ctx.fillStyle = "#fff";
        ctx.font = "italic 24px serif";
        ctx.textAlign = "center";
        ctx.shadowColor = "#000";
        ctx.shadowBlur = 4;
        ctx.fillText(dateStr, stripWidth / 2, stripHeight - (frameMargin / 2) - 30);

        photostripUrl = photoStripCanvas.toDataURL("image/png");
    }
</script>

<main class="photobooth">
    <h1 class="photobooth__title">Capture.</h1> 
    {#if currentView === 'camera'}
        <div class="photobooth__camera-area">
            <CameraFeed bind:videoElement/>
            {#if isSequenceActive && timer !== null}
                {#if timer > 0}
                    <div class="photobooth__timer-overlay">{timer}</div>
                {:else if timer === 0}
                    <div class="photobooth__flash-overlay"></div>
                {/if}
            {/if}
        </div>
        <button 
            class="photobooth__button photobooth__start-btn"
            onclick={startPhotoSequence}
            disabled={isSequenceActive}>
            {isSequenceActive ? 'Taking Photos...' : 'Take a Photo!'}
        </button>
        {#if isSequenceActive}
            <p class="photobooth__count">{photos.length} / 3 photos taken</p>
        {/if}

    {:else if currentView === 'photo'}
        <div class="photobooth__result-area">
            {#if photostripUrl}
                <div class="photobooth__photo-strip">
                    <img class="photobooth__photo-img" src={photostripUrl} alt="Photostrip" />                            
                </div>
            {:else}
                <p class="photobooth__no-photo">No photo captured yet.</p>
            {/if}
            <div class="photobooth__button-row">
                <button 
                class="photobooth__button photobooth__retake-btn"
                onclick={() => {
                    photos = [];
                    photostripUrl = null;
                    currentView = 'camera'}}
            >
                Retake
            </button>
            {#if photostripUrl}
                <button
                    class="photobooth__button photobooth__download-btn"
                    onclick={() => {
                        if (!photostripUrl) return;
                        const link = document.createElement('a');
                        link.href = photostripUrl;
                        link.download = 'photostrip.png';
                        document.body.appendChild(link);
                        link.click();
                        document.body.removeChild(link);
                    }}
                >
                    Download
                </button>
            {/if}
            </div>   
        </div>
    {/if}
</main>

<canvas bind:this={photoCanvas} style="display: none;"></canvas>
<canvas bind:this={photoStripCanvas} style="display: none;"></canvas>


<style>
    @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@700&display=swap');
    .photobooth {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: flex-start;
        max-width: 800px;
        margin: 0 auto;
        padding: 2rem;
        min-height: 100vh;
    
        background: transparent;
        color: #2b2b2b;
    }
    .photobooth__title {
        position: relative;
        font-family: 'Poppins', 'Montserrat', Arial, Helvetica, sans-serif;
        font-size: 4rem;
        font-weight: 700;
        letter-spacing: 2px;
        margin-bottom: 2.5rem;
        color: #4a2c2a;
        text-shadow: 1px 2px 8px #fff8, 0 2px 0 #fff4;
        transition: transform 0.3s ease, color 0.3s ease, text-shadow 0.3s ease;
        cursor: pointer;
    }

    .photobooth__title:hover {
        color: #a62c2a;
        text-shadow: 0 0 10px #fff, 0 0 20px #ffe7d6;
        transform: scale(1.05) rotate(-1deg);
        animation: title-flash 0.3s ease;
    }
    @keyframes title-flash {
        0% {
            opacity: 1;
        }
        50% {
            opacity: 0.8;
            transform: scale(1.07) rotate(1deg);
        }
        100% {
            opacity: 1;
            transform: scale(1.05) rotate(-1deg);
        }
    }
    .photobooth__camera-area {
        position: relative;
        border-radius: 1.2rem;
        overflow: hidden;
    }
    .photobooth__timer-overlay {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 6rem;
        font-weight: bold;
        color: #fff;
        background: rgba(0,0,0,0.3);
        z-index: 2;
        pointer-events: none;
        user-select: none;
        border-radius: 1.2rem;
    }
    .photobooth__flash-overlay {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background: #fff;
        opacity: 0.85;
        z-index: 3;
        pointer-events: none;
        animation: flash-fade 1s linear forwards;
        border-radius: 1.2rem;
    }

    @keyframes flash-fade {
        form { opacity: 0.85; }
        to { opacity: 0; }
    }

    .photobooth__photo-strip {
        width: 100%;
        max-width: 400px;
        padding: 1rem;
        box-sizing: border-box;
        display: flex;
        border-radius: 0.75rem;
        justify-content: center;
    }

    .photobooth__photo-strip {
        transition: transform 0.3s ease, box-shadow 0.3s ease;
    }

    .photobooth__photo-strip:hover {
        transform: scale(1.03);
        box-shadow: 0 12px 30px rgba(0, 0, 0, 0.25);
    }
    .photobooth__photo-img {
        max-width: 100%;
        height: auto;
        display: block;
        margin: 0 auto;
        box-shadow: 0 0 10px rgba(0, 0, 0, 0.3); 
    }

    .photobooth__result-area {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        gap: 2rem;
        padding: 2rem;
        background-color: #fdfaf4;
        border-radius: 1.5rem;
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
        margin-top: 2rem;
        max-width: 90vw;
        width: 100%;
        box-sizing: border-box;
        transition: box-shadow 0.3s ease, transform 0.3s ease;
    }


    .photobooth__button-row {
        display: flex;
        gap: 1rem; 
        justify-content: center;
        margin-top: 1.5rem;
        flex-wrap: wrap;
    }

    .photobooth__no-photo {
        font-size: 1.3rem;
        font-weight: 500;
        color: #888;
        text-align: center;
        padding: 1.5rem;
        border: 2px dashed #ccc;
        border-radius: 1rem;
        max-width: 80%;
    }

    .photobooth__button {
        background: #4a2c2a;
        color: #fff;
        font-family: inherit;
        border: none;
        border-radius: 2rem;
        cursor: pointer;
        box-shadow: 0 2px 8px #0002;
        transition: background 0.2s, transform 0.2s, box-shadow 0.2s;
    }

    .photobooth__button:hover:not(:disabled),
    .photobooth__button:focus-visible:not(:disabled) {
        background: #6d3d36;
        transform: translateY(-2px) scale(1.04);
        box-shadow: 0 4px 16px #0003;
    }

    .photobooth__button:disabled {
        opacity: 0.6;
        cursor: not-allowed;
    }

    .photobooth__start-btn {
        margin-top: 2rem;
        font-size: 1.5rem;
        padding: 0.75rem 2.5rem;
    }

    .photobooth__retake-btn,
    .photobooth__download-btn {
        font-size: 1.2rem;
        padding: 0.6rem 2rem;
    }
    .photobooth__count {
        font-size: 1.2rem;
        font-weight: 600;
        margin: 1rem 0 0.5rem 0;
        color: #4a2c2a;
        letter-spacing: 1px;
    }
</style>