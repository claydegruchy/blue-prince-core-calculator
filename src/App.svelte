<script lang="ts">
  import Dialog from "./lib/Blocks/Dialog.svelte";

  var coreNumer = "864555555";
  var valid = false;
  var progressReport = [];

  validate();
  // calculateCore(coreNumer);

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

  function progress(...str) {
    progressReport = [...progressReport, ...str];
  }

  async function calculateCore(number, depth = 0) {
    console.group("calcuating core" + depth);
    console.log("calcuating core", number, depth);
    if (depth < 1) {
      progress(`Starting calcuation for ${number}`);
    } else {
      progress(`Starting sub-calcuation for ${number}, depth:` + depth);
    }
    running = true;

    if (!validate()) {
      progress("Number not valid:" + number);
      console.log("not valid");
      return;
    }

    number = String(number);

    // break into groups
    let groupBreakQuanity = coreNumer.length - 3;
    console.log({ groupBreakQuanity });

    /*

you always get 3 cuts, as you must end up with 4 parts
so it makes more sense to take the full number and apply the cuts 
3 cut heads
each step over once the last has finished everything in its range


*/

    var numberGroups = [];

    var cutPosition1 = 1;
    var cutPosition2 = 1;
    var cutPosition3 = 1;

    progress("Finding number groupings");
    console.log("Finding number groupings");
    var cycles = 0;
    for (const ix of range(100)) {
      var loop = ix % 4;
      var remaining = number;
      var acc = [];

      cycles += loop == 0 ? 1 : 0;
      cutPosition1 = 1 + (loop == 1 ? cycles : 0);
      cutPosition2 = 1 + (loop == 2 ? cycles : 0);
      cutPosition3 = 1 + (loop == 3 ? cycles : 0);

      // we dont need to loop if theres only ever 3 cuts
      var [removed, rem] = splitAt(remaining, cutPosition1);
      remaining = rem;
      if (rem == "") continue;

      acc.push(removed);

      var [removed, rem] = splitAt(remaining, cutPosition2);
      remaining = rem;
      if (rem == "") continue;

      acc.push(removed);

      var [removed, rem] = splitAt(remaining, cutPosition3);
      remaining = rem;
      if (rem == "") continue;

      acc.push(removed);
      acc.push(remaining);
      var packed = JSON.stringify(acc);

      // ix += 1;

      if (!numberGroups.includes(packed)) {
        numberGroups.push(packed);
      }
    }

    numberGroups = numberGroups.map(JSON.parse);
    progress(`Found ${numberGroups.length} number group combinations`);
    console.log("Numebr groupings", numberGroups.length);
    console.log("Starting ordering calculation");
    progress(`Starting ordering calculation`);

    var lowest = Infinity;
    for (const numberGroup of numberGroups) {
      // yes there are smarter ways to do this but were working with a finte set
      var orderings = [
        ["ADD", "SUBTRACT", "DIVIDE", "MULTIPLY"],
        ["ADD", "SUBTRACT", "MULTIPLY", "DIVIDE"],
        ["ADD", "DIVIDE", "SUBTRACT", "MULTIPLY"],
        ["ADD", "DIVIDE", "MULTIPLY", "SUBTRACT"],
        ["ADD", "MULTIPLY", "SUBTRACT", "DIVIDE"],
        ["ADD", "MULTIPLY", "DIVIDE", "SUBTRACT"],
      ];
      for (const operations of orderings) {
        var current = 0;
        for (let index = 0; index < operations.length; index++) {
          const operation = operations[index];
          const currentNumber = Number(numberGroup[index]);
          if (current % 1 !== 0) {
            break;
          }
          switch (operation) {
            case "ADD":
              current += currentNumber;
              break;
            case "SUBTRACT":
              current -= currentNumber;
              break;
            case "DIVIDE":
              current /= currentNumber;
              break;
            case "MULTIPLY":
              current *= currentNumber;
              break;
          }
        }
        if (current % 1 == 0) {
          if (Math.abs(current) > 0 && Math.abs(current) < Math.abs(lowest)) {
            lowest = current;
            console.log("new candidate", lowest);
            progress("Found new candidate number:" + lowest);
          }
        }
      }
    }
    if (String(Math.abs(lowest)).length > 3) {
      console.log("too large, recursing");
      progress(
        "Smallest number is still too large:" + lowest,
        "Starting subcalcuation..."
      );
      lowest = await calculateCore(lowest, depth + 1);
    }
    if (depth < 1) console.log("Complete, result", lowest);
    console.groupEnd();

    return Number(lowest);
  }

  var input;
  var result;

  async function findCore() {
    progressReport = [];
    input = coreNumer;
    result = await calculateCore(coreNumer);
  }

  function* range(n) {
    for (let i = 0; i < n; i++) yield i;
  }

  function splitAt(arr, index) {
    return [arr.slice(0, index), arr.slice(index)];
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
    <h3>Blue Prince</h3>
    <h2><i>Numeric Core Calculator</i></h2>
    <div>
      <input
        class="blueprint"
        type="string"
        placeholder="Enter core number"
        bind:value={coreNumer}
        on:keyup={validate}
      />
    </div>
    <button disabled={!valid} on:click={findCore}>Start</button>

    {#if result != null}
      {#if Math.abs(result) < 1}
        <h3>There is no valid numeric core of {input}</h3>
      {:else}
        <h3>Numeric core of {input} is {result}</h3>
      {/if}
    {/if}
    {#if running}
      <ol class="reverse">
        <li>Calculating core...</li>
        {#each progressReport as log, i}
          <li>
            {log}
          </li>
        {/each}
      </ol>
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

  .reverse {
    counter-reset: reversed-counter;
    list-style: none;
    padding: 0;
    margin: 0 auto; /* centers the list block */
    width: max-content; /* shrink to fit content */
    display: flex;
    flex-direction: column-reverse;
  }

  .reverse li {
    counter-increment: reversed-counter;
    text-align: left; /* keeps text left-aligned */
  }

  .reverse li::before {
    content: counter(reversed-counter) ". ";
    margin-right: 0.5em;
  }
  main {
    display: inline-block; /* shrink to fit content */

    background-color: rgba(255, 255, 255, 0.2); /* subtle light overlay */
    padding: 20px;
    margin: 10px;
    color: white;
  }

 input.blueprint {
    font-family: 'Courier New', Courier, monospace;
    font-size: 1.1em;
    color: #ffffff; /* bright text for contrast */
    background-color: rgba(12, 96, 168, 0.8); /* semi-transparent blueprint blue */
    border: 2px solid #65b2f4; /* lighter blueprint line color */
    border-radius: 2px;
    padding: 0.5em 0.75em;
    width: 300px; /* wider box */
    text-align: center; /* center the text inside */
    box-shadow: inset 0 0 5px rgba(0,0,0,0.3);
    letter-spacing: 0.5px;
    outline: none;
    transition: box-shadow 0.2s, border-color 0.2s;
}

input.blueprint:focus {
    border-color: #4992d2; /* slightly darker accent on focus */
    box-shadow: 0 0 8px rgba(101,178,244,0.5), inset 0 0 5px rgba(0,0,0,0.4);
}

</style>
