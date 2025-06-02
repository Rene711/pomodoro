<script>
  let {
    practiseTime = 1500,
    shortPauseTime = 300,
    longPauseTime = 1500,
    nowRepetition = 4,
    callback,
  } = $props();

  let practiseTimeFace = $derived(
    `${formatNumber(Math.floor(practiseTime / 60))}:${formatNumber(practiseTime % 60)}`,
  );
  let shortPauseTimeFace = $derived(
    `${formatNumber(Math.floor(shortPauseTime / 60))}:${formatNumber(shortPauseTime % 60)}`,
  );
  let longPauseTimeFace = $derived(
    `${formatNumber(Math.floor(longPauseTime / 60))}:${formatNumber(shortPauseTime % 60)}`,
  );
  let repetition = $state(nowRepetition);
  // @ts-ignore
  function formatNumber(num) {
    if (num === 0) return "00";
    return num < 10 ? "0" + num : num.toString();
  }
  function save() {
    callback(
      parseTimeString(practiseTimeFace),
      parseTimeString(shortPauseTimeFace),
      parseTimeString(longPauseTimeFace),
      Number(repetition),
    );
  }
  /**
   * @param {string} timeString
   */
  function parseTimeString(timeString) {
    let parts = timeString.split(":");
    let minSeconds = Number(parts[0]) * 60;
    let seconds = Number(parts[1]);
    return minSeconds + seconds;
  }
</script>

<main>
  <div class="settings-card">
    <div class="input-group">
      <label for="practise">Exercise interval:</label>
      <input
        type="text"
        name="practise"
        id="practise"
        bind:value={practiseTimeFace}
      />
    </div>

    <div class="input-group">
      <label for="shortPause">Short break:</label>
      <input
        type="text"
        name="shortPause"
        id="shortPause"
        bind:value={shortPauseTimeFace}
      />
    </div>

    <div class="input-group">
      <label for="longPause">Long break:</label>
      <input
        type="text"
        name="longPause"
        id="longPause"
        bind:value={longPauseTimeFace}
      />
    </div>

    <div class="input-group">
      <label for="repetition">Repetitions:</label>
      <input
        type="text"
        name="repetition"
        id="repetition"
        bind:value={repetition}
      />
    </div>

    <button onclick={save}>Save</button>
  </div>
</main>

<style>
  main {
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .settings-card {
    width: 80%;
    max-width: 400px;
    background-color: var(--ctp-surface1);
    padding: 2rem;
    border-radius: 16px;
    display: flex;
    flex-direction: column;
    gap: 1.2rem;
    box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
  }

  .input-group {
    display: flex;
    flex-direction: column;
    gap: 0.4rem;
  }

  label {
    font-weight: 600;
    font-size: 0.95rem;
    color: var(--ctp-text);
  }

  input {
    padding: 0.6rem;
    border: 1px solid var(--ctp-border);
    border-radius: 8px;
    background-color: var(--ctp-surface2);
    color: var(--ctp-text);
    font-size: 1rem;
    outline: none;
    transition: border-color 0.2s ease;
  }

  input:focus {
    border-color: var(--accent);
  }

  button {
    margin-top: 1rem;
    padding: 0.75rem;
    border: none;
    border-radius: 10px;
    background-color: var(--ctp-overlay0);
    color: var(--ctp-text);
    font-size: 1rem;
    font-weight: 600;
    cursor: pointer;
    transition: background-color 0.2s ease;
  }

  button:hover {
    background-color: var(--ctp-overlay1);
    color: var(--ctp-green);
  }
  @media (max-width: 768px) {
    button {
      background-color: var(--ctp-overlay1);
      color: var(--ctp-green);
    }
  }
</style>
