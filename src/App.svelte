<script>
  import TrackList from './lib/TrackList.svelte';
</script>

<div class="app-shell">
  <TrackList />
</div>

<style>
  .app-shell {
    width: min(100vw, 100%);
    min-height: 100vh;
    display: grid;
    place-items: center;
    padding: 24px 16px;
    background: #050505;
  }
</style>
