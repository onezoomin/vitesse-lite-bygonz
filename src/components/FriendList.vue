<script setup lang="ts">
import { reactive } from 'vue'

const ztaxCDNURL = 'https://ztax-cdn.fission.app/bygonz/index.mjs'
const bygonz = await import(/* @vite-ignore */ztaxCDNURL)
const { doTest, liveQuery } = bygonz
const db = await doTest(bygonz) // as BygonzDexie & {Friends: any}

// this only shows the first array (after the first refresh) and is non-reactive:
// const friends = useObservable(liveQuery(() => db.Friends.toArray()) as any) as Ref<Friend[]> 

// consider https://github.com/dexie/Dexie.js/issues/1528#issuecomment-1085388832
const rxState = reactive({ allFriends: await db.Friends.toArray() })

const friendsObservable = liveQuery(async () => await db.Friends.toArray())
friendsObservable.subscribe({
  next: async (result: any[]) => {
    console.log('Got  result in vue:', { result })
    const resultWithAgeHistory = []
    for (const eachFriend of result) {
      resultWithAgeHistory.push({
        ...eachFriend,
        // TODO improve ergonomics
        ageHistory: (await eachFriend.getAttributeHistoryArray('age')).join(', '),
      })
    }

    rxState.allFriends = resultWithAgeHistory
  },
  error: error => console.error(error),
})

console.log({ db, rxState })
/// traditional dexie recomendations don't seem to work with vue3 composition
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
  <ul class="max-w-md m-auto">
    <li v-for="friend in rxState.allFriends" :key="friend.id" class="text-left">
      {{ friend.name }}, {{ friend.age }}{{ friend.ageHistory?.length ? `, Previous ages: ${friend.ageHistory}` : '' }}
    </li>
  </ul>
</template>
