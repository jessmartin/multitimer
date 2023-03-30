<script lang="ts">
  import { onMount } from 'svelte'

  let timerInterval: number | undefined
  let startTime: number | undefined
  let seconds = 0

  const startTimer = () => {
    if (timerInterval) {
      return
    }

    // reset all of the timers
    timers.forEach((timer) => timer.elapsed = false)
    // set the start time to the current time
    startTime = Date.now()
    
    timerInterval = window.setInterval(() => {
      const elapsedTime = Date.now() - startTime
      seconds = elapsedTime / 1000

      timers.forEach((timer)=> {
        if (timer.elapsed) {
          return
        }
        if (seconds > timer.time) {
          const sound = new Audio('/tickSound.mp3')
          sound.play()
          timer.elapsed = true
        }
      })
    }, 10)
  }
  
  const stopTimer = () => {
    if (!timerInterval && !startTime) {
      return
    }

    clearInterval(timerInterval)
    timerInterval = undefined
  }

  onMount(() => {
    return () => {
      if (timerInterval) {
        clearInterval(timerInterval);
      }
    };
  });

  type Timer = {
    time: number,
    elapsed: boolean
  }
  let timers: Timer[] = [{time: 45, elapsed: false}, {time: 105, elapsed: false}, {time: 195, elapsed: false}]

  const addNewTimer = () => {
    const largestTime = timers.reduce((max, timer) => {
      return timer.time > max ? timer.time : max;
    }, 0);

    timers.push({time: largestTime + 10, elapsed: false})
    timers = timers
  }

  const removeTimer = (event) => {
    const index = event.target.getAttribute('data-attribute-index')
    timers.splice(index, 1)
    timers = timers
  } 

  const secondsToTime = (seconds) => {
    const minutes = Math.floor(seconds / 60);
    const remainingSeconds = Math.round((seconds % 60) * 100) / 100;
    const formattedMinutes = minutes < 10 ? `0${minutes}` : `${minutes}`;
    const formattedSeconds = remainingSeconds < 10 ? `0${remainingSeconds}` : `${remainingSeconds}`;
    return `${formattedMinutes}:${formattedSeconds}`;
  };
</script>

<svelte:head>
	<title>Home</title>
	<meta name="description" content="Svelte demo app" />
</svelte:head>

<h1 class="text-3xl font-bold underline">Multi Timer</h1>

<div class="mb-2">
  <div class="text-4xl">{secondsToTime(seconds)}</div>
</div>
<button class="border px-2 bg-slate-300" on:click={startTimer}>Start</button>
<button class="border px-2 bg-slate-300" on:click={stopTimer}>Stop</button>

<h2 class="mt-3">Timers</h2>
<ul>
  {#each timers as timer, index}
    <li>
      <input class="border bg-slate-300 px-2" type="text" bind:value={timer.time} >
      <button class="bg-red-200 px-2" data-attribute-index={index} on:click={removeTimer}>Remove</button>
    </li>
  {/each}
</ul>
<button class="border px-2 bg-slate-300 mt-2" on:click={addNewTimer}>Add Timer</button>
