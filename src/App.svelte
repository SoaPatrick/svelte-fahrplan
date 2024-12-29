<script>
  import { onMount, onDestroy } from "svelte";

  let timetables = [];
  let error = null;
  let intervalId;
  let limit = 6;
  let isLoading = false;
  let reloadInterval = 30000;

  let stations = [
    { id: "8500160", name: "Basel, IWB" },
    { id: "8500193", name: "Basel, Markthalle" },
    { id: "8589340", name: "Basel, Margarethen" },
    { id: "8589316", name: "Basel, Gewerbeschule"}
  ];

  let selectedStation = stations[0].id;

  async function fetchTimetables() {
    try {
      isLoading = true;
      const response = await fetch(
              `https://transport.opendata.ch/v1/stationboard?id=${selectedStation}&limit=${limit}`
      );

      if (!response.ok) {
        throw new Error(`Error: ${response.status} ${response.statusText}`);
      }

      const data = await response.json();
      stationName = data.station.name;
      timetables = data.stationboard.map((entry) => {
        const departureTime = new Date(entry.stop.departure);
        const delay = entry.stop.delay || 0;

        departureTime.setSeconds(departureTime.getSeconds() + delay);

        const now = new Date();
        const minutesToDeparture = Math.max(
                Math.round((departureTime - now) / 60000),
                0
        );

        const stationOnly = entry.to.startsWith("Basel,")
                ? entry.to.split(",")[1]?.trim()
                : entry.to;

        return {
          ...entry,
          minutesToDeparture,
          stationOnly,
        };
      });
    } catch (err) {
      error = err.message;
    } finally {
      isLoading = false; // Ladeanzeige beenden
    }
  }

  function onStationChange(event) {
    selectedStation = event.target.value;
    fetchTimetables();
  }

  onMount(() => {
    fetchTimetables();
    intervalId = setInterval(fetchTimetables, reloadInterval);
  });

  onDestroy(() => {
    clearInterval(intervalId);
  });
</script>

<main>
  <h1>Fahrplan</h1>
  <div class="heading">
    <select id="stations" bind:value={selectedStation} on:change={onStationChange}>
      {#each stations as station}
        <option value={station.id}>{station.name}</option>
      {/each}
    </select>
  </div>

  {#if error}
    <p style="color: red;">Fehler beim Laden der Daten: {error}</p>
  {:else if isLoading}
    <p>Laden...</p>
  {:else if timetables.length === 0}
    <p>Daten werden geladen...</p>
  {:else}
    <div>
      {#each timetables as timetable}
        <div class="line">
          <div class="number">{timetable.number}</div>
          <div class="destination">{timetable.stationOnly}</div>
          <div class="departure-time">
            {#if timetable.minutesToDeparture === 0}
              <svg class="train-icon" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24"
                   fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <rect width="16" height="16" x="4" y="3" rx="2"/>
                <path d="M4 11h16"/>
                <path d="M12 3v8"/>
                <path d="m8 19-2 3"/>
                <path d="m18 22-2-3"/>
                <path d="M8 15h.01"/>
                <path d="M16 15h.01"/>
              </svg>
            {:else}
              {timetable.minutesToDeparture}'
            {/if}
          </div>
        </div>
      {/each}
    </div>
  {/if}
</main>

<style>
  .line {
    display: flex;
    justify-content: space-between;
    gap: 1rem;
    margin-bottom: 5px;
    color: #F6CC20;
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

  main {
    text-align: left;
    width: 350px;
    font-size: 1.5rem;
  }

  .heading {
    margin-bottom: 4rem;
  }

  select {
    appearance: none;
    border-radius: 3px;
    background: transparent;
    border: 1px solid rgba(255,255,255,.25);
    color: white;
    font-size: 1.2rem;
    padding: 0.5rem;
    width: 100%;
    background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="white" class="size-6"><path stroke-linecap="round" stroke-linejoin="round" d="m19.5 8.25-7.5 7.5-7.5-7.5" /></svg> ');
    background-repeat: no-repeat;
    background-position: calc(100% - .5em) center;
    padding-inline-end: 2rem;
    background-size: .875em;
  }
</style>