<script setup lang="ts">
  // import type {BygonzDexie, Table}  from 'bygonz'

  // import { useObservable } from "@vueuse/rxjs";
  // import{ refFrom } from 'vuse-rx'
  import { reactive } from "vue"

  const ztaxCDNURL = 'https://ztax-cdn.fission.app/bygonz/index.mjs'
  const bygonz = await import(ztaxCDNURL/* @vite-ignore */ )
  const { doTest, liveQuery } = bygonz
  const db = await doTest(bygonz) as BygonzDexie & {Friends:Table}

  // const friends = useObservable(liveQuery(() => db.Friends.toArray()) as any) as Ref<Friend[]>
  let friends = reactive({allFriends: await db.Friends.toArray()})// = await db.Friends.toArray()

  const friendsObservable = liveQuery(async () => await db.Friends.toArray())
  friendsObservable.subscribe({
    next: result => {
      console.log('Got  result in vue:', { result })
      friends.allFriends = result
    },
    error: error => console.error(error),
  })

  console.log({db, friends})
  //  export default {
  //   name: "FriendList",
  //   setup() {
  //     return {
  //       db,
  //       friends: useObservable(
  //         liveQuery(() => db.friends.toArray())
  //       ),
  //     };
  //   },
  // };
</script>

<template>
    <ul>
      <li v-for="friend in friends.allFriends" :key="friend.id">
        {{ friend.name }}, {{ friend.age }}
      </li>
    </ul>
 
</template>
