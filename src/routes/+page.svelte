<script>
  // @ts-nocheck

  import ReloadSvg from "$lib/components/reloadSvg.svelte";
  import SettingsComponent from "$lib/components/settingsComponent.svelte";
  import SettingSvg from "$lib/components/settingSvg.svelte";
  import audioFile from "$lib/alarm.mp3";
  import { onMount } from "svelte";
  const ptiKey = "practiseTimeInterval";
  const spiKey = "shortPauseInterval";
  const lpiKey = "longPauseInterval";
  const riKey = "repetitionKey";
  const Status = {
    PRACTISE_TIME_INTERVAL: 0,
    SHORT_PAUSE_INTERVAL: 1,
    LONG_PAUSE_INTERVAL: 2,
  };
  const Popup = {
    No: 0,
    Settings: 1,
  };

  let alarm = null;
  let practiseTimeInterval = $state(1500);
  let shortPauseInterval = $state(300);
  let longPauseInterval = $state(1500);
  let repetitionInterval = $state(4);

  let clockIsRunning = false;
  // svelte-ignore state_referenced_locally
  let clockValue = $state(practiseTimeInterval);
  let currentRepetition = 0;
  let currentStatus = Status.PRACTISE_TIME_INTERVAL;
  let clockIntervalId = -1;
  // @ts-ignore
  let clockFace = $derived(
    `${formatNumber(Math.floor(clockValue / 60))}:${formatNumber(clockValue % 60)}`,
  );

  let showPopup = $state(Popup.No);

  // @ts-ignore
  function formatNumber(num) {
    if (num === 0) return "00";
    return num < 10 ? "0" + num : num.toString();
  }
  // @ts-ignore
  function clockClick() {
    if (!clockIsRunning) {
      clockIntervalId = setInterval(() => {
        clockValue -= 1;
        if (clockValue == 0) switchStatus();
      }, 1000);
    } else {
      clearInterval(clockIntervalId);
    }
    clockIsRunning = !clockIsRunning;
  }
  function switchStatus() {
    if (!clockIsRunning) return;
    clearInterval(clockIntervalId);
    clockIsRunning = false;
    if (
      currentStatus == Status.PRACTISE_TIME_INTERVAL &&
      currentRepetition < repetitionInterval - 1
    ) {
      currentStatus = Status.SHORT_PAUSE_INTERVAL;
      clockValue = shortPauseInterval;
    } else if (
      currentStatus == Status.PRACTISE_TIME_INTERVAL &&
      currentRepetition == repetitionInterval - 1
    ) {
      currentStatus = Status.LONG_PAUSE_INTERVAL;
      clockValue = longPauseInterval;
    } else if (currentStatus == Status.SHORT_PAUSE_INTERVAL) {
      currentStatus = Status.PRACTISE_TIME_INTERVAL;
      clockValue = practiseTimeInterval;
      currentRepetition += 1;
    } else {
      currentStatus = Status.PRACTISE_TIME_INTERVAL;
      clockValue = practiseTimeInterval;
      currentRepetition = 1;
    }
    alarm.currentTime = 0;
    alarm.play();
    setTimeout(() => {
      alarm.pause();
      alarm.currentTime = 0; // reset so it’s ready next time
    }, 4000);
  }
  function onSettingClick() {
    showPopup = Popup.Settings;
  }
  function onReloadClick() {
    if (
      (currentStatus == Status.PRACTISE_TIME_INTERVAL &&
        clockValue == practiseTimeInterval) ||
      (currentStatus == Status.SHORT_PAUSE_INTERVAL &&
        clockValue == shortPauseInterval) ||
      (currentStatus == Status.LONG_PAUSE_INTERVAL &&
        clockValue == longPauseInterval)
    ) {
      currentRepetition = 0;
      currentStatus = Status.PRACTISE_TIME_INTERVAL;
    }
    clockIsRunning = false;
    clearInterval(clockIntervalId);
    if (currentStatus == Status.PRACTISE_TIME_INTERVAL)
      clockValue = practiseTimeInterval;
    else if (currentStatus == Status.SHORT_PAUSE_INTERVAL)
      clockValue = shortPauseInterval;
    else if (currentStatus == Status.LONG_PAUSE_INTERVAL)
      clockValue = longPauseInterval;
  }
  // @ts-ignore
  function saveSettings(pi, sp, lp, r) {
    practiseTimeInterval = pi;
    shortPauseInterval = sp;
    longPauseInterval = lp;
    repetitionInterval = r;
    localStorage.setItem(ptiKey, practiseTimeInterval.toString());
    localStorage.setItem(spiKey, shortPauseInterval.toString());
    localStorage.setItem(lpiKey, longPauseInterval.toString());
    localStorage.setItem(riKey, repetitionInterval.toString());

    showPopup = Popup.No;
    clockIsRunning = false;
    clearInterval(clockIntervalId);
    if (currentStatus == Status.PRACTISE_TIME_INTERVAL)
      clockValue = practiseTimeInterval;
    else if (currentStatus == Status.SHORT_PAUSE_INTERVAL)
      clockValue = shortPauseInterval;
    else if (currentStatus == Status.LONG_PAUSE_INTERVAL)
      clockValue = longPauseInterval;
  }

  onMount(() => {
    let zwPti = localStorage.getItem(ptiKey);
    let zwSpi = localStorage.getItem(spiKey);
    let zwLpi = localStorage.getItem(lpiKey);
    let zwRi = localStorage.getItem(riKey);
    if (zwPti == null) {
      localStorage.setItem(ptiKey, practiseTimeInterval.toString());
      localStorage.setItem(spiKey, shortPauseInterval.toString());
      localStorage.setItem(lpiKey, longPauseInterval.toString());
      localStorage.setItem(riKey, repetitionInterval.toString());
    } else {
      practiseTimeInterval = Number(zwPti);
      shortPauseInterval = Number(zwSpi);
      longPauseInterval = Number(zwLpi);
      repetitionInterval = Number(zwRi);
    }
    alarm = new Audio(audioFile);
  });
</script>

<main>
  <!-- svelte-ignore a11y_click_events_have_key_events -->
  <!-- svelte-ignore a11y_no_static_element_interactions -->
  <div id="clock" onclick={clockClick}>
    <h1 style="font-size: 5em;user-select: none;">
      {clockFace}
    </h1>
  </div>
  <div id="button-container">
    <!-- svelte-ignore a11y_click_events_have_key_events -->
    <!-- svelte-ignore a11y_no_static_element_interactions -->
    <div onclick={onSettingClick} class="myButton"><SettingSvg /></div>
    <!-- svelte-ignore a11y_click_events_have_key_events -->
    <!-- svelte-ignore a11y_no_static_element_interactions -->
    <div onclick={onReloadClick} class="myButton"><ReloadSvg /></div>
  </div>
  {#if showPopup != Popup.No}
    <div id="popup-panel" style="">
      <SettingsComponent
        callback={saveSettings}
        practiseTime={practiseTimeInterval}
        shortPauseTime={shortPauseInterval}
        longPauseTime={longPauseInterval}
        nowRepetition={repetitionInterval}
      />
    </div>
  {/if}
</main>
<div id="footer" style="">
  <a href="./imprint">Imprint</a>
  <p>&copy; Rene</p>
</div>

<style>
  #footer {
    width: 100%;
    height: 20%;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
  }
  #popup-panel {
    position: fixed;
    z-index: 1;
    width: 30%;
    height: 30%;
  }
  main {
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
  }
  #clock {
    width: 20%;
    height: 10%;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  #clock:hover {
    border: 1px solid var(--accent);
    border-radius: 10px;
    transition: border 0.3s ease;
  }
  #clock > h1 {
    color: var(--ctp-text) !important;
  }
  #button-container {
    width: 20%;
    height: 5%;
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 10%;
    border: 1px solid var(--accent);
    border-radius: 10px;

    opacity: 1;
    animation: fadeout 5s forwards;
  }

  #button-container:hover {
    opacity: 0;
    animation: fadeIn 0.5s forwards;
  }
  @keyframes fadeIn {
    to {
      opacity: 1;
    }
  }
  @keyframes fadeout {
    to {
      opacity: 0;
    }
  }
  @media (max-width: 768px) {
    main {
      padding: 1rem;
    }

    #clock {
      width: 100%;
      max-width: 300px;
      height: auto;
    }

    #clock:hover {
      border: none;
    }

    #button-container {
      width: 100%;
      max-width: 300px;
      height: auto;
      flex-wrap: wrap;
      border-width: 1px;
      animation: none; /* Disable fadeout on mobile */
      opacity: 1;
      border: none;
    }

    #button-container:hover {
      animation: none;
      opacity: 1;
    }

    #popup-panel {
      width: 90%;
      height: auto;
      display: flex;
      justify-content: center;
      align-items: center;
      position: fixed;
      inset: 0; /* shorthand for top: 0; right: 0; bottom: 0; left: 0 */
      margin: auto;
    }

    #footer {
      height: auto;
      padding: 1rem 0;
    }

    h1 {
      font-size: 3em; /* smaller heading for clock */
    }
  }
</style>
