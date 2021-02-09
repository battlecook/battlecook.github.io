---
layout: post
title:  flutter, Stateless 위젯에서 Navigator.pop 후 rebuild 하는 방법
comments: true
---
flutter 에서 Navigator 에 위젯을 푸쉬한 후 pop 으로 되돌아 오면 push 했을때 와는 다르게 rebuild 를 하지 않는다.

```dart

import 'package:flutter/material.dart';

class OtherPageWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
        body: SafeArea(
            child: Center(
      child: Row(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: RaisedButton(
                onPressed: () {
                  Navigator.of(context).push(
                    MaterialPageRoute(
                      builder: (BuildContext context) => HomeWidget(),
                    ),
                  );
                },
                child: Text("push back")),
          ),
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: RaisedButton(
                onPressed: () {
                  Navigator.pop(context);
                },
                child: Text("pop back")),
          ),
        ],
      ),
    )));
  }
}

class SampleWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(home: HomeWidget());
  }
}

class HomeWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    print("build");
    return Scaffold(
      body: SafeArea(
        child: Center(
          child: RaisedButton(
            onPressed: () {
              Navigator.of(context).push(
                MaterialPageRoute(
                  builder: (BuildContext context) => OtherPageWidget(),
                ),
              );
            },
            child: Text("other page"),
          ),
        ),
      ),
    );
  }
}

void main() {
  runApp(SampleWidget());
}
```

위의 코드를 실행시켰을때 씬을 이동한 후 push 로 돌아올때는 build 라는글자가 콘솔에 찍히지만 pop으로 돌아올때는 찍히지 않는 것을 볼 수 있다.

pop시에 rebuild 하지 않는게 가끔은 문제가 될 수 있다. 첫번째 페이지의 데이터를 두 번째 페이지에서 지우고 돌아온다고 가정해 보자

```dart
import 'package:flutter/material.dart';

class RemoveTextButtonWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
        home: Scaffold(
            body: SafeArea(
              child: Center(
                child: Wrap(
                    alignment: WrapAlignment.center,
                    spacing: 10,
                    runSpacing: 10,
                    children: Repository.textList.map((String content) {
                      return RaisedButton(
                        onPressed: () {
                          Repository.textList.remove(content);
                          Navigator.pop(context);
                        },
                        child: Text("remove " + content),
                      );
                    }).toList()),
              ),
            )));
  }
}

class Repository {
  static List<String> textList = ['text1', 'text2'];
}

class SampleWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(home: HomeWidget());
  }
}

class HomeWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            crossAxisAlignment: CrossAxisAlignment.center,
            children: [
              Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: Repository.textList.map((String content) {
                return Padding(
                  padding: const EdgeInsets.all(8.0),
                  child: Text(content),
                );
              }).toList()),
              RaisedButton(
                onPressed: () {
                  Navigator.of(context).push(
                    MaterialPageRoute(
                      builder: (BuildContext context) => RemoveTextButtonWidget(),
                    ),
                  );
                },
                child: Text("other page"),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

void main() {
  runApp(SampleWidget());
}
```

두 번째 페이지에서 이미 데이터를 지웠지만 pop 후 돌아오면 rebuild 를 하지 않기 때문에 화면은 그대로 text1, text2 가 모두 그대로 그려져 있다.

해결방법은 첫 번째 페이지에서 push 로 페이지를 라우트 할때 then 으로 돌아왔을때의 이벤트를 작성해 주면 된다.

```dart
Navigator.of(context).push(
        MaterialPageRoute(
            builder: (BuildContext context) => RemoveTextButtonWidget(),
        ),
    ).then((value) {
        // pop 으로 돌아왔을 때 event 작성
    }
);
```

StatefulWidget 이라면 setState(() {}); 작성해주고 StatelessWidget 이라면 Stream 을 이용해서 rebuild를 시켜주면 된다.

StatelessWidget 에서 전체 코드는 다음과 같다.

```dart
import 'package:flutter/material.dart';
import 'dart:async';

class RemoveTextButtonWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
        home: Scaffold(
            body: SafeArea(
      child: Center(
        child: Wrap(
            alignment: WrapAlignment.center,
            spacing: 10,
            runSpacing: 10,
            children: Repository.textList.map((String content) {
              return RaisedButton(
                onPressed: () {
                  Repository.textList.remove(content);
                  Navigator.pop(context);
                },
                child: Text("remove " + content),
              );
            }).toList()),
      ),
    )));
  }
}

class Repository {
  static List<String> textList = ['text1', 'text2'];
}

class SampleWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(home: HomeWidget());
  }
}

class HomeWidget extends StatelessWidget {

  StreamController<List<String>> streamController = StreamController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            crossAxisAlignment: CrossAxisAlignment.center,
            children: [
              StreamBuilder(
                  stream: streamController.stream,
                  builder: (context, AsyncSnapshot<List<String>> snapshot) {
                    if(snapshot.hasData) {
                      return Row(
                          mainAxisAlignment: MainAxisAlignment.center,
                          children: snapshot.data.map((String content) {
                            return Padding(
                              padding: const EdgeInsets.all(8.0),
                              child: Text(content),
                            );
                          }).toList());
                    }
                    return Row(
                        mainAxisAlignment: MainAxisAlignment.center,
                        children: Repository.textList.map((String content) {
                          return Padding(
                            padding: const EdgeInsets.all(8.0),
                            child: Text(content),
                          );
                        }).toList());

                  }),
              RaisedButton(
                onPressed: () {
                  Navigator.of(context)
                      .push(
                    MaterialPageRoute(
                      builder: (BuildContext context) => RemoveTextButtonWidget(),
                    ),
                  )
                      .then((value) {
                        print(Repository.textList);
                    streamController.sink.add(Repository.textList);
                  });
                },
                child: Text("other page"),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

void main() {
  runApp(SampleWidget());
}
```

참고 자료

[https://stackoverflow.com/questions/49804891/force-flutter-navigator-to-reload-state-when-popping](https://stackoverflow.com/questions/49804891/force-flutter-navigator-to-reload-state-when-popping)