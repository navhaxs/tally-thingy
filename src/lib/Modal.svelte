<script lang="ts">
    import { onMount, onDestroy, tick } from 'svelte';
    export let open: boolean = false;
    export let onClose: () => void = () => {};

    let modalContent: HTMLDivElement | null = null;

    function trapFocus(e: KeyboardEvent) {
        if (!open || !modalContent) return;

        // Tab key focus trap
        if (e.key === 'Tab') {
            const focusable = Array.from(
                modalContent.querySelectorAll<HTMLElement>(
                    'a, button, textarea, input, select, [tabindex]:not([tabindex="-1"])'
                )
            ).filter(el => !el.hasAttribute('disabled') && !el.getAttribute('aria-hidden'));

            if (focusable.length === 0) return;

            const first = focusable[0];
            const last = focusable[focusable.length - 1];
            const active = document.activeElement;

            if (!e.shiftKey && active === last) {
                e.preventDefault();
                first.focus();
            } else if (e.shiftKey && active === first) {
                e.preventDefault();
                last.focus();
            }
        }

        // Escape key triggers last button
        if (e.key === 'Escape') {
            const buttons = Array.from(
                modalContent.querySelectorAll<HTMLButtonElement>('button')
            ).filter(el => !el.hasAttribute('disabled') && !el.getAttribute('aria-hidden'));
            if (buttons.length > 0) {
                buttons[buttons.length - 1].click();
            } else {
                onClose();
            }
        }
    }

    onMount(() => {
        document.addEventListener('keydown', trapFocus, true);
    });

    onDestroy(() => {
        document.removeEventListener('keydown', trapFocus, true);
    });

    $: if (open) {
        tick().then(() => {
            if (modalContent) {
                // Try to focus the first focusable element, else focus the modal container
                const focusable = Array.from(
                    modalContent.querySelectorAll<HTMLElement>(
                        'a, button, textarea, input, select, [tabindex]:not([tabindex="-1"])'
                    )
                ).filter(el => !el.hasAttribute('disabled') && !el.getAttribute('aria-hidden'));
                if (focusable.length > 0) {
                    focusable[0].focus();
                } else {
                    modalContent.focus();
                }
            }
        });
    }
</script>

{#if open}
    <div class="fixed inset-0 bg-black/40 flex items-center justify-center z-50" on:click={onClose}>
        <div
            class="bg-white rounded shadow-lg p-6 min-w-[300px]"
            bind:this={modalContent}
            tabindex="0"
            on:click|stopPropagation
        >
            <slot />
        </div>
    </div>
{/if}