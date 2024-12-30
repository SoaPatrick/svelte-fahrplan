<script>
    import Transit from "./Transit.svelte";
    let { timetable } = $props();
</script>
<div class="line">
    <div class="number">{timetable.number}</div>
    <div class="destination">{timetable.stationOnly}</div>
    <div class="departure-time">
        {#if timetable.minutesToDeparture === 0}
            <div class="train-icon">
                <Transit/>
            </div>
        {:else}
            {timetable.minutesToDeparture}'
        {/if}
    </div>
</div>

<style>
    .line {
        font-family: Handjet, serif;
        display: flex;
        justify-content: space-between;
        gap: 1rem;
        margin-bottom: 5px;
        color: var(--clr-brand);
    }

    .number {
        width: 3ch;
        text-align: right;
    }

    .departure-time {
        text-align: right;
        flex: 1;
    }

    .train-icon {
        animation: blink 1s infinite steps(1, start);
        font-size: 1.5rem;
    }

    @keyframes blink {
        50% {
            opacity: 0;
        }
    }

</style>