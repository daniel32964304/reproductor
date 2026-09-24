<script>
  const controls = [
    { label: 'Previous', icon: '⏮', variant: 'small' },
    { label: 'Play', icon: '⏸', variant: 'primary' },
    { label: 'Next', icon: '⏭', variant: 'small' }
  ]

  const navItems = [
    { label: 'Home', icon: '⌂', active: true },
    { label: 'Search', icon: '⌕', active: false }
  ]
</script>

<div class="player-shell">
  <header class="statusbar" aria-label="Status bar"></header>

  <main class="player-body">
    <div class="brand-block">
      <div class="logo">lirt</div>
      <div class="subheading">Now Playing</div>
    </div>

    <div class="album-stage" aria-label="Album artwork">
      <div class="art-glow"></div>
      <div class="album-art">
        <div class="art-pattern"></div>
      </div>
    </div>

    <section class="track-info" aria-label="Track information">
      <h1>Echoes of Light</h1>
      <p>Solar Bloom</p>
    </section>

    <div class="progress-block" aria-label="Track progress">
      <div class="time-row">
        <span>2:18</span>
        <span>-2:37</span>
      </div>
      <div class="progress-track">
        <div class="progress-fill"></div>
      </div>
    </div>

    <div class="controls" aria-label="Playback controls">
      {#each controls as control}
        <button
          class:primary={control.variant === 'primary'}
          class:small={control.variant === 'small'}
          type="button"
          aria-label={control.label}
        >
          {control.icon}
        </button>
      {/each}
    </div>
  </main>

  <nav class="bottom-nav" aria-label="Main navigation">
    {#each navItems as item}
      <button class:active={item.active} type="button" aria-label={item.label}>
        {item.icon}
      </button>
    {/each}
  </nav>
</div>
