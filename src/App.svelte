<script>
  import Dropper from './lib/Dropper.svelte';
  import logoIcon from './assets/logo.svg';
  import bgNoise from './assets/bg-noise.png';

  let colors = $state(['#60A2D9', '#EEE2D6', '#758CAB', '#9B9A9F', '#758CAB', '#9B9A9F']);
</script>

<main class="desktop-layout" data-node-id="3:76">
  <!-- Subtle noise background texture (Rectangle 9, #4:99) -->
  <div class="bg-noise" style="background-image: url('{bgNoise}');" data-node-id="4:99"></div>

  <!-- Header -->
  <header class="header">
    <div class="brand" data-node-id="3:83">
      <div class="logo-wrapper" data-node-id="3:84">
        <img src={logoIcon} alt="PhotoPallete logo" class="logo-icon" data-node-id="3:85" />
      </div>
      <span class="brand-title" data-node-id="3:86">PhotoPallete</span>
    </div>
    <a
      href="https://github.com/akshdzn/PhotoPallete"
      target="_blank"
      rel="noopener noreferrer"
      class="github-link"
      data-node-id="3:87"
    >
      Github
    </a>
  </header>

  <!-- Main Dropper Stage -->
  <section class="main-stage">
    <Dropper bind:colors />
  </section>

  <!-- 4 Blurred Background Color Swatches (blurred-elems, #4:98) -->
  <div class="blurred-elems" data-node-id="4:98">
    <div
      class="blurred-rect rect-1"
      style="background-color: {colors[0]};"
      data-node-id="4:93"
    ></div>
    <div
      class="blurred-rect rect-2"
      style="background-color: {colors[1]};"
      data-node-id="4:94"
    ></div>
    <div
      class="blurred-rect rect-3"
      style="background-color: {colors[2]};"
      data-node-id="4:96"
    ></div>
    <div
      class="blurred-rect rect-4"
      style="background-color: {colors[3]};"
      data-node-id="4:97"
    ></div>
  </div>
</main>

<style>
  .desktop-layout {
    position: relative;
    width: 100%;
    min-height: 100dvh;
    background-color: #F3F3F3;
    overflow-x: hidden;
    overflow-y: auto;
    display: flex;
    flex-direction: column;
    align-items: center;
    box-sizing: border-box;
  }

  /* Tiled background noise pattern */
  .bg-noise {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
    background-repeat: repeat;
    background-size: 40px;
    opacity: 0.35;
    pointer-events: none;
    z-index: 1;
  }

  /* Header */
  .header {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 8px;
    padding-top: max(20px, env(safe-area-inset-top));
    padding-left: 16px;
    padding-right: 16px;
    position: relative;
    z-index: 10;
  }

  .brand {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
  }

  .logo-wrapper {
    width: 24px;
    height: 24px;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
  }

  .logo-icon {
    width: 20px;
    height: 20px;
    display: block;
  }

  .brand-title {
    font-family: 'Sunghyun Sans', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
    font-size: 20px;
    font-weight: 500;
    line-height: normal;
    color: #000000;
    letter-spacing: -0.01em;
  }

  .github-link {
    font-family: 'Sunghyun Sans', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
    font-size: 16px;
    font-weight: 400;
    line-height: normal;
    color: #000000;
    text-decoration: none;
    transition: opacity 150ms ease;
    -webkit-tap-highlight-color: transparent;
  }

  .github-link:hover {
    opacity: 0.65;
  }

  /* Main area centering the Dropper */
  .main-stage {
    flex: 1;
    width: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    position: relative;
    z-index: 10;
    padding: 24px 16px max(64px, env(safe-area-inset-bottom));
    box-sizing: border-box;
  }

  /* 4 Blurred Rectangles at the bottom */
  .blurred-elems {
    position: absolute;
    bottom: 0;
    left: 50%;
    transform: translateX(-50%);
    width: 1027px;
    max-width: 100%;
    height: 96px;
    filter: blur(44.2px);
    pointer-events: none;
    z-index: 2;
  }

  .blurred-rect {
    position: absolute;
    transition: background-color 400ms ease;
  }

  /* Exact geometry matching Figma coordinates */
  .rect-1 {
    left: 0;
    top: 17.63px;
    width: 257px;
    height: 78.37px;
  }

  .rect-2 {
    left: 257px;
    top: 50.94px;
    width: 256px;
    height: 45.06px;
  }

  .rect-3 {
    left: 513px;
    top: 0px;
    width: 257px;
    height: 96px;
  }

  .rect-4 {
    left: 770px;
    top: 21.12px;
    width: 257px;
    height: 74.88px;
  }

  @media (max-width: 1060px) {
    .blurred-elems {
      width: 100%;
      height: 72px;
      filter: blur(36px);
    }
    .rect-1 { width: 25%; left: 0; }
    .rect-2 { width: 25%; left: 25%; }
    .rect-3 { width: 25%; left: 50%; }
    .rect-4 { width: 25%; left: 75%; }
  }

  @media (max-width: 600px) {
    .header {
      padding-top: max(16px, env(safe-area-inset-top));
      gap: 6px;
    }
    .main-stage {
      padding: 16px 12px max(48px, env(safe-area-inset-bottom));
    }
    .blurred-elems {
      height: 48px;
      filter: blur(24px);
    }
  }
</style>
