<template>
  <section class="section">
    <button class="button" v-stream:click="{ subject: plus$, data: 2 }">
      +(2)
    </button>
    <button class="button" v-stream:click="minus$">-(1)</button>
    <h1 class="title">{{ count }}</h1>
  </section>
</template>

<script>
import { Subject, merge } from "rxjs"
import { startWith, scan, mapTo, pluck } from "rxjs/operators"

export default {
  subscriptions() {
    this.plus$ = new Subject().pipe(pluck("data"))
    this.minus$ = new Subject().pipe(mapTo(-1))

    return {
      count: merge(this.plus$, this.minus$).pipe(
        startWith(0),
        scan((total, change) => total + change)
      )
    }
  }
}
</script>

<style></style>
