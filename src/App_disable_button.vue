<template>
  <section class="section">
    <button class="button" v-stream:click="click$" :disabled="disabled$">
      {{ buttonText$ }}
    </button>
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
  startWith
} from "rxjs/operators"

export default {
  subscriptions() {
    this.click$ = new Subject()
    this.imageError$ = new Subject()

    const createLoader = url => from(this.$http.get(url)).pipe(pluck("data"))

    const luke$ = this.click$.pipe(
      mapTo("https://starwars.egghead.training/people/1"),
      switchMap(createLoader),
      catchError(() =>
        // of({
        //   name: "Failed...:("
        // })
        createLoader("https://starwars.egghead.training/people/2")
      ),
      share()
    )

    const disabled$ = merge(
      this.click$.pipe(mapTo(true)),
      luke$.pipe(mapTo(false))
    ).pipe(startWith(false))

    const buttonText$ = disabled$.pipe(
      map(bool => (bool ? "Loading..." : "Load"))
    )

    const name$ = luke$.pipe(pluck("name"))
    const loadImage$ = luke$.pipe(
      pluck("image"),
      map(image => `https://starwars.egghead.training/${image}`)
    )

    const failImage$ = this.imageError$.pipe(mapTo("cat2.jpeg"))

    const image$ = merge(loadImage$, failImage$)

    return {
      name$,
      image$,
      disabled$,
      buttonText$
    }
  }
}
</script>

<style></style>
