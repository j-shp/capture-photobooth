<script lang="ts">

	import CameraFeed from "$lib/components/CameraFeed.svelte";
    
    let photosTaken = $state(0);
    
    let videoElement = $state<HTMLVideoElement | undefined>(undefined);

    function takePhoto(){
        if (!videoElement) {
            alert('Camera not ready.');
            return;
        }
        photosTaken++;
        alert('Photo taken!');
    }

</script>

<main>
    <h1>Capture</h1>
    
    <h2><i>by j-shp</i></h2>
    <CameraFeed bind:videoElement/>
    <p>Photos taken: {photosTaken}</p>
    <button onclick={takePhoto}>
        <span>Take Photo</span>
    </button>
</main>

<style>
    main {
        font-family: 'Courier New', Courier, monospace;
        text-align: center;
        padding: 20px;
        color: #333;

    }

    h1 {
        color: black;
        font-size: 2.5em;
        margin-bottom: 10px;
    }

    button {
        padding: 10px 20px;
        font-size: 1.1em;
        background-color: #28a745;
        color: white;
        border: none;
        border-radius: 5px;
        cursor: pointer;
        margin-top: 20px;
    }

    button:hover {
        background-color: #218838;
    }
    
</style>