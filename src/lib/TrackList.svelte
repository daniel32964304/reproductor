<script lang="ts">
  import { onMount } from 'svelte';

  type Artist = {
    id?: string;
    name?: string;
    image?: string;
    thumbnail?: string;
  };

  type Album = {
    id?: string;
    title?: string;
    image?: string;
    thumbnail?: string;
  };

  type Track = {
    id: string;
    title: string;
    audio: string;
    artist?: Artist;
    album?: Album;
  };

  type ApiResponse = {
    tracks?: Track[];
  };

  let tracks: Track[] = [];
  let selectedIndex = 0;
  let loading = true;
  let error = '';
  let audio: HTMLAudioElement | undefined;
  let isPlaying = false;
  let currentTime = 0;
  let duration = 0;
  let volume = 0.75;
  let shuffle = false;

  onMount(async () => {
    try {
      const response = await fetch('https://leonardoapi.vercel.app/api/tracks');
      if (!response.ok) {
        throw new Error(`Error HTTP: ${response.status}`);
      }

      const data = (await response.json()) as ApiResponse;
      tracks = Array.isArray(data?.tracks) ? data.tracks : [];
      selectedIndex = 0;
    } catch (err) {
      console.error(err);
      error = 'No se pudieron cargar las pistas.';
    } finally {
      loading = false;
    }
  });

  $: currentTrack = tracks[selectedIndex] ?? tracks[0] ?? null;

  $: if (audio && audio.volume !== volume) {
    audio.volume = volume;
  }

  $: if (audio && currentTrack?.audio && audio.src !== currentTrack.audio) {
    audio.src = currentTrack.audio;
    audio.load();
  }

  function formatTime(value: number): string {
    if (!Number.isFinite(value) || value < 0) return '0:00';
    const minutes = Math.floor(value / 60);
    const seconds = Math.floor(value % 60)
      .toString()
      .padStart(2, '0');
    return `${minutes}:${seconds}`;
  }

  function selectTrack(index: number): void {
    if (index < 0 || index >= tracks.length) return;
    selectedIndex = index;
    currentTime = 0;
    if (audio) {
      audio.currentTime = 0;
      if (isPlaying) {
        audio.play();
      }
    }
  }

  function togglePlay(): void {
    if (!audio || !currentTrack?.audio) return;

    if (audio.paused) {
      audio.play();
      isPlaying = true;
    } else {
      audio.pause();
      isPlaying = false;
    }
  }

  function prevTrack(): void {
    if (!tracks.length) return;

    if (currentTime > 5) {
      if (audio) {
        audio.currentTime = 0;
      }
      currentTime = 0;
      return;
    }

    const nextIndex = shuffle
      ? Math.floor(Math.random() * tracks.length)
      : (selectedIndex - 1 + tracks.length) % tracks.length;
    selectTrack(nextIndex);
  }

  function nextTrack(): void {
    if (!tracks.length) return;

    const nextIndex = shuffle
      ? Math.floor(Math.random() * tracks.length)
      : (selectedIndex + 1) % tracks.length;
    selectTrack(nextIndex);
  }

  function updateProgress(event: Event): void {
    const target = event.target as HTMLInputElement;
    const value = Number(target.value);
    if (audio) {
      audio.currentTime = value;
    }
    currentTime = value;
  }

  function updateVolume(event: Event): void {
    const target = event.target as HTMLInputElement;
    volume = Number(target.value);
    if (audio) {
      audio.volume = volume;
    }
  }

  function onAudioTimeUpdate(): void {
    currentTime = audio?.currentTime || 0;
    duration = audio?.duration || 0;
  }

  function onAudioLoadedMetadata(): void {
    duration = audio?.duration || 0;
  }

  function onAudioEnded(): void {
    nextTrack();
  }
</script>

<section class="track-app">
  {#if loading}
    <div class="state">Cargando canciones...</div>
  {:else if error}
    <div class="state error">{error}</div>
  {:else}
    <header class="app-header">
      <span class="eyebrow">LIRT</span>
      <button type="button" class:active={shuffle} class="shuffle-btn" on:click={() => (shuffle = !shuffle)}>
        Aleatorio
      </button>
    </header>

    <article class="player-card">
      <div class="art-wrapper">
        <img
          src={currentTrack?.album?.image || currentTrack?.artist?.image || 'https://placehold.co/600x600/111/fff?text=Cover'}
          alt={currentTrack?.title || 'Portada'}
        />
      </div>

      <div class="meta">
        <p>Now playing</p>
        <h3>{currentTrack?.title || 'Sin título'}</h3>
        <span>{currentTrack?.artist?.name || 'Artista desconocido'}</span>
      </div>

      <div class="progress-block">
        <div class="time-row">
          <span>{formatTime(currentTime)}</span>
          <span>{formatTime(duration)}</span>
        </div>

        <input
          type="range"
          min="0"
          max={duration || 0}
          step="0.1"
          value={currentTime}
          on:input={updateProgress}
          aria-label="Progreso de la canción"
        />
      </div>

      <div class="controls">
        <button type="button" class="icon-btn" aria-label="Pista anterior" on:click={prevTrack}>⏮</button>
        <button type="button" class="play-btn" aria-label={isPlaying ? 'Pausar' : 'Reproducir'} on:click={togglePlay}>
          {isPlaying ? '⏸' : '▶'}
        </button>
        <button type="button" class="icon-btn" aria-label="Siguiente pista" on:click={nextTrack}>⏭</button>
      </div>

      <div class="volume-block">
        <span>🔊</span>
        <input
          type="range"
          min="0"
          max="1"
          step="0.01"
          value={volume}
          on:input={updateVolume}
          aria-label="Volumen"
        />
        <strong>{Math.round(volume * 100)}%</strong>
      </div>
    </article>

    <audio
      bind:this={audio}
      preload="metadata"
      on:loadedmetadata={onAudioLoadedMetadata}
      on:timeupdate={onAudioTimeUpdate}
      on:play={() => (isPlaying = true)}
      on:pause={() => (isPlaying = false)}
      on:ended={onAudioEnded}
    ></audio>

    <ul class="playlist" aria-label="Lista de canciones">
      {#each tracks as track, index}
        <li class:selected={index === selectedIndex}>
          <button type="button" on:click={() => selectTrack(index)}>
            <img
              src={track.album?.thumbnail || track.artist?.thumbnail || 'https://placehold.co/100x100/111/fff?text=Track'}
              alt={track.title}
            />

            <div class="track-copy">
              <strong>{track.title}</strong>
              <small>{track.artist?.name || 'Artista desconocido'}</small>
            </div>

            <span class="status">
              {index === selectedIndex ? (isPlaying ? 'ON' : 'PAUSA') : 'PLAY'}
            </span>
          </button>
        </li>
      {/each}
    </ul>
  {/if}
</section>

<style>
  :global(body) {
    background: #0b0b0b;
  }

  .track-app {
    width: min(100%, 440px);
    background: linear-gradient(180deg, #101010 0%, #171717 100%);
    border: 1px solid rgba(255, 255, 255, 0.08);
    border-radius: 28px;
    padding: 20px 18px 14px;
    box-shadow: 0 30px 80px rgba(0, 0, 0, 0.55);
  }

  .app-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 18px;
  }

  .eyebrow {
    color: #1ed760;
    font-size: 0.7rem;
    font-weight: 700;
    letter-spacing: 0.14em;
    text-transform: uppercase;
  }

  .shuffle-btn {
    background: rgba(255, 255, 255, 0.04);
    border: 1px solid rgba(255, 255, 255, 0.1);
    border-radius: 999px;
    color: #dfe4df;
    padding: 7px 12px;
    font-size: 0.76rem;
    cursor: pointer;
    transition: all 0.2s ease;
  }

  .shuffle-btn.active {
    background: rgba(30, 215, 96, 0.14);
    border-color: rgba(30, 215, 96, 0.7);
    color: #7ef0a1;
  }

  .player-card {
    display: flex;
    flex-direction: column;
    gap: 14px;
  }

  .art-wrapper {
    position: relative;
    width: 100%;
    border-radius: 26px;
    overflow: hidden;
    box-shadow: 0 18px 44px rgba(0, 0, 0, 0.4);
  }

  .art-wrapper::before {
    content: '';
    position: absolute;
    inset: 0;
    background: linear-gradient(135deg, rgba(30, 215, 96, 0.12), transparent 50%);
    pointer-events: none;
  }

  .art-wrapper img {
    width: 100%;
    aspect-ratio: 1;
    object-fit: cover;
    display: block;
    background: #1c1c1c;
  }

  .meta {
    text-align: center;
  }

  .meta p {
    margin: 0 0 10px;
    color: #9ca39a;
    font-size: 0.72rem;
    letter-spacing: 0.14em;
    text-transform: uppercase;
  }

  .meta h3 {
    margin: 0;
    font-size: clamp(1.8rem, 5vw, 2.5rem);
    line-height: 1.05;
    letter-spacing: -0.06em;
    color: #f4f7f4;
  }

  .meta span {
    display: inline-block;
    margin-top: 8px;
    color: #b8b8b8;
    font-size: 1rem;
  }

  .progress-block {
    display: flex;
    flex-direction: column;
    gap: 8px;
  }

  .time-row {
    display: flex;
    justify-content: space-between;
    color: #b3b7b4;
    font-size: 0.78rem;
  }

  input[type='range'] {
    width: 100%;
    accent-color: #1ed760;
    cursor: pointer;
  }

  .controls {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 18px;
    margin-top: 4px;
  }

  .icon-btn,
  .play-btn {
    border: 0;
    border-radius: 50%;
    background: rgba(255, 255, 255, 0.06);
    color: #edf2ee;
    cursor: pointer;
    transition: transform 0.2s ease, background 0.2s ease;
  }

  .icon-btn:hover,
  .play-btn:hover {
    transform: translateY(-1px);
    background: rgba(255, 255, 255, 0.12);
  }

  .icon-btn {
    width: 38px;
    height: 38px;
    font-size: 1.1rem;
  }

  .play-btn {
    width: 64px;
    height: 64px;
    background: #1ed760;
    color: #0d0d0d;
    font-size: 1.5rem;
    font-weight: 700;
    box-shadow: 0 12px 24px rgba(30, 215, 96, 0.32);
  }

  .volume-block {
    display: grid;
    grid-template-columns: auto 1fr auto;
    gap: 10px;
    align-items: center;
    padding: 8px 0 0;
    color: #dfe7e0;
  }

  .volume-block strong {
    font-size: 0.78rem;
    color: #c7d1c8;
  }

  .playlist {
    list-style: none;
    margin: 18px 0 0;
    padding: 0;
    display: flex;
    flex-direction: column;
    gap: 8px;
  }

  .playlist li {
    border-radius: 16px;
    background: rgba(255, 255, 255, 0.02);
    border: 1px solid transparent;
    transition: border-color 0.2s ease, background 0.2s ease;
  }

  .playlist li.selected {
    border-color: rgba(30, 215, 96, 0.7);
    background: rgba(30, 215, 96, 0.08);
  }

  .playlist button {
    width: 100%;
    display: flex;
    align-items: center;
    gap: 12px;
    padding: 10px 12px;
    background: transparent;
    border: 0;
    color: inherit;
    text-align: left;
    cursor: pointer;
  }

  .playlist img {
    width: 52px;
    height: 52px;
    border-radius: 12px;
    object-fit: cover;
    flex-shrink: 0;
  }

  .track-copy {
    min-width: 0;
    flex: 1;
    overflow: hidden;
  }

  .track-copy strong,
  .track-copy small {
    display: block;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .track-copy strong {
    font-size: 0.96rem;
    color: #f5f7f6;
  }

  .track-copy small {
    margin-top: 4px;
    color: #a5aaa7;
  }

  .status {
    color: #b9c5bb;
    font-size: 0.66rem;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    white-space: nowrap;
  }

  .state {
    min-height: 240px;
    display: grid;
    place-items: center;
    text-align: center;
    color: #edf2ee;
    font-size: 1.05rem;
    padding: 16px;
  }

  .state.error {
    color: #ffb4b4;
  }
</style>
