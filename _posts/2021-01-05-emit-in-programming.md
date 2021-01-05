---
layout: post
title:  프로그래밍 에서 emit()
comments: true
---

프로그래밍을 하다보면 emit() 이라는 함수를 자주 보게 됩니다.

우리말로 emit 은 "방출하다" 로 번역할 수 있습니다.

컴퓨터사이언스에서 해당 용어에 대해 물어보는 스택오버플로 질문도 존재합니다.

[https://stackoverflow.com/questions/31270657/what-does-emit-mean-in-general-computer-science-terms](https://stackoverflow.com/questions/31270657/what-does-emit-mean-in-general-computer-science-terms)

emit() 는 보통 등록된 이벤트들을 실행시키는 함수 라고 생각하면 됩니다.

**등록한 이벤트를 방출한다.** 로 이해하면 될듯 합니다.

emit 을 구현한 코드들도 많이 있는데 php 로 구현된 EventEmitter를 보도록 합시다.

[https://github.com/igorw/evenement/blob/master/src/Evenement/EventEmitterTrait.php](https://github.com/igorw/evenement/blob/master/src/Evenement/EventEmitterTrait.php)

```php
public function emit($event, array $arguments = [])
    {
        if ($event === null) {
            throw new \InvalidArgumentException('event name must not be null');
        }

        $beforeOnceListeners = [];
        if (isset($this->beforeOnceListeners[$event])) {
            $beforeOnceListeners = \array_values($this->beforeOnceListeners[$event]);
        }

        $listeners = [];
        if (isset($this->listeners[$event])) {
            $listeners = \array_values($this->listeners[$event]);
        }

        $onceListeners = [];
        if (isset($this->onceListeners[$event])) {
            $onceListeners = \array_values($this->onceListeners[$event]);
        }

        if(empty($beforeOnceListeners) === false) {
            unset($this->beforeOnceListeners[$event]);
            foreach ($beforeOnceListeners as $listener) {
                $listener(...$arguments);
            }
        }

        if(empty($listeners) === false) {
            foreach ($listeners as $listener) {
                $listener(...$arguments);
            }
        }

        if(empty($onceListeners) === false) {
            unset($this->onceListeners[$event]);
            foreach ($onceListeners as $listener) {
                $listener(...$arguments);
            }
        }

        foreach ($this->children as $child) {
            $child->emit($event, $arguments);
        }
    }
```

event 종류에 따라 다르게 동작하는 (한번만 실행한다던지...) 기능들이 있지만 결과적으로는 등록된 이벤트를 실행하는걸 볼 수있습니다.

dart 언어로는 Broadcast Stream 을 이용하여 다음과 같이 간단하게 구현해 볼 수 있습니다.

```dart
import 'dart:async';

main() async {
  final StreamController ctrl = StreamController<int>.broadcast();

  var event1 = (data) {
    print("event1 executed, data : $data" );
  };
  var event2 = (data) {
    print("event2 executed, data : $data" );
  };

  final StreamSubscription subscription1 = ctrl.stream.listen((data) => event1(data));
  final StreamSubscription subscription2 = ctrl.stream.listen((data) => event2(data));

  for (int i = 1; i <= 3; i++) {
    ctrl.sink.add(i);
  }

  ctrl.close();

  await Future.delayed(const Duration(seconds: 1), () {});
}
```