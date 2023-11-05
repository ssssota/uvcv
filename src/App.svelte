<script lang="ts">
  import { onMount } from "svelte";

  let devicesPromise: Promise<MediaDeviceInfo[]> = new Promise(() => {});

  let selected = false;
  let videoId: string | undefined;
  let audioId: string | undefined;

  let video: HTMLVideoElement;

  onMount(() => {
    devicesPromise = navigator.mediaDevices.enumerateDevices();
  });

  const onSubmit = () => {
    selected = true;
    navigator.mediaDevices
      .getUserMedia({
        video: {
          width: { ideal: 1920, max: 3840 },
          height: { ideal: 1080, max: 2160 },
          deviceId: videoId,
        },
        audio: {
          deviceId: audioId,
        },
      })
      .then((stream) => {
        video.srcObject = stream;
      });
  };
</script>

{#if !selected}
  <form on:submit|preventDefault={onSubmit}>
    {#await devicesPromise}
      Loading devices...
    {:then devices}
      <label>
        Video:
        <select bind:value={videoId}>
          {#each devices.filter((d) => d.kind === "videoinput") as device}
            <option value={device.deviceId}>{device.label}</option>
          {/each}
        </select>
      </label>
      <label>
        Audio:
        <select bind:value={audioId}>
          {#each devices.filter((d) => d.kind === "audioinput") as device}
            <option value={device.deviceId}>{device.label}</option>
          {/each}
        </select>
      </label>
    {:catch error}
      {error.message}
    {/await}
    <button type="submit">Submit</button>
  </form>
{:else}
  <video bind:this={video} controls autoplay>
    <p>Video stream not available.</p>
    <track kind="captions" />
  </video>
{/if}

<style>
  video {
    width: 100%;
    height: 100%;
    object-fit: contain;
  }
</style>
