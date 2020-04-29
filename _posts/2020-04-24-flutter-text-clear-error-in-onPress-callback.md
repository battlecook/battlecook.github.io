---
layout: post
title:  flutter, onPress 콜백에서 TextEditingController 의 clear 사용시 에러 해결
comments: true
---

아래는 인풋박스에서 텍스트를 쓰고 clear버튼 (X 아이콘 ) 을 누르면 썼던 글자가 지워지는 코드입니다.

```flutter
import 'package:flutter/material.dart';
import 'package:flutter/widgets.dart';

void main() => runApp(TestWidget());

class TestWidget extends StatelessWidget {
  final _controller = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
        home: Scaffold(
      body: SingleChildScrollView(
        padding: const EdgeInsets.symmetric(vertical: 16),
        child: Column(
          children: <Widget>[
            const SizedBox(
              height: 16,
            ),
            Padding(
              padding: const EdgeInsets.symmetric(vertical: 15, horizontal: 15),
              child: TextField(
                controller: _controller,
                focusNode: FocusNode(),
                decoration: InputDecoration(
                  enabledBorder: const OutlineInputBorder(
                    borderSide: BorderSide(
                      color: Color(0x4437474F),
                    ),
                  ),
                  focusedBorder: OutlineInputBorder(
                    borderSide: BorderSide(color: Theme.of(context).primaryColor),
                  ),
                  suffixIcon: IconButton(
                      onPressed: () {
                        _controller.clear();
                      },
                      icon: Icon(Icons.clear)
                  ),
                  border: InputBorder.none,
                  hintText: "input here...",
                  contentPadding: const EdgeInsets.only(left: 16, right: 20, top: 14, bottom: 14,),
                ),
              ),
            ),
          ],
        ),
      ),
    ));
  }
}
```


 ![flutter_input_box]({{ site.url }}/assets/20200424/flutter_input_box.jpg)
 ![flutter_input_box_text]({{ site.url }}/assets/20200424/flutter_input_box_text.jpg)
 
해당 인풋박스에서 X 아이콘을 클릭하면 동작은 잘 하지만 아래와 같은 에러코드가 발생했습니다.

```flutter
════════ Exception caught by gesture ═══════════════════════════════════════════════════════════════
The following assertion was thrown while handling a gesture:
invalid text selection: TextSelection(baseOffset: 3, extentOffset: 3, affinity: TextAffinity.upstream, isDirectional: false)

When the exception was thrown, this was the stack: 
#0      TextEditingController.selection= (package:flutter/src/widgets/editable_text.dart:193:7)
#1      EditableTextState._handleSelectionChanged (package:flutter/src/widgets/editable_text.dart:1485:23)
#2      RenderEditable._handleSelectionChange (package:flutter/src/rendering/editable.dart:417:7)
#3      RenderEditable.selectPositionAt (package:flutter/src/rendering/editable.dart:1568:5)
#4      RenderEditable.selectPosition (package:flutter/src/rendering/editable.dart:1539:5)
...
Handler: "onTapUp"
Recognizer: _TransparentTapGestureRecognizer#d8176
  debugOwner: _TextSelectionGestureDetectorState#f3402
  state: ready
  won arena
  finalPosition: Offset(261.7, 73.1)
  finalLocalPosition: Offset(246.7, 31.1)
  button: 1
  sent tap down
════════════════════════════════════════════════════════════════════════════════════════════════════
```

해결 방법은 onPressed 콜백 코드를 다음과 같이 고치는 것 입니다.

기존 코드

```
            suffixIcon: IconButton(
                onPressed: () {
                  _controller.clear();
                },
                icon: Icon(Icons.clear)
            ),
```

수정 코드

```
            suffixIcon: IconButton(
                onPressed: () {
                  WidgetsBinding.instance.addPostFrameCallback((_) => _controller.clear());
                },
                icon: Icon(Icons.clear)
            ),
```


#### 참고 사이트

[https://github.com/flutter/flutter/issues/17647](https://github.com/flutter/flutter/issues/17647)

[https://www.didierboelens.com/2019/04/addpostframecallback/](https://www.didierboelens.com/2019/04/addpostframecallback/)