# Some snippets I'm using cuz im lazy boi

### Used libs
```ts
import { docData, collectionData } from 'rxfire/firestore'
import { Subject, combineLatest } from 'rxjs'
import { takeUntil } from 'rxjs/operators'
```

```ts
// used to be unsubscribed when component is destroyed/unmounted [!memory-leak]
destroy$: Subject<boolean> = new Subject<boolean>()
```

### Getting a tweet and it's aggregated distributed counters

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
        const countLikes: number = shards.reduce((acc, { count }) => acc + count, 0)

        return { ...post, isLiking, countLikes }
      }
    )
      .pipe(takeUntil(this.destroy$))
      .subscribe((post: Post) => (/* do whatever you please */)
  }
```
