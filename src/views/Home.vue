<template>
  <div class="home">
    <button @click="requestKenHead">Shake it!!!</button>
    <img
      src="../assets/logo.png"
      :style="item.style"
      v-for="(item, key) in images"
      v-if="item.disable"
    >
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { of, merge, timer } from "rxjs";
import { webSocket, WebSocketSubject } from "rxjs/webSocket";
import { groupBy, mergeMap, takeUntil, finalize, tap } from "rxjs/operators";

@Component
export default class Home extends Vue {
  socket$: WebSocketSubject<string> | any = new WebSocketSubject(
    "ws://localhost:8080"
  );
  images: any = {};

  mounted() {
    const alphaHead = this.socket$.pipe(
      groupBy((data: any) => data.uid),
      mergeMap((singleAlphaStream: any) =>
        singleAlphaStream.pipe(
          takeUntil(timer(3000)),
          finalize(() => {
            
            const image = this.images[singleAlphaStream.key].disable = false;
            console.log(image);
            
          })
        )
      )
    );

    alphaHead.subscribe(
      (data: any) => {
        const { x, y, uid } = data;
        this.images[data.uid] = {
          uid: data.uid,
          disable: true,
          style: {
            position: "absolute",
            left: x + "px",
            top: y + "px",
          }
        };
        const obj = Object.assign({}, this.images);
        this.$set(this.images, 'images', obj)
      },
      (err: any) => console.error(err),
      () => console.warn("Completed!")
    );
    this.requestKenHead();
  }

  requestKenHead() {
    this.socket$.next({
      event: "events",
      data: { type: "REQUEST_KEN_HEAD" }
    });
  }
}
</script>
