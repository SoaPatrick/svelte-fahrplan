<script>
    import TransitIcon from "./TransitIcon.svelte";
    let { timetable } = $props();
</script>
<div class="line">
    <div class="number">Linie : {timetable.number}</div>
    <div class="destination">{timetable.stationOnly}</div>
    <div class="delay">
        {#if timetable.delay !== 0}
            + {timetable.delay} '
        {/if}
    </div>
    <div class="departure-time">
        {#if timetable.minutesToDeparture === 0}
            <div class="train-icon">
                <TransitIcon/>
            </div>
        {:else}
            {timetable.minutesToDeparture} '
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

    .destination {
        display: none;
    }

    .number {
        text-align: right;
    }

    .delay {
        flex: 1;
        text-align: right;
        color: var(--clr-error);
    }

    .departure-time {
        text-align: right;
        width: 4ch;
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