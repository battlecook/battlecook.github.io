---
layout: post
title:  golang , c# byte array 값 비교 
comments: true
---

string 을 byte array 로 변환 할 때 변환되는 방식은 아스키 코드의 값으로 변환됩니다.

예를 들어 스트링 "test" 인 경우 아스키 코드표를 참고하면 다음과 같습니다.  

```
t => 116

e => 101

s => 115

t => 116
```

go 언어로 테스트 케이스를 짜보겠습니다.

```golang
func Test_ByteArrayDisplay(t *testing.T) {
	//given
	str := "test"

	//when
	actual := []byte(str)

	//then
	expected := []byte{116, 101, 115, 116}
	assert.Equal(t, expected, actual)
}
```
 
c# 언어로 테스트 케이스를 짜보겠습니다.
 
```csharp
[Test]
public void TestByteArrayDisplay()
{
    //given
    string str = "test";
    
    //when
    string actual = BitConverter.ToString(Encoding.UTF8.GetBytes(str));
    
    //then
    string expected = "74-65-73-74";
    Assert.AreEqual(expected, actual);
}
```
 
두 언어의 byte array 변수값을 눈으로 보고 싶을때가 있습니다. 

go 언어 기준으로 10진수로 바꿔서 프린트 해 보겠습니다.

```golang
fmt.Println([]byte("test"))
```

```csharp
private void displayDecimalByteArray(string str)
{
    string hexValues = BitConverter.ToString(Encoding.UTF8.GetBytes(str));
    string[] hexValuesSplit = hexValues.Split('-');

    Console.Out.Write("[");
    foreach (string hex in hexValuesSplit)
    {
        int value = Convert.ToInt32(hex, 16);
    
        Console.Out.Write(value);
        Console.Out.Write(" ");
    }

    Console.Out.Write("]");
}

string str = "test";

displayDecimalByteArray(str);        
```

결과값 
```
c#

[116 101 115 116 ]

golang

[116 101 115 116]
```
