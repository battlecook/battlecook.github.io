---
layout: post
title:  flutter, RaisedButton 위젯에서 버튼을 disable 하는 방법
comments: true
---

플루터 RaisedButton 위젯의 인자는 다음과 같습니다.

```
 const RaisedButton({
    Key key,
    @required VoidCallback onPressed,
    VoidCallback onLongPress,
    ValueChanged<bool> onHighlightChanged,
    ButtonTextTheme textTheme,
    Color textColor,
    Color disabledTextColor,
    Color color,
    Color disabledColor,
    Color focusColor,
    Color hoverColor,
    Color highlightColor,
    Color splashColor,
    Brightness colorBrightness,
    double elevation,
    double focusElevation,
    double hoverElevation,
    double highlightElevation,
    double disabledElevation,
    EdgeInsetsGeometry padding,
    ShapeBorder shape,
    Clip clipBehavior = Clip.none,
    FocusNode focusNode,
    bool autofocus = false,
    MaterialTapTargetSize materialTapTargetSize,
    Duration animationDuration,
    Widget child,
  })
```

onPressed 버튼을 disable 하는 방법은 onPressed 값을 null 로 주면 됩니다. 

onPressed 값을 null 로 주면 disabledColor 에 지정한 값이 적용이 됩니다.

아래버튼을 눌렀을때 enable, disable 이 변경 되면서 onPressed 값이 등록된콜백 과 null 값으로 변환 됩니다. 

onPressed 값이 null 일 경우 disabledTextColor, disabledColor 값이 적용되는 걸 확인 할 수 있습니다.

```flutter

import 'package:flutter/material.dart';
import 'package:flutter/widgets.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(home: Scaffold(body: Center(child: Button())));
  }
}

class Button extends StatefulWidget {
  ButtonState createState() => ButtonState();
}

class ButtonState extends State<Button> {
  bool isEnabled = true;

  onPressedFunction() {
    print('Clicked');
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        body: Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: <Widget>[
          RaisedButton(
            onPressed: isEnabled ? () => onPressedFunction() : null,

            disabledTextColor: Colors.black,
            disabledColor: Colors.red,

            color: Colors.blue,
            textColor: Colors.white,

            child: Text('Sample Button'),
          ),
          RaisedButton(
            onPressed: () {
              setState(() {
                isEnabled == true ? isEnabled = false : isEnabled = true;
              });
            },
            color: Colors.green,
            textColor: Colors.white,
            child: Text('Enable/Disable change button'),
          ),
        ],
      ),
    ));
  }
}

```



 ![flutter_onPressed_enable]({{ site.url }}/assets/20200113/flutter_onPressed_enable.jpg)
 ![flutter_onPressed_disable]({{ site.url }}/assets/20200113/flutter_onPressed_disable.jpg)



#### 참고 사이트

[https://flutter-examples.com/flutter-enable-disable-raisedbutton/](https://flutter-examples.com/flutter-enable-disable-raisedbutton/)