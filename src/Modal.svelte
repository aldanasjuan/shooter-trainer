<script>
    import { createEventDispatcher } from 'svelte';
    import { fade } from 'svelte/transition';
    export let open = false;
    export let classes = ""
    export let style = ""
    export let showCloseBtn = true;
    export let closeOnBackdrop = false;
    const dispatch = createEventDispatcher();
    function handleClose() {
      if (!closeOnBackdrop) return;
      close();
    }
    function close() {
      open = false;
      dispatch('close');
    }
  </script>

  <style>
      .modal{
          position: fixed;
          top:0;
          left:0;
          display: grid;
          z-index: 50;
          place-items: center;
          height: 100%;
          width:100%;
          background: rgba(0,0,0,0.7);
      }
      .close{
          position: absolute;
          cursor: pointer;
          color:#fff; 
          font-size: 18px; 
          top:18px; 
          right: 20px;
      }
  </style>
  
  {#if open}
    <div
        class="modal"
        transition:fade={{ duration: 200 }}
        on:click={handleClose}
    >
      {#if showCloseBtn}
        <span class="close"
          on:click={close}
        >
            𐌢
        </span>
      {/if}
      <div on:click|stopPropagation class="{classes}" {style}>
        <slot />
      </div>
    </div>
  {/if}
  