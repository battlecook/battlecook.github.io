---
layout: post
title:  flutter, Key 클레스에서 인자로 넣었던 value 값 구하는 방법
comments: true
---

flutter 에서 foundation 라이브러리의 Key 클레스 global Key, Local Key, Value Key 등의 추상 클레스 입니다.

가끔 추상화된 Key 클레스를 리턴받아서 인자로 넣어줬던 value 값을 쓰고 싶은경우가 있습니다.

```dart
import 'package:flutter/material.dart';

class SimpleButtonWidget extends StatelessWidget {

  SimpleButtonWidget(Key key) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return RaisedButton(
        onPressed: () {},
        child: Text(key.toString()),
        );
  }
}

class SimpleButtonRowWidget extends StatelessWidget {

  @override
  Widget build(BuildContext context) {

    String value = "id";
    SimpleButtonWidget btn1 = new SimpleButtonWidget(Key(value));
    
    print(btn1.key.toString());
    
    return Scaffold(
      body: SafeArea(
        child: Center(
          child: Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [btn1],
          ),
        ),
      ),
    );
  }
}
```

위와 같은 경우에서  btn1.key.toString() 의 값은 [<'id'>] 값이 됩니다.

조금더 심플한 다른 코드를 보겠습니다.

```dart
import 'package:flutter/material.dart';

void main() {
  String value = "this is key";
  String toStringValue = Key(value).toString();

  print(value);
  print(toStringValue);
}
```

결과값

```
this is key
[<'this is key'>]
```

위와 같은경우에 인자로 넣었던 값을 얻는 방법이 몇가지가 있습니다.

첫번째는 Key 추상화 객체를 구현한 실제 runtime type 의 객체로 타입캐스팅 후 value 값을 얻는 것입니다.

```dart
import 'package:flutter/material.dart';

void main() {
  String value = "this is key";
  Key key = Key(value);
  String toStringValue = key.toString();
  String typeCastingValue = (key as ValueKey).value;

  print(value);
  print(toStringValue);
  print(typeCastingValue);
}
```

결과값

```dart
this is key
[<'this is key'>]
this is key
```

두번째 방법은 toString() 한 값에서  추가된 스트링을 제거해줍니다.

예를들어 ValueKey 의 경우 "[<'" , "'>]" 를 제거해 줍니다.

```dart
String removeQuotes(Key key) {
  return key.toString().substring(3,key.toString().length - 3);
}
```

두번째 방법의 경우 아래처럼 toString() 함수가 개별 구현클레스 마다 다르므로 첫번째 방법을 쓰는 것이 좋습니다.

ObjectKey 의 toString()

```dart
@override
  String toString() {
    if (runtimeType == ObjectKey)
      return '[${describeIdentity(value)}]';
    return '[${objectRuntimeType(this, 'ObjectKey')} ${describeIdentity(value)}]';
  }
```

ValueKey 의 toString()

```dart
@override
  String toString() {
    final String valueString = T == String ? "<'$value'>" : '<$value>';
    // The crazy on the next line is a workaround for
    // https://github.com/dart-lang/sdk/issues/33297
    if (runtimeType == _TypeLiteral<ValueKey<T>>().type)
      return '[$valueString]';
    return '[$T $valueString]';
  }
```

참고 자료

타입케스트 오퍼레이션 as

[https://dart.dev/guides/language/language-tour#type-test-operators](https://dart.dev/guides/language/language-tour#type-test-operators)

flutter foundation 라이브러리 key 객체 코드 [https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/foundation/key.dart](https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/foundation/key.dart)

깃허브 flutter 레포 issue

[https://github.com/flutter/flutter/issues/27241](https://github.com/flutter/flutter/issues/27241)