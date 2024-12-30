<script>
  import { onMount, onDestroy } from "svelte";
  import LocateIcon from "./lib/LocateIcon.svelte";
  import LoadingIcon from "./lib/LoadingIcon.svelte";
  import Timetable from "./lib/Timetable.svelte";

  let timetables = [];
  let stationName = "";
  let error = null;
  let intervalId;
  let limit = 6;
  let isLoadingStations = false;
  let isLoadingTimetables = false;
  let reloadInterval = 60000;
  let geolocationSuccessful = false;

  let stations = [];
  let selectedStation = "";

  let position = { latitude: null, longitude: null };

  async function fetchNearbyStations(latitude, longitude) {
    try {
      isLoadingStations = true;
      error = null;
      const response = await fetch(
              `https://transport.opendata.ch/v1/locations?x=${latitude}&y=${longitude}`
      );

      if (!response.ok) {
        throw new Error(`Error: ${response.status} ${response.statusText}`);
      }

      const data = await response.json();

      if (!data.stations || data.stations.length === 0) {
        throw new Error("Kaini Haltstelle in dr Nechi gfunde.");
      }

      stations = data.stations.map((station) => ({
        id: station.id,
        name: station.name,
      }));

      selectedStation = "";
      timetables = [];
      geolocationSuccessful = true;
    } catch (err) {
      error = err.message;
      geolocationSuccessful = false;
    } finally {
      isLoadingStations = false;
    }
  }

  function getGeolocationAndFetchStations() {
    if (!navigator.geolocation) {
      error = "Geolocations wäärde vo däm Gräät nit understitzt.";
      geolocationSuccessful = false;
      return;
    }

    isLoadingStations = true;
    navigator.geolocation.getCurrentPosition(
            (positionData) => {
              position.latitude = positionData.coords.latitude;
              position.longitude = positionData.coords.longitude;

              fetchNearbyStations(position.latitude, position.longitude);
            },
            (err) => {
              error = `Fäähler bim Abrieffevo vo de Geolocations: ${err.message}`;
              isLoadingStations = false;
              geolocationSuccessful = false;
            }
    );
  }

  async function fetchTimetables() {
    if (!selectedStation) return;

    try {
      isLoadingTimetables = true;
      error = null;
      const response = await fetch(
              `https://transport.opendata.ch/v1/stationboard?id=${selectedStation}&limit=${limit}&transportations[]=tram&transportations[]=bus`
      );

      if (!response.ok) {
        throw new Error(`Error: ${response.status} ${response.statusText}`);
      }

      const data = await response.json();

      if (!data.station) {
        throw new Error("Haltstelle hän nit kenne glaade wäärde.");
      }

      stationName = data.station.name;

      const rawTimetables = data.stationboard.map((entry) => {
        const departureTime = new Date(entry.stop.departure);
        const delay = entry.stop.delay || 0;

        //departureTime.setSeconds(departureTime.getSeconds() + delay);

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
          delay,
        };
      });


      timetables = rawTimetables.reduce((groups, timetable) => {
        const key = timetable.stationOnly;
        if (!groups[key]) {
          groups[key] = [];
        }
        groups[key].push(timetable);
        return groups;
      }, {});

    } catch (err) {
      error = err.message;
    } finally {
      isLoadingTimetables = false;
    }
  }

  function onStationChange(event) {
    selectedStation = event.target.value;
    error = null;
    timetables = [];
    fetchTimetables();
  }

  onMount(() => {
    intervalId = setInterval(() => {
      fetchTimetables();
    }, reloadInterval);
  });

  onDestroy(() => {
    clearInterval(intervalId);
  });
</script>

<main>
  <h1>Faarblaan</h1>
  <div class="select-box">
    <select
            id="stations"
            bind:value={selectedStation}
            on:change={onStationChange}
            disabled={!geolocationSuccessful || isLoadingStations}
    >
      <option value="" disabled selected>Haltstell uuswäälen</option>
      {#each stations as station}
        <option value={station.id}>{station.name}</option>
      {/each}
    </select>
    <button
            on:click={getGeolocationAndFetchStations}
            class="fetch-button"
            disabled={isLoadingStations}
    >
      {#if isLoadingStations}
        <LoadingIcon />
      {:else}
        <LocateIcon />
      {/if}
    </button>
  </div>

  {#if error}
    <p class="error">Fäähler: {error}</p>
  {:else if isLoadingStations}
    <p>Haltstelle wäärde suecht...</p>
  {:else if isLoadingTimetables}
    <p>Haltstelledaate wäärde glaade...</p>
  {:else if !selectedStation}
    <p>Muesch e Haltstelle uuswääle.</p>
  {:else if timetables.length === 0}
    <p>Kaini Informazioone verfiegbaar.</p>
  {:else}
    <div>
      {#each Object.entries(timetables) as [station, entries]}
        <h3>{station}</h3>
        {#each entries as entry}
          <Timetable timetable={entry} />
        {/each}
      {/each}
    </div>
  {/if}
</main>

<style>

  h1 {
    margin-top: 1rem;
    font-size: 3.5rem;
  }

  h3 {
    font-family: Handjet, serif;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    color: rgba(255,255,255,.5);
    margin-bottom: 0.5rem;
    padding-bottom: 0.5rem;
    border-bottom: 1px solid rgba(255,255,255,.25);
  }

  main {
    text-align: left;
    width: 340px;
    font-size: 1.5rem;
    margin-bottom: 2rem;
  }

  .select-box {
    margin-bottom: 2rem;
    display: flex;
    gap: 1rem;
  }

  select {
    appearance: none;
    border-radius: 3px;
    background-color: transparent;
    background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="white" class="size-6"><path stroke-linecap="round" stroke-linejoin="round" d="m19.5 8.25-7.5 7.5-7.5-7.5" /></svg> ');
    background-repeat: no-repeat;
    background-position: calc(100% - .5em) center;
    padding-inline-end: 2rem;
    background-size: .875em;
    font-family: inherit;
    border: 1px solid rgba(255,255,255,.25);
    color: white;
    font-size: 1.2rem;
    padding: 0.5rem;
    width: 100%;
  }

  .fetch-button {
    font-size: 1.2rem;
    padding: 0.5rem;
    background-color: var(--clr-brand);
    color: black;
    border: none;
    border-radius: 3px;
    cursor: pointer;
    aspect-ratio: 1;
    display: flex;
    place-items: center;
    font-family: Inter, system-ui, Avenir, Helvetica, Arial, sans-serif;
  }

  .fetch-button:hover {
    background-color: var(--clr-brand--hover);
  }

  .error {
    color: var(--clr-error);
  }
</style>