# Some snippets I'm using cuz im lazy boi :fire:


### Getting a tweet and it's aggregated distributed counters

> Imagine your tweet goes viral and thousands of users are liking/fav it in a relative small time window. Since Firestore's limit is up to one write/second for a same document, you might get incorrect aggregated data because of too much contention. The idea here is to break your likes counter into multiple fragments called shards and run a transaction on a random shard reference when a user likes a tweet, to.. limit this contention.

> Note: there is a workaround with the RTDB but you have to copy the aggregated data to Firestore using a cron-job or something.

> Note2: there is another workaround using Cloud Dataflow and pubsub topics to aggregate a large amount of user inputs and write them back to Firestore. [Read more](https://medium.com/evenbit/aggregate-thousands-of-inputs-per-second-with-firebase-76111212b850)

---

#### Used libs
```ts
import { docData, collectionData } from 'rxfire/firestore'
import { Subject, combineLatest } from 'rxjs'
import { takeUntil } from 'rxjs/operators'
```

```ts
// used to be unsubscribed when component is destroyed/unmounted [!memory-leak]
destroy$: Subject<boolean> = new Subject<boolean>()
```

---

```ts
getTweet(tweetId: string) {
    // setting up some data flows
    const tweet$: Observable<{}> = docData(db.doc(`tweets/${tweetId}`), 'id')
    
    const shards$: Observable<{}[]> = collectionData(
      db.collection(`tweets/${tweetId}/shards`), 'id')
      
    const userLikesTweet$: Observable<{}> = docData(
      db.doc(`likes/${this.currentUserIdYouGetFromSomewhere}_${tweetdId}`),
      'id'
    )

    // combining them into one observable
    combineLatest(
      tweet$,
      shards$,
      userLikesTweet$,
      (tweet: Tweet, { isLiking }, shards) => {
        // merging the counter value of each shard
        const countLikes: number = shards.reduce((acc, { count }) => acc + count, 0)
           
        // returning a brand new object
        return { ...post, isLiking, countLikes }
      }
    )
      // fuk u memory leak
      .pipe(takeUntil(this.destroy$))
      .subscribe((post: Post) => (/* do whatever you please */)
  }
```
