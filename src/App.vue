<template>
  <section class="section">
    <b-tabs v-model="activeTab">
      <b-tab-item
        v-for="person of people$"
        :label="person.name"
        :key="person.id"
      ></b-tab-item>
    </b-tabs>
    <h1 class="title">{{ name$ }}</h1>
    <img v-stream:error="imageError$" :src="image$" />
  </section>
</template>

<script>
import { from, Subject, merge } from "rxjs"
import {
  pluck,
  map,
  switchMap,
  mapTo,
  catchError,
  share,
  combineLatest
  // startWith,
  // exhaustMap,
} from "rxjs/operators"

export default {
  data() {
    return {
      activeTab: 0
      // people: [
      //   { name: "Luke", id: 1 },
      //   { name: "Darth", id: 4 },
      //   { name: "Leia", id: 5 },
      //   { name: "Yoda", id: 0 }
      // ]
    }
  },
  subscriptions() {
    // this.click$ = new Subject()
    this.imageError$ = new Subject()
    const activeTab$ = this.$watchAsObservable("activeTab", {
      immediate: true
    }).pipe(pluck("newValue"))

    const createLoader = url => from(this.$http.get(url)).pipe(pluck("data"))

    const cache = {}
    const cachePerson = cache => url => {
      return cache[url] ? cache[url] : (cache[url] = createLoader(url))
    }

    const people$ = createLoader(
      "https://starwars.egghead.training/people"
    ).pipe(map(people => people.slice(0, 7)))

    const person$ = activeTab$.pipe(
      combineLatest(people$, (tabId, people) => people[tabId]),
      map(person => `https://starwars.egghead.training/people/${person.id}`),
      switchMap(cachePerson(cache)), // pause
      catchError(() =>
        // of({
        //   name: "Failed...:("
        // })
        createLoader("https://starwars.egghead.training/people/2")
      ),
      share()
    )

    // const disabled$ = merge(
    //   this.click$.pipe(mapTo(true)),
    //   person$.pipe(mapTo(false))
    // ).pipe(startWith(false))

    // const buttonText$ = disabled$.pipe(
    //   map(bool => (bool ? "Loading..." : "Load"))
    // )

    const name$ = person$.pipe(pluck("name"))
    const loadImage$ = person$.pipe(
      pluck("image"),
      map(image => `https://starwars.egghead.training/${image}`)
    )

    const failImage$ = this.imageError$.pipe(mapTo("cat2.jpeg"))

    const image$ = merge(loadImage$, failImage$)

    return {
      people$,
      name$,
      image$
      // disabled$,
      // buttonText$,
      // activeTab$
    }
  }
}
</script>

<style></style>
