---
layout: post
title:  You are a game bot 논문의 자기 유사도 (self-similarity) 설명
comments: true
---

NC 소프트에서 발표한 You are a game bot 논문 입니다.

영어

[https://www.ndss-symposium.org/wp-content/uploads/2017/09/you-are-game-bot-uncovering-game-bots-mmorpgs-via-self-similarity-wild.pdf](https://www.ndss-symposium.org/wp-content/uploads/2017/09/you-are-game-bot-uncovering-game-bots-mmorpgs-via-self-similarity-wild.pdf)

한국어

[https://www.koreascience.or.kr/article/JAKO201609562997970.pdf](https://www.koreascience.or.kr/article/JAKO201609562997970.pdf)

해당 논문은 NC 소프트가 리니지, 아이온, 블레이드 소울의 로그를 분석하여 어뷰져 들의 공통 특성을 찾아낸 것입니다. 여러 내용들이 있지만 그중에서 자기유사도 ( self-similarity ) 에 대해서 알아 보도록 하겠습니다.

논문내용을 보면 1) 단위시간당 유져들의 엑션을 벡터로 만들고 2) 그 벡터와 모든 원소가 1인 벡터의 코사인유사도 ( cosine similarity ) 를 구한 후, 3) 그 코사인유사도를 가지고 자기유사도 ( self-similarity ) 를 구하면 됩니다.

하나하나 보도록 하겠습니다.

첫 번째로 단위시간당 유져들의 엑션을 벡터로 만드는 과정입니다.

논문에는 이렇게 되어있습니다.

![gamelogs-to-vector]({{ site.url }}/assets/20210210/gamelogs_to_vector.png)

단위시간은 1분으로 잡았고, Event id 는 유져들의 행동 입니다. 예를들어 몬스터공격, 상점구매 등으로 보면 됩니다. 단위시간당 Event id 가 발생한 횟수로 행렬을 만듭니다.

두번째는 코사인유사도 입니다. 코사인 유사도는 다음의 식으로 구할 수 있습니다.

![cosine_similarity]({{ site.url }}/assets/20210210/cosine_similarity.png)

논문내용을 보면 아래와 같은 설명이 있습니다.

```
2) Measuring the cosine similarity between log vectors:
We measure the cosine similarity between each log vector
and the unit vector in which all elements are one. The cosine
similarity between two vectors is defined as follows:
```

여기서 unit vector 라는 단어가 나오는데 보통 unit vector 는 길이가 1인 벡터를 뜻하지만 여기서는 모든 원소가 1인 벡터를 뜻하는 거 같습니다. ( the unit vector in which all elements are one. )

코사인 유사도값은 -1에서 1의 범위를 가지는 스칼라 값입니다. ( 동일한 코사인각도면 1, 반대의 코사인 각도면 -1 )

이렇게 해서 구한 단위시간당 코사인 유사도 값들을 가지고 자기유사도 ( self-similarity ) 를 구 할 수 있습니다. 자기유사도의 공식은 다음과 같습니다.

```
H = 1 - 0.5 * σ

H : 자기유사도 ( self-similarity )
σ : 코사인유사도들의 표준편차 값
```

논문내용을 보면 봇들과 일반유져들의 자기유사도(self-similarity) 를 비교해보면 리니지, 아이온, B&S 모두에서 봇들이 일반유져들 보다 자기유사도 값이 높게 나온다고 합니다.

![comparison_between_bots_and_normal_users]({{ site.url }}/assets/20210210/comparison_between_bots_and_normal_users.png)

간단한 예시를 들어가면서 자기유사도 ( self-similarity ) 를 구해 보도록 하겠습니다.

유져 A 가 event 2개 몬스터공격,  아이템사용 를 하는경우를 가정하겠습니다.

12:00 분 : 몬스터공격 5회, 아이템사용 3회

12:01 분 : 몬스터공격 2회, 아이템사용 1회

12:02 분 : 몬스터공격 3회, 아이템사용 2회

를 했다고 한다면

12:00 분의 코사인 유사도는 (5,3) 과 (1,1) 를 가지고 구한 약 0.970142

12:01 분의 코사인 유사도는 (2,1) 과 (1,1) 를 가지고 구한 약 0.948683

12:02 분의 코사인 유사도는 (3,2) 과 (1,1) 를 가지고 구한 약 0.980580

0.970142, 0.948683, 0.980580 값의 표준편차는 약 0.0132784

자기유사도(self-similarity) 는 0.9933608 이 됩니다.

<br>

**자기유사도(self-similarity) 외에 어뷰져들의 특징**

논문을 보면 self-similarity 외에도 어뷰져를 특징이 나와있는데 간략히 소개해 보도록 하겠습니다.

![list_of_features]({{ site.url }}/assets/20210210/list_of_features.png)

event 종류는 3개,단위시간은 4개를 가진다고 가정

단위시간 1 : [1, 2, 0] 코사인 유사도 : 0.77

단위시간 2 : [0, 0, 0] 코사인 유사도 : 0

단위시간 3 : [3, 0, 0] 코사인 유사도 : 0.57

단위시간 4 : [3, 0, 0] 코사인 유사도 : 0.57

self similarity

0.8565

vector count

3  ( [1,2,0] , [0, 0, 0] [3, 0, 0] [3,0,0] 중 접속하지 않은 한번을 제외 한 값 )

uniq vector count

2  ( [0,0,0] 제외 [3,0,0] [1,2,0]  2개)

cosim zero count

1  ( 코사인 sim,  0.77 0, 0.57, 0.57 중 0인 값의 개수 )

vector mode

2 (  [1,2,0] 1회 [0,0,0] 제외 [3,0,0] 2회 )

total log count

9 ( 호출 횟수 1, 2 ,3, 3 의 합 )

<br>

**논문을 읽으면서 의문사항**

<br>

**1) 코사인유사도 의문**

제가 논문을 처음 보면서 들었던 의문은 만약에 유져A 가 몬스터공격3회, 아이템사용2회를 하고 유져B가 몬스터공격2회, 아이템사용3회를 하면 유져들이 다른행동을 했음에도 self-similarity 값이 같아 버리게 되지 않는가 하는 것이었습니다.

(3,2) 와 (1,1) 의 코사인 유사도 : 0.9805806

(2,3) 와 (1,1) 의 코사인 유사도 : 0.9805806

아마도 event 의 종류가 많아서 크게 영향을 주진 않을 것으로 보이지만 조금 검색을 해보았습니다.

저의 의문증은 코사인유사도의 유일성문제 라는것을 알게 되었고 그 외에도 이진값 문제도 있었습니다.

( 이진값 문제는 몬스터공격1회, 아이템사용1회 의 유져나 몬스터공격10회 아이템사용10회의 유져가 동일한 코사인유사도를 가지는 문제입니다. )

벡터간의 유사도를 표현하는 방법엔 여러가지가 있는데  [https://bab2min.tistory.com/566](https://bab2min.tistory.com/566) 해당 블로그에 자세히 설명되어 있습니다.

<br>

**2) 유져가 플레이하지 않고 있는 경우에 코사인 유사도값에 대한 처리**

유져가 플레이하지 않는 경우의 코사인 유사도에 대한 처리방법도 몇 가지가 있을 수 있습니다.

코사인유사도의 결과값을 0으로 처리한다던지, 아니면 그시간대에 유져를 뺀다던지 등입니다.

스택오버플로에 모든 원소를 0 으로로 넣었을때에 대한 질문도 있습니다.

[https://stackoverflow.com/questions/26699659/cosine-similarity-when-one-of-vectors-is-all-zeros](https://stackoverflow.com/questions/26699659/cosine-similarity-when-one-of-vectors-is-all-zeros)

<br>

**참고한 문서**

[https://wikidocs.net/24603](https://wikidocs.net/24603)

[https://ko.wikipedia.org/wiki/코사인_유사도](https://ko.wikipedia.org/wiki/%EC%BD%94%EC%82%AC%EC%9D%B8_%EC%9C%A0%EC%82%AC%EB%8F%84)

[https://onlinemschool.com/math/assistance/vector/angl/](https://www.calculator.net/standard-deviation-calculator.html?numberinputs=0.77%2C+0%2C+0.57&ctype=p&x=48&y=26)

[https://bab2min.tistory.com/566](https://bab2min.tistory.com/566)