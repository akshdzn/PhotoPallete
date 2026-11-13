<script>
  import { getPalette } from 'colorthief';
  import dropBgPattern from '../assets/drop-bg-pattern.png';
  import imageIcon from '../assets/image-icon.svg';
  import closeIcon from '../assets/close-icon.svg';
  import downloadIcon from '../assets/download-icon.svg';
  import copyIcon from '../assets/copy-icon.svg';

  let {
    class: className = '',
    colors = $bindable(['#60A2D9', '#EEE2D6', '#758CAB', '#9B9A9F', '#758CAB', '#9B9A9F']),
    ...restProps
  } = $props();

  let imageSrc = $state(null);
  let isDragging = $state(false);
  let fileInput = $state(null);

  let copiedIndex = $state(null);
  let copyTimeout;

  async function copyColor(color, index) {
    if (!color) return;
    try {
      await navigator.clipboard.writeText(color);
      copiedIndex = index;
      clearTimeout(copyTimeout);
      copyTimeout = setTimeout(() => {
        copiedIndex = null;
      }, 1500);
    } catch (err) {
      console.error('Failed to copy hex color to clipboard:', err);
    }
  }

  function getCopyTextColor(hex) {
    if (!hex || !hex.startsWith('#')) return 'rgba(0, 0, 0, 0.5)';
    const clean = hex.replace('#', '');
    const r = parseInt(clean.substring(0, 2), 16) || 0;
    const g = parseInt(clean.substring(2, 4), 16) || 0;
    const b = parseInt(clean.substring(4, 6), 16) || 0;
    const yiq = (r * 299 + g * 587 + b * 114) / 1000;
    return yiq < 128 ? 'rgba(255, 255, 255, 0.7)' : 'rgba(0, 0, 0, 0.5)';
  }

  async function extractPalette(imgElementOrSrc) {
    try {
      let source = imgElementOrSrc;
      if (typeof imgElementOrSrc === 'string') {
        const img = new Image();
        img.crossOrigin = 'anonymous';
        img.src = imgElementOrSrc;
        if (!img.complete) {
          await new Promise((resolve, reject) => {
            img.onload = () => resolve();
            img.onerror = reject;
          });
        }
        source = img;
      }
      const palette = await getPalette(source, { colorCount: 6 });
      if (palette && palette.length > 0) {
        colors = colors.map((fallback, i) => palette[i]?.hex() ?? palette[palette.length - 1]?.hex() ?? fallback);
      }
    } catch (err) {
      console.error('Failed to extract color palette:', err);
    }
  }

  function handleFile(file) {
    if (file && file.type.startsWith('image/')) {
      if (imageSrc && imageSrc.startsWith('blob:')) {
        URL.revokeObjectURL(imageSrc);
      }
      const url = URL.createObjectURL(file);
      imageSrc = url;
      extractPalette(url);
    }
  }

  function handleDragOver(e) {
    e.preventDefault();
    if (e.dataTransfer) {
      e.dataTransfer.dropEffect = 'copy';
    }
    isDragging = true;
  }

  function handleDragEnter(e) {
    e.preventDefault();
    isDragging = true;
  }

  function handleDragLeave(e) {
    e.preventDefault();
    if (e.currentTarget && e.relatedTarget && e.currentTarget.contains(e.relatedTarget)) {
      return;
    }
    isDragging = false;
  }

  function handleDrop(e) {
    e.preventDefault();
    isDragging = false;
    const files = e.dataTransfer?.files;
    if (files && files.length > 0) {
      handleFile(files[0]);
    }
  }

  function handleContainerClick() {
    if (!imageSrc && fileInput) {
      fileInput.click();
    }
  }

  function handleFileInputChange(e) {
    const file = e.target?.files?.[0];
    if (file) {
      handleFile(file);
    }
  }

  function handleRemove(e) {
    e.stopPropagation();
    if (imageSrc && imageSrc.startsWith('blob:')) {
      URL.revokeObjectURL(imageSrc);
    }
    imageSrc = null;
    colors = ['#60A2D9', '#EEE2D6', '#758CAB', '#9B9A9F', '#758CAB', '#9B9A9F'];
    copiedIndex = null;
    clearTimeout(copyTimeout);
    if (fileInput) {
      fileInput.value = '';
    }
  }
</script>

<input
  type="file"
  accept="image/*"
  bind:this={fileInput}
  onchange={handleFileInputChange}
  style="display: none;"
/>

<div
  class="dropper-outer {imageSrc ? 'is-received' : 'is-empty'} {isDragging ? 'is-dragging' : ''} {className}"
  data-node-id={imageSrc ? '3:77' : '3:66'}
  style={imageSrc ? `border-color: ${colors[0]};` : ''}
  ondragover={handleDragOver}
  ondragenter={handleDragEnter}
  ondragleave={handleDragLeave}
  ondrop={handleDrop}
  onclick={handleContainerClick}
  role="region"
  aria-label="Image dropper"
  {...restProps}
>
  {#if !imageSrc}
    <!-- Empty state (data-node-id="3:66") -->
    <div class="dropzone-inner" data-node-id="3:65">
      <img
        src={dropBgPattern}
        alt=""
        class="bg-pattern"
      />
      <div class="content" data-node-id="3:64">
        <div class="icon-wrapper" data-node-id="3:57" data-name="Frame">
          <img src={imageIcon} alt="" class="icon" />
        </div>
        <p class="drop-text" data-node-id="3:60">
          Drop an image to get colors
        </p>
      </div>
    </div>
  {:else}
    <!-- Received state (data-node-id="3:77") -->
    <div class="received-wrapper">
      <!-- Image frame (data-node-id="3:78") -->
      <div class="image-card" data-node-id="3:78">
        <img
          src={imageSrc}
          alt="Preview"
          class="received-image"
          onload={(e) => extractPalette(e.currentTarget)}
        />

        <!-- Floating action buttons (data-node-id="4:111") -->
        <div class="action-bar" data-node-id="4:111">
          <button
            type="button"
            class="action-btn icon-only"
            onclick={handleRemove}
            aria-label="Remove image"
            data-node-id="5:134"
          >
            <div class="btn-icon-wrapper" data-node-id="5:138" data-name="Frame">
              <img src={closeIcon} alt="" class="btn-icon" />
            </div>
          </button>
        </div>
      </div>

      <!-- Color swatches row -->
      <div class="swatches-row">
        {#each colors as color, i}
          <button
            type="button"
            class="swatch {i === 0 ? 'swatch-first' : ''} {i === colors.length - 1 ? 'swatch-last' : ''}"
            style="background-color: {color};"
            onclick={() => copyColor(color, i)}
            aria-label="Copy hex code {color}"
          >
            <p
              class="swatch-copy-text"
              class:is-copied={copiedIndex === i}
              style="color: {getCopyTextColor(color)};"
            >
              {copiedIndex === i ? 'Copied!' : 'Copy'}
            </p>
          </button>
        {/each}
      </div>
    </div>
  {/if}
</div>

<style>
  .dropper-outer {
    display: inline-flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    border-radius: 20px;
    box-sizing: border-box;
    position: relative;
    max-width: 100%;
    transition: all ease-in-out 200ms;
  }

  .dropper-outer.is-empty {
    padding: 4px;
    border: 2px dashed rgba(0, 0, 0, 0.2);
    cursor: pointer;
  }

  .dropper-outer.is-empty:hover,
  .dropper-outer.is-empty.is-dragging {
    padding: 8px;
    border-color: rgba(0, 0, 0, 0.4);
  }

  .dropper-outer.is-received {
    padding: 6.5px;
    border: 3px dashed #60a2d9;
  }

  /* Empty state styles */
  .dropzone-inner {
    width: 407px;
    max-width: calc(100vw - 44px);
    height: 280px;
    max-height: calc((100vw - 44px) * 280 / 407);
    aspect-ratio: 407 / 280;
    position: relative;
    border-radius: 12px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    overflow: hidden;
    box-sizing: border-box;
  }

  .bg-pattern {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
    opacity: 0.1;
    pointer-events: none;
    border-radius: 12px;
  }

  .content {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 4px;
    position: relative;
    z-index: 1;
    width: min(250px, 90%);
    padding: 0 8px;
    box-sizing: border-box;
  }

  .icon-wrapper {
    width: 48px;
    height: 48px;
    position: relative;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
  }

  .icon {
    display: block;
    width: 48px;
    height: 48px;
    max-width: none;
  }

  .drop-text {
    margin: 0;
    font-family: 'Sunghyun Sans', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
    font-size: clamp(16px, 4.5vw, 20px);
    font-weight: 500;
    line-height: 1.3;
    color: #000000;
    opacity: 0.7;
    text-align: center;
    word-break: break-word;
  }

  /* Received state styles */
  .received-wrapper {
    display: flex;
    flex-direction: column;
    position: relative;
    width: 407px;
    max-width: calc(100vw - 44px);
  }

  .image-card {
    width: 100%;
    height: 280px;
    max-height: calc((100vw - 44px) * 280 / 407);
    aspect-ratio: 407 / 280;
    border-radius: 12px;
    position: relative;
    overflow: hidden;
    z-index: 2;
    box-sizing: border-box;
  }

  .received-image {
    width: 100%;
    height: 100%;
    object-fit: cover;
    display: block;
    pointer-events: none;
    border-radius: 12px;
  }

  .action-bar {
    position: absolute;
    bottom: clamp(10px, 3.5vw, 14px);
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    align-items: center;
    justify-content: center;
    gap: clamp(4px, 1.8vw, 8px);
    max-width: calc(100% - 16px);
    z-index: 10;
  }

  .action-btn {
    height: clamp(28px, 7.5vw, 32px);
    padding: 0 clamp(6px, 2vw, 9px);
    border-radius: 10px;
    background: linear-gradient(180deg, rgba(255, 255, 255, 0) 0%, rgba(255, 255, 255, 0.51) 100%);
    backdrop-filter: blur(12.5px);
    -webkit-backdrop-filter: blur(12.5px);
    box-shadow: 0px 0px 0px 1px rgba(0, 0, 0, 0.2);
    border: none;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    overflow: hidden;
    box-sizing: border-box;
    outline: none;
    touch-action: manipulation;
    -webkit-tap-highlight-color: transparent;
  }

  .action-btn.with-text {
    gap: 4px;
  }

  .btn-icon-wrapper {
    width: clamp(16px, 4.5vw, 20px);
    height: clamp(16px, 4.5vw, 20px);
    position: relative;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
  }

  .btn-icon {
    width: 100%;
    height: 100%;
    display: block;
  }

  .btn-text {
    font-family: 'Sunghyun Sans', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
    font-size: clamp(13px, 3.6vw, 16px);
    font-weight: 500;
    line-height: normal;
    color: rgba(0, 0, 0, 0.8);
    white-space: nowrap;
  }

  /* Color swatches */
  .swatches-row {
    display: flex;
    width: 100%;
    height: clamp(52px, 15vw, 67px);
    margin-top: -13px;
    position: relative;
    z-index: 1;
  }

  .swatch {
    flex: 1;
    min-width: 0;
    height: 100%;
    display: flex;
    align-items: flex-end;
    justify-content: center;
    padding: 0 2px clamp(10px, 3.5vw, 17px) 2px;
    margin: 0;
    border: none;
    box-sizing: border-box;
    cursor: pointer;
    outline: none;
    transition: filter 150ms ease;
    touch-action: manipulation;
    -webkit-tap-highlight-color: transparent;
  }

  .swatch:hover,
  .swatch:active {
    filter: brightness(0.97);
  }

  .swatch-first {
    border-bottom-left-radius: 12px;
  }

  .swatch-last {
    border-bottom-right-radius: 12px;
  }

  .swatch-copy-text {
    margin: 0;
    font-family: 'Sunghyun Sans', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
    font-size: clamp(11px, 2.8vw, 14px);
    font-weight: 500;
    line-height: normal;
    text-align: center;
    white-space: nowrap;
    opacity: 0;
    transition: opacity 150ms ease;
    user-select: none;
    pointer-events: none;
  }

  .swatch:hover .swatch-copy-text,
  .swatch:active .swatch-copy-text,
  .swatch-copy-text.is-copied {
    opacity: 1;
  }
</style>
