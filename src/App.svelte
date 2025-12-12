<script lang="ts">
  import Dialog from "./lib/Blocks/Dialog.svelte";

  var coreNumer = "864555";
  var valid = false;
  validate();
  calculateCore();

  function validate() {
    var number = String(coreNumer);

    if (number.length < 4) {
      valid = false;
      return;
    }
    if (/[^0-9]/.test(number)) {
      valid = false;
      return;
    }

    valid = true;
    return valid;
  }
  var running = false;
  async function calculateCore() {
    console.log("calcuating core", coreNumer);
    running = true;

    if (!validate()) {
      console.log("not valid");
      return;
    }

    var number = String(coreNumer);

    // break into groups
    let groupBreakQuanity = coreNumer.length - 4;
    console.log({ groupBreakQuanity });

    /*

you always get 3 cuts, as you must end up with 4 parts
so it makes more sense to take the full number and apply the cuts 
3 cut heads
each step over once the last has finished everything in its range


*/

    var numberGroups = [];

    var splitNumber = number.split("");

    for (const i of range(groupBreakQuanity)) {
      var s = [...splitNumber];
      var breakIndex = 0;
      for (const ix of range(s.length)) {
        console.log("loop", ix, s[ix], s[ix + 1]);
        if (ix == s.length - 1) break;
        var o = [...splitNumber];

        console.log({ groupBreakQuanity });

        var joinGroup = [];
        for (const element of range(groupBreakQuanity + 1)) {
          joinGroup.push();
        }

        o.splice(breakIndex, 2, s[ix] + s[ix + 1]);
        numberGroups.push(o);
        breakIndex++;
      }
    }
    console.log(numberGroups);
  }

  function* range(n) {
    for (let i = 0; i < n; i++) yield i;
  }
</script>

<main>
  <nav class="right">
    <Dialog>
      <div slot="button">?</div>
      <div slot="content">
        <p>
          By the Third Era, Numeric Cores had fallen completely out of favor and
          were no longer being included in proofs by serious arithmeticians.
        </p>
        <p>
          This concept, once considered sacred, was now regarded as an
          "overengineered concept that only served to slow further advancements
          in the field", as Phillip Wilkins once famously put it.
        </p>
        <h2>- STEPS TO FIND A NUMERIC CORE -</h2>
        <ol>
          <li>
            Take any number with four or more digits. Without changing the
            sequence, split that number into four smaller numbers. (ex. 86455
            becomes 8 6 45 5)
          </li>
          <li>
            Next, assign each a different color, resulting in a valid
            mathematical equation that produces the smallest whole number
            possible. The first number should always be assigned teal to begin
            with a positive number. (ex. + 8 - 6 x 45 ÷ 5)
          </li>
          <li>
            If the result is a number with more than three digits, repeat the
            above process. The final number you obtain that is less than four
            digits is considered the "numeric core" of the larger number. (ex.
            18)
          </li>
        </ol>
      </div></Dialog
    >
  </nav>

  <div>
    <h1>BLue Prince Core Calc</h1>
    <div>
      <input
        type="string"
        placeholder="Enter core number"
        bind:value={coreNumer}
        on:keyup={validate}
      />
    </div>
    <button disabled={!valid} on:click={calculateCore}>Start</button>
    {#if running}
      <div>Calculating core...</div>
      <div></div>
    {/if}
  </div>
</main>

<style>
  .box {
    width: 100%;
    height: 300px; /* your size */
    border: 1px solid #ddd;
    display: flex;
    align-items: center;
    justify-content: center;
    position: relative;
    z-index: 1;
  }

  .spinner-wrap {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .spinner {
    --size: 40px;
    width: var(--size);
    height: var(--size);
    border-radius: 50%;
    border: 4px solid rgba(0, 0, 0, 0.1);
    border-top-color: rgba(0, 0, 0, 0.6);
    animation: spin 0.8s linear infinite;
  }

  .depth {
    margin-top: 6px;
    font-size: 0.9rem;
    font-weight: 600;
  }

  @keyframes spin {
    to {
      transform: rotate(360deg);
    }
  }

  nav {
    position: absolute;
    z-index: 1000;
    bottom: 10px;
  }
  nav.left {
    left: 10px;
  }

  nav.right {
    right: 10px;
  }

  .node-list {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
  }
  .node {
    padding: 6px 10px;
    border: 1px solid #ccc;
  }

  .links-table {
    width: 100%;
    border-collapse: collapse;
  }

  .links-table td {
    padding: 4px 8px;
    border-bottom: 1px solid #ddd;
  }
</style>
