<script>
  import { onMount } from 'svelte'
  /** @type {HTMLAudioElement}*/
  let audioEl,
    audioElSource,
    /** @type {AudioContext}*/
    ctx,
    gainNode,
    /** @type {AnalyserNode}*/
    visualizer,
    /** @type {Uint8Array}*/
    dataArray,
    bufferLength,
    /** @type {HTMLCanvasElement}*/
    canvas,
    /** @type {CanvasRenderingContext2D}*/
    canvasCtx,
    playing = false,
    visualizerType = 'volume',
    windowWidth,
    windowHeight

  onMount(() => {
    ctx = new AudioContext()
    canvasCtx = canvas.getContext('2d')

    audioElSource = ctx.createMediaElementSource(audioEl)
    visualizer = ctx.createAnalyser()
    bufferLength = visualizer.frequencyBinCount
    dataArray = new Uint8Array(bufferLength)

    audioElSource.connect(visualizer).connect(ctx.destination)

    audioEl.onended = () => {
      playing = false
    }
  })

  function draw() {
    requestAnimationFrame(draw)
    if (visualizerType === 'frequency') {
      visualizer.getByteFrequencyData(dataArray)
    } else {
      visualizer.getByteTimeDomainData(dataArray)
    }

    canvasCtx.fillStyle = 'hsl(0, 0%, 5%)'
    canvasCtx.fillRect(0, 0, canvas.width, canvas.height)
    canvasCtx.lineWidth = 3
    canvasCtx.strokeStyle = 'chartreuse'
    canvasCtx.beginPath()

    let sliceWidth = (canvas.width * 1.0) / bufferLength
    let x = 0

    for (let i = 0; i < bufferLength; i++) {
      let v = dataArray[i] / 128.0
      let y = (v * canvas.height) / 2

      if (i === 0) {
        canvasCtx.moveTo(x, canvas.height - y)
      } else {
        canvasCtx.lineTo(x, canvas.height - y)
      }

      x += sliceWidth
    }

    canvasCtx.stroke()
  }

  function play() {
    ctx.resume()
    if (!audioEl) return

    if (audioEl.paused) {
      audioEl.play()
      playing = true
    }

    draw()
  }

  function pause() {
    if (!audioEl) return

    audioEl.pause()
    playing = false
  }
</script>

<svelte:window bind:innerHeight={windowHeight} bind:innerWidth={windowWidth} />

<main>
  <div class="container">
    <canvas height={windowHeight} width={windowWidth} bind:this={canvas} />

    <div id="controls">
      <button
        aria-label="play track"
        class:active={playing}
        on:click={play}
        class="toggle-btn"
      >
        ►
      </button>
      <button
        aria-label="pause track"
        class:active={!playing}
        on:click={pause}
        class="toggle-btn"
      >
        ◻︎
      </button>
      <label>
        Volume
        <input
          value="volume"
          bind:group={visualizerType}
          name="visualizerType"
          type="radio"
        />
      </label>
      <label>
        Frequency
        <input
          value="frequency"
          bind:group={visualizerType}
          name="visualizerType"
          type="radio"
        />
      </label>
    </div>
  </div>
</main>
<audio loop bind:this={audioEl} src="/yo.mp3" />

<style>
  :global(*, *::before, *::after) {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  :global(html, body, #svelte) {
    max-width: 100%;
    height: 100%;
    background-color: hsl(0, 0%, 5%);
    color-scheme: dark;
    accent-color: chartreuse;

    font-family: sans-serif;
  }

  main {
    position: relative;
    display: grid;
    place-content: center;
    height: 100%;
    width: 100%;
  }

  .container {
    width: 100%;

    display: flex;
    flex-direction: column;
    align-items: center;
  }

  canvas {
    background-color: hsl(0, 0%, 5%);
    border-radius: 7px 7px 0 0;
  }

  .toggle-btn {
    position: relative;
    /* background: whitesmoke; */
    border: 1px solid lightgrey;
    cursor: pointer;
    display: flex;
    justify-content: center;
    align-items: center;

    font-size: 1.3rem;
    width: 3rem;
    height: 2rem;
  }

  .toggle-btn.active {
    background-color: hsl(0, 0%, 5%);
  }

  #controls {
    position: absolute;
    top: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 1rem;

    background: hsl(0, 0%, 15%);
    border-radius: 0 0 7px 7px;
    padding: 1rem 2rem;

    max-width: 400px;
  }
</style>
