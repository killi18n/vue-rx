<template>
  <section class="section">
    {{activeTab$}}
    <b-tabs v-model="activeTab">
      <b-tab-item v-for="person of people$" :key="person.id" :label="person.name"></b-tab-item>
    </b-tabs>
    <h1 class="title">{{message$}}</h1>
    <h1 class="title">{{timesTwo$}}</h1>
    <h1 class="title">{{timesThree$}}</h1>
    <h1 class="title">{{random$}}</h1>
    <section class="flexSection">
      <div v-bind:key="number" v-for="number in personNumber">
        <button
          class="button"
          :disabled="disabled$"
          v-stream:click="{subject: click$, data: number}"
          style="margin-right: 1rem"
        >{{buttonText$}}</button>
      </div>
    </section>
    <!-- <h1 class="title">{{person$}}</h1> -->
    <h1 class="title">{{luke$}}</h1>
    <h1 class="title">{{name$}}</h1>

    <img v-stream:error="imageError$" :src="image$" alt>
  </section>
</template>

<script>
import { interval, of, from, merge, combineLatest } from "rxjs";
import {
  map,
  pluck,
  // switchMap,
  mapTo,
  catchError,
  share,
  startWith,
  exhaustMap,
  switchMap
} from "rxjs/operators";

export default {
  domStreams: ["click$", "imageError$"],
  subscriptions: function() {
    const activeTab$ = this.$watchAsObservable("activeTab", {
      immediate: true
    }).pipe(pluck("newValue"));

    const createLoader$ = url => from(this.$http.get(url)).pipe(pluck("data"));

    const people$ = createLoader$(`https://starwars.egghead.training/people`);

    const interval$ = interval(1000);
    const timesTwo$ = interval$.pipe(map(i => i * 2));
    const timesThree$ = interval$.pipe(map(i => i * 3));
    const message$ = of("vue-rx lessons");
    const random$ = this.click$.pipe(map(() => Math.random()));

    // const luke$ = this.click$.pipe(
    //   pluck("data"),
    //   map(id => `https://starwars.egghead.training/people/${id}`),
    //   exhaustMap(createLoader$),
    //   catchError(() =>
    //     createLoader$("https://starwars.egghead.training/people/1")
    //   ),
    //   share()
    // );

    const luke$ = activeTab$.pipe(
      switchMap(id =>
        combineLatest(people$, people => {
          return people[id].id;
        })
      ),
      map(id => `https://starwars.egghead.training/people/${id}`),
      exhaustMap(createLoader$),
      catchError(() =>
        createLoader$("https://starwars.egghead.training/people/1")
      ),
      share()
    );

    const disabled$ = merge(
      this.click$.pipe(mapTo(true)),
      luke$.pipe(mapTo(false))
    );

    const buttonText$ = disabled$.pipe(
      startWith(false),
      map(bool => (bool ? "Loading..." : "Load"))
    );

    const name$ = luke$.pipe(pluck("name"));
    const loadImage$ = luke$.pipe(
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
      // person$,
      luke$,
      name$,
      image$,
      disabled$,
      buttonText$,
      activeTab$,
      people$
    };
  },
  data: function() {
    return {
      personNumber: 10,
      activeTab: 0,
      people: [
        {
          name: "Luke",
          id: 1
        },
        {
          name: "Darth",
          id: 4
        },
        {
          name: "Leia",
          id: 5
        },
        {
          name: "Yoda",
          id: 20
        }
      ]
    };
  }
};
</script>

<style>
.flexSection {
  display: flex;
}
</style>
