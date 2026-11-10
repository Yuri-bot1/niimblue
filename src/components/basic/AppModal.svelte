<script lang="ts">
  import Modal from "bootstrap/js/dist/modal";
  import { onDestroy, onMount, type Snippet } from "svelte";

  interface Props {
    show?: boolean;
    title: string;
    onClose?: () => void;
    onShow?: () => void;
    children: Snippet;
    footer?: Snippet;
    noBody?: boolean;
    noCloseButton?: boolean;
    size?: "sm" | "lg" | "xl";
    inline?: boolean;
    footerClass?: string;
    visible?: boolean;
  }

  let {
    show = $bindable(true),
    title,
    onClose,
    onShow,
    children,
    footer,
    noBody,
    size,
    noCloseButton,
    inline,
    footerClass,
    visible = true,
  }: Props = $props();

  let modalEl = $state<HTMLElement | undefined>();
  let modal: Modal | null = null;

  const handleHide = () => {
    if (document.activeElement instanceof HTMLElement) {
      document.activeElement.blur();
    }
  };

  const handleHidden = () => {
    show = false;
    onClose?.();
  };

  const handleShown = () => {
    onShow?.();
  };

  onMount(() => {
    if (inline || !modalEl) {
      return;
    }

    modal = new Modal(modalEl);

    modalEl.addEventListener("shown.bs.modal", handleShown);
    modalEl.addEventListener("hide.bs.modal", handleHide);
    modalEl.addEventListener("hidden.bs.modal", handleHidden);

    if (show) {
      modal.show();
    }
  });

  $effect(() => {
    if (!modal || inline) return;

    if (show) {
      modal.show();
    } else {
      modal.hide();
    }
  });

  onDestroy(() => {
    if (modalEl) {
      modalEl.removeEventListener("shown.bs.modal", handleShown);
      modalEl.removeEventListener("hide.bs.modal", handleHide);
      modalEl.removeEventListener("hidden.bs.modal", handleHidden);
    }

    modal?.dispose();
    modal = null;
  });

  export function hide() {
    modal?.hide();
  }
</script>

{#if inline}
  <section class="inline-modal card" class:d-none={!visible}>
    <div class="card-header">
      <h2 class="fs-5 mb-0">{title}</h2>
    </div>
    {#if !noBody}
      <div class="card-body">
        {@render children()}
      </div>
    {/if}
    {#if footer}
      <div class="card-footer d-flex flex-wrap gap-2 {footerClass ?? ''}">
        {@render footer()}
      </div>
    {/if}
  </section>
{:else}
  <div
    bind:this={modalEl}
    class="modal fade"
    class:nobody={noBody}
    class:modal-sm={size === "sm"}
    class:modal-lg={size === "lg"}
    class:modal-xl={size === "xl"}
    tabindex="-1"
    aria-hidden="true">
    <div class="modal-dialog modal-dialog-centered">
      <div class="modal-content">
        <div class="modal-header">
          <h1 class="modal-title fs-5">{title}</h1>
          {#if !noCloseButton}
            <button aria-label="Dismiss" type="button" class="btn-close" data-bs-dismiss="modal"></button>
          {/if}
        </div>

        {#if !noBody}
          <div class="modal-body">
            {@render children()}
          </div>
        {/if}

        {#if footer}
          <div class="modal-footer">
            {@render footer()}
          </div>
        {/if}
      </div>
    </div>
  </div>
{/if}

<style>
  .modal.nobody .modal-header {
    border-bottom: none;
  }

  .modal.nobody .modal-footer {
    border-top: none;
  }
</style>
