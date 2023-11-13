<script>
  // import Header from './Header.svelte';
  import "./styles.css";
  import "../app.css";
</script>

<div class="app">
  <main>
    <slot />
  </main>
</div>

<style lang="postcss">
  .app {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
  }

  main {
    flex: 1;
    display: flex;
    flex-direction: column;
    padding: 0;
    width: 100%;
    margin: 0;
    box-sizing: border-box;
  }
</style>
