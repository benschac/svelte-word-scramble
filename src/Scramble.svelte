<script lang="ts">
  import { onMount } from "svelte";
  import Letter from "./Letter.svelte";

  export let name: string;
  export let duration: number;
  // Speed of the scramble
  export let speed: number;

  let scrambled: Scramble[];
  let unscrambled: Scramble[] = [];

  let nameLength: number = name.length;
  let nameArray: string[] = Array.from(name);

  // TODOS
  // API

  // Figure out how to dynamically get at slot

  // Speed of transitions
  // Characters to Scramble
  // Colors to Scramble
  // Direction to Scramble
  // Totol time of Scramble
  //

  // TODO: Figure out what tick does.

  type Scramble = { letter: string; color: string };

  const nonAlpha = "!\"£$%^&*()#{}[]?<>'';:/`|@,.~-_=+€¥";
  const lowerCase = "abcdefghijklmnopqrstuvwxyz";
  const upperCase = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
  const numbers = "1234567890";

  let defaultEncoding: string[] = [
    ...Array.from(nonAlpha),
    ...Array.from(lowerCase),
    ...Array.from(upperCase),
    ...Array.from(numbers),
  ];

  let color = [
    "#66FF00",
    "#1974D2",
    "#08E8DE",
    "#FFF000",
    "#FFAA1D",
    "#FF007F",
  ];
  let i: number = 0;
  let done;

  let randomScramble: (number) => number = (length) =>
    Math.floor(Math.random() * length);

  function getScramble(timestamp: DOMHighResTimeStamp): void {
    setTimeout(() => {
      let scramble: Scramble[] = [];

      let totalScrambles = Math.floor(duration / speed);
      let currentScrables = Math.floor(timestamp / speed);

      let intervalOfScrambleShift = Math.floor(totalScrambles / nameLength);

      // The bug is here. Some times on each render will be able to push
      // a letter to unscrambled, other times it wont.

      if (currentScrables % intervalOfScrambleShift === 0) {
        // The promblem is that we might not get to the end of the list
        // if we don't have time.

        // We need to think about
        console.log(nameArray.length, i, totalScrambles, currentScrables);
        if (i < nameArray.length) {
          unscrambled = [
            ...unscrambled,
            { letter: nameArray[i], color: "#fffffff" },
          ];
          i += 1;
          console.log(i);
        }
      }

      for (let j = i; j < nameLength; j++) {
        scramble.push({
          color: color[randomScramble(color.length)],
          letter: defaultEncoding[randomScramble(defaultEncoding.length)],
        });
      }

      if (timestamp < duration) {
        window.requestAnimationFrame(getScramble);
      } else if (timestamp > duration) {
        // scrambled = [];
        // unscrambled = unscrambled;
        console.log({ unscrambled });
        done = true;
      }

      scrambled = scramble;
    }, speed);
  }
  let start;
  let originalValue;

  const singleScramble = (idx: number, array: Scramble[]) => {
    if (!originalValue) {
      originalValue = array;
    }
    console.log(originalValue, "from closure");
    return (timestamp: DOMHighResTimeStamp) => {
      // console.log("hi from single scramble", idx, { timestamp, duration });
      console.log(array[idx], "hi from array");
      // debugger;
      setTimeout(() => {
        if (!start) {
          start = timestamp;
        }
        array[idx] = {
          letter: defaultEncoding[randomScramble(defaultEncoding.length)],
          color: color[randomScramble(color.length)],
        };

        unscrambled = array;

        if (timestamp - start > duration) {
          unscrambled = originalValue;
          window.requestAnimationFrame(singleScramble(idx, unscrambled));
          start;
        }
      }, speed);
    };
  };

  function hover() {
    console.log("hovered");
  }
  const animateScramble = () => window.requestAnimationFrame(getScramble);

  onMount(() => {
    animateScramble();
  });

  function letterClick(event) {
    window.requestAnimationFrame(
      singleScramble(event.detail.detail, unscrambled)
    );
  }
</script>

<style>
  * {
    font-size: 10ch;
  }
</style>

<main>
  {#if unscrambled}
    {#each unscrambled as { letter, color }, idx}
      <Letter
        {idx}
        on:hover={hover}
        on:letterclick={letterClick}
        let:hovering={active}
        {letter}
        {color} />
    {/each}
  {/if}
  {#if scrambled}
    {#each scrambled as { letter, color }}
      <Letter on:hover={hover} let:hovering={active} {letter} {color} />
    {/each}
  {/if}

  <slot />
</main>
