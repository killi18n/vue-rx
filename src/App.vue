<template>
  <section class="section">
    <h1 class="title">{{message$}}</h1>
    <h1 class="title">{{timesTwo$}}</h1>
    <h1 class="title">{{timesThree$}}</h1>
    <h1 class="title">{{random$}}</h1>
    <h1 class="title">{{person$}}</h1>
    <h1 class="title">{{luke$}}</h1>
    <h1 class="title">{{name$}}</h1>
    <img v-stream:error="imageError$" :src="image$" alt>
    <button class="button" v-stream:click="click$">Click</button>
  </section>
</template>

<script>
import { interval, of, from, merge } from "rxjs";
import { map, pluck, switchMap, mapTo } from "rxjs/operators";

export default {
  domStreams: ["click$", "imageError$"],
  subscriptions: function() {
    const interval$ = interval(1000);
    const timesTwo$ = interval$.pipe(map(i => i * 2));
    const timesThree$ = interval$.pipe(map(i => i * 3));
    const message$ = of("vue-rx lessons");
    const random$ = this.click$.pipe(map(() => Math.random()));
    const person$ = from(
      this.$http.get("https://starwars.egghead.training/people/2")
    ).pipe(pluck("data"));
    const luke$ = this.click$.pipe(switchMap(() => person$));

    const name$ = person$.pipe(pluck("name"));
    const loadImage$ = person$.pipe(
      map(({ image }) => `https:///starwars.egghead.training/${image}`)
    );

    const failImage$ = this.imageError$.pipe(
      mapTo(`http://via.placeholder.com/400x400`)
    );
    const image$ = merge(loadImage$, failImage$);
    return {
      timesTwo$,
      timesThree$,
      message$,
      random$,
      person$,
      luke$,
      name$,
      image$
    };
  }
};
</script>

<style>
</style>
