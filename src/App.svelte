<script>
  import { differenceInDays, startOfDay } from "date-fns";
  import soundUrl from "./assets/ding.mp3"

  let streak = parseInt(
    localStorage.getItem("streak") || "0"
  );
  const last_session_millis = parseInt(
    localStorage.getItem("last_session_millis") || "0"
  );
  const elapsed_days = differenceInDays(
    startOfDay(new Date()),
    startOfDay(new Date(last_session_millis)),
  );

  if (elapsed_days > 1) {
    streak = 0;
    localStorage.setItem("streak", "0");
  }

  const duration_minutes = streak + 1;
  const duration_millis = duration_minutes * 60_000;

  let start_millis;
  let progress = 0;
  let state = elapsed_days < 1 ? "FINISHED" : "READY";
  let intervalRef;

  function start() {
    new Audio(soundUrl).play();
    state = "IN_PROGRESS";
    start_millis = Date.now();
    intervalRef = setInterval(() => {
      progress = (Date.now() - start_millis) / duration_millis;
      if (progress >= 1) finish();
    }, 100);
  }

  function finish() {
    new Audio(soundUrl).play();
    state = "FINISHED";
    clearInterval(intervalRef);
    localStorage.setItem("streak", (streak + 1).toString());
    localStorage.setItem("last_session_millis", Date.now().toString());
  }

  function cancel() {
    state = "READY";
    clearInterval(intervalRef);
  }
</script>

<main>
  <div class="howdy">
    {#if state === "READY"}
      <h1>
        Today's session will last {duration_minutes}
        minute{duration_minutes > 1 ? "s" : ""}.
      </h1>
      <button on:click={start}>Begin</button>
    {:else if state === "IN_PROGRESS"}
      <progress max="1" value={progress} />
      <button on:click={cancel}>Cancel</button>
    {:else if state === "FINISHED"}
      <h1>Today's session is finished.</h1>
      <p>Come back tomorrow to extend your streak.</p>
    {/if}
  </div>
</main>

<style>
  main {
    display: grid;
    align-content: center;
    height: 100%;
    padding: 3rem;
    font-size: larger;
  }
  .howdy {
    display: grid;
    gap: 1rem;
  }
  button {
    font-size: inherit;
    padding: 0.5rem;
    width: min-content;
  }
</style>
