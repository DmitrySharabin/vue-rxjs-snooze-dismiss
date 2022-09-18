<!-- Rebuilt from this inspiring talk: https://youtu.be/_74d-NdeqOI -->

<script setup lang="ts">
import { ref } from "vue";

import { useObservable, fromEvent } from "@vueuse/rxjs";

import { interval, of } from "rxjs";
import { scan, startWith, takeWhile, concatWith, repeat, takeUntil, map } from "rxjs/operators";

const wakeUp = "Wake Up! 🎉";

// DOM elements
const snoozeButton = ref<HTMLButtonElement>(null);
const dismissButton = ref<HTMLButtonElement>(null);

// Streams
const snooze$ = fromEvent(snoozeButton, "click");
const dismiss$ = fromEvent(dismissButton, "click");

const counter$ = interval(1000).pipe(
  startWith(5),
  scan(time => time - 1),
  takeWhile(time => time > 0),
  concatWith(of(wakeUp)),
  repeat({
    delay: () => snooze$.pipe(takeUntil(dismiss$))
  }),
  concatWith(of("Have a wonderful day! 🤗"))
);
const counter = useObservable(counter$);

const disabled$ = counter$.pipe(
  map(time => time !== wakeUp)
);
const disabled = useObservable(disabled$);
</script>

<template>
  <output>{{ counter }}</output>
  <button ref="snoozeButton" :disabled="disabled">Snooze</button>
  <button ref="dismissButton" :disabled="disabled">Dismiss</button>
</template>

<style scoped>
output {
  display: block;
  margin-block: 0.3em;
  font-family: 'Courier New', Courier, monospace;
  font-size: 200%;
  line-height: 1.2;
}

button + button {
  margin-inline-start: 0.5em;
}
</style>
