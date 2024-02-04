# vespa-global-phase-empty-sorting-string-problem

When a global `ranking-phase` is combined with the `ranking.sorting=''` 
(i.e. an empty string) then hits are returned not in the order of the `relevance` value but the order of
the previous phase ranking. I find this behavior unexpected.

## Steps to reproduce

```shell
docker run --rm --detach \
  --name vespa \
  --publish 8080:8080 \
  --publish 19071:19071 \
  --publish 19050:19050 \
  vespaengine/vespa:8.296.15
```

Deploy the app and feed data:
```shell
vespa deploy -w 100
vespa feed ext/*
```

Make a search request:
```shell
vespa query \
  'select documentid 
  from sources * 
  where userInput("to") or true 
  limit 3' \
  ranking.profile='rank_albums' \
  ranking.sorting=''
```

Which returns docs like this:
```json
{
    "root": {
        "id": "toplevel",
        "relevance": 1.0,
        "fields": {
            "totalCount": 5
        },
        "coverage": {
            "coverage": 100,
            "documents": 5,
            "full": true,
            "nodes": 1,
            "results": 1,
            "resultsFull": 1
        },
        "children": [
            {
                "id": "id:mynamespace:music::hardwired-to-self-destruct",
                "relevance": 0.23999335849657655,
                "source": "music",
                "fields": {
                    "documentid": "id:mynamespace:music::hardwired-to-self-destruct"
                }
            },
            {
                "id": "id:mynamespace:music::love-is-here-to-stay",
                "relevance": 0.9377636546269059,
                "source": "music",
                "fields": {
                    "documentid": "id:mynamespace:music::love-is-here-to-stay"
                }
            },
            {
                "id": "id:mynamespace:music::when-we-all-fall-asleep-where-do-we-go",
                "relevance": 0.5533042987808585,
                "source": "music",
                "fields": {
                    "documentid": "id:mynamespace:music::when-we-all-fall-asleep-where-do-we-go"
                }
            }
        ]
    }
}
```

The hits are not sorted by the `relevance` value.
It might take a couple of requests to experience exactly this behaviour because of random scores.

If the `global-phase` is renamed into the `second-phase` then hits are sorted by the `relevance` value.

The only change from the sample app is the addition of the `global-phase` in the ranking profile.
```shell
global-phase {
    expression: random
    rerank-count: 10
}
```

We can put any expression in the `global-phase` that changes the relevance and the unexpected behavior is reproducible.
