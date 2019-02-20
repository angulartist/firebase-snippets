# Some snippets I'm using cuz im lazy boi :fire:

### Used libs

- RxFire
- RxJS

> here, db is just holding the firestore() function

### Getting a tweet and it's aggregated distributed counters

> Imagine your tweet goes viral and thousands of users are liking/fav it in a relative small time window. Since Firestore's limit is up to one write/second for a same document, you might get incorrect aggregated data because of too much contention. The idea here is to break your likes counter into multiple fragments called shards and run a transaction on a random shard reference when a user likes a tweet, to.. limit this contention.

> Note: there is a workaround with the RTDB but you have to copy the aggregated data to Firestore using a cron-job or something.

> Note2: there is another workaround using Cloud Dataflow and pubsub topics to aggregate a large amount of user inputs and write them back to Firestore. [Read more](https://medium.com/evenbit/aggregate-thousands-of-inputs-per-second-with-firebase-76111212b850)

---

```ts
export interface Post {
  id: string
  authorName: string
  content: string
  countLikes?: number
}

export interface Shard {
   // you might (not) grab the id
   id?: string
   count: number
}
```

```ts
getTweet(tweetId: string) {
    // setting up some data flows
    const tweet$: Observable<Post> = docData(db.doc(`tweets/${tweetId}`), 'id')
    
    const shards$: Observable<Shard[]> = collectionData(
      db.collection(`tweets/${tweetId}/shards`), 'id')

    // combining them into one observable
    combineLatest(
      tweet$,
      shards$,
      (tweet: Tweet, shards: Shard[]) => {
        // merging the counter value of each shard
        const countLikes: number = shards.reduce((acc, { count }) => acc + count, 0)
           
        // returning a brand new object
        return { ...post, countLikes }
      }
    )
      // fuk u memory leak
      .pipe(takeUntil(/* some Subject<boolean> */))
      .subscribe((post: Post) => (/* do whatever you please */)
  }
```

### Basic transaction to add a player to a room

> Imagine you have a basic game holding some rooms and each room have 2 slots. One slot for the owner, one slot for the owner's opponent. With some high traffic app, you might get into this situation : multiple players trying to join the same room at the same time. You might end up with 5, 6 or 20 players in a room and that shit gonna breaks. So, to deal with these concurrent situations, you have to use transactions. In this example, we're reading the up-to-date room state before adding a player. If the room is still open... add a player, otherwise reject.

---

```ts
const addPlayerToRoom = (opponentId: string, roomRef: FirebaseFirestore.DocumentReference): Promise<void> => {
  return db.runTransaction(async t => {
    try {
      // Getting back the room document
      const roomSnapshot = await t.get(roomRef)
      // Grab the key we want to check
      const { state } = roomSnapshot.data()
      // If the room is still open, add the opponent
       if (state === STATE.OPEN) {
         t.update(roomRef, { opponentId, state: STATE.CLOSED })
         return Promise.resolve('Added opponent to the room. Closed the room.');
       } else {
         return Promise.reject('Room is full.');
       }
    } catch (error) {
      throw new Error(`Error addPlayerToRoom: ${error}`)
    }
  })
}
```
