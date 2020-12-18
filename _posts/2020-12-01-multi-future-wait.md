---
layout: post
title:  future, async, await 에서 여러 future 를 실행하고 작업이 끝날 때까지 대기하는 방법
comments: true
---


python 3.6.x
```python

import asyncio


loop = asyncio.get_event_loop()
async def test(number):
    print(number)

async def main():

    futures = []
    for i in range(20):
        future = test(i)
        futures.append(future)

    print("future count : " + str(len(futures)))
    (done, pending) = await asyncio.wait(futures)
    if pending:
        print("there is {} tasks not completed".format(len(pending)))
        for f in pending:
            f.cancel()
    for f in done:
        ret = await f
        if ret is not None:
            print(ret)

loop.run_until_complete(main())

```
dart 

future 리스트를 만들고 일이 처리 될 때 까지 wait  

```dart
import 'dart:async';

Future main() async {
  var data = [];
  var futures = <Future>[];
  for (var d in data) {
    futures.add(d.loadData());
  }
  await Future.wait(futures);
} 
```


#### 참고 사이트

[https://stackoverflow.com/questions/42176092/dartlang-wait-more-than-one-future](https://stackoverflow.com/questions/42176092/dartlang-wait-more-than-one-future)