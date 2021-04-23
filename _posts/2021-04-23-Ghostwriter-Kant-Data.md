---
Title : "Ghostwriter Kant Data Preprocess"
layout : post
date : 2021-04-17
category : Internship
blog : true
author : dleunji
description : Network Layer
---

### Ghostwriter Data preprocess

Teachable-NLP는 코드를 따로 작성할 필요없이 **하나의 텍스트 파일**만으로 GPT-2을 튜닝할 수 있는 프로그램입니다.

이를 활용하여 철학과 학부 시절 자주하던 상상을 실현하고자 칸트의 '순수이성비판'을 활용하여 칸트식의 문장을 생성할 수 있는 모델을 생성하기를 마음 먹었습니다. 단순히 문장을 생성하는 것보다는 스토리라인을 덧붙여 마치 칸트가 대필작가(Ghostwriter)가 되어 학생들 대신 글을 써주는 방식으로 프로그램, [Ghostwriter Kant](https://master-kant-dleunji.endpoint.ainize.ai)를 구상하였습니다.

1. **Acquiring the data**

칸트의 가장 대표적인 저서 중 '순수이성비판(The Critique of Pure Reason) 은 [Project Gutenberg](https://www.gutenberg.org)에서 무료로 txt 형식으로 구할 수 있었습니다. 원문은 독일어로 작성되었으나, 원활한 학습과 생성된 글의 활용성을 고려하여 영어 번역본을 데이터로 선정하였습니다. 그리고 이는 고전 도서로, Public Domain License 상태였습니다. 이로써 학습에 활용하여도 저작권적인 문제가 발생하지 않음을 확인하였습니다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f147c9d1-e2d8-47e9-8fda-541cf8c161a1/_2021-04-15__10.33.30.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f147c9d1-e2d8-47e9-8fda-541cf8c161a1/_2021-04-15__10.33.30.png)

**2. Preprocessing the data**

해당 데이터를 바로 사용하기에는 아래와 같은 불필요한 헤딩과 도식들이 섞여 있었습니다.

SECOND, INTRODUCTION, I.과 같은 헤딩들은 글의 구조를 명료하게 하나, 칸트의 내용과 무관하였습니다. 이는 문장 생성 시 문맥을 해치기 때문에 헤딩을 제거하였습니다.

```
SECOND DIVISION—TRANSCENDENTAL LOGIC

TRANSCENDENTAL DIALECTIC.
INTRODUCTION.

I. Of Transcendental Illusory Appearance

We termed dialectic in general a logic of appearance. This does not
signify a doctrine of probability; for probability is truth, only
cognized upon insufficient grounds, and though the information it gives
us is imperfect, it is not therefore deceitful. Hence it must not be
separated
```

또한 칸트의 인식론을 설명하는 도식은 불규칙한 공백과 함께 단순한 낱말의 열거로 인식되기 때문에 학습에 지장을 줄 것이라 예상하였습니다. 이러한 불필요한 도식들을 제거하기 위해서 공백으로 시작하는 문장을 모두 제거하기로 결정하였습니다. 텍스트 파일에서 유효한 문장은 공백이 아닌 문자로 시작함을 확인하였기에 가능한 처리였습니다.

```
											NOTHING
                        AS

                        1
                Empty Conception
                 without object,
                  _ens rationis_
           2                               3
     Empty object of               Empty intuition
      a conception,                without object,
     _nihil privativum              ens imaginarium_
                        4
                   Empty object
                 without conception,
                  _nihil negativum_
```

간단한 텍스트 처리였기에 `Python` 코드로 전처리 작업하였습니다.

PREFACE 이전의 텍스트 전반부는 출판 관련 내용과 차례에 해당하여 처리하지 않고, 텍스트 파일을 줄마다 확인한 후(readlines) PREFACE부터 필요한 부분만 추출하였습니다. 특히 문장의 도입부를 천천히 살피며 어떠한 패턴으로 헤딩과 공백이 있는지부터 살폈습니다.

그 결과, Section, Chaper, Introduction 혹은 주로 대문자로 이루어진 단어, 논리학 용어, 로마자 등이 처리되어야 했습니다. 만약 괄호와 같은 부가적인 내용을 제외하고 활용할 수 있는 문장들은 substring을 통해 최대한 문장 자원을 버리지 않고, 문맥을 유지하려 노력하였습니다.

```python
file_name = "The Critique of Pure Reason.txt"
f = open(file_name, "rt", encoding='utf-8')
file = f.readlines()
f.close()
sentences = []
start = 0
for line in file: 
  if line.startswith('PREFACE'): 
	# PREFACE부터 데이터 가능성을 고려하였습니다.
	# 변수 start는 이를 위한 flag 역할을 수행하였습니다. 
    start = 1
    continue
  elif start != 1:
    continue
  elif line.startswith('End'):
	# PREFACE와 반대로 END로 문장이 시작할 시에 뒤의 부수적인 문장은 고려하지 않기 위해
	# break를 통해 for loop를 나옵니다.
    break
  elif line.startswith(' '):
	# 앞서 말씀드린 도식 외에도 주석 또한 공백으로 시작되어, 이 경우 모두 제거하였습니다.
    continue
	# 헤딩 제거
  elif line.startswith('Section'):
    continue
  elif line.startswith('Chapter'):
    continue
  elif line.startswith('Introduction'):
    continue
  elif line.startswith('FIRST') or line.startswith('SECOND') or line.startswith(
      'THIRD'):
    continue
  elif line.startswith('PROOF') or line.startswith('ANTITHESIS'):
    continue
  elif line.startswith('OBSERVATION'):
    continue
  elif line.startswith('ON THE THESIS'):
    continue
  elif line.startswith('I.') or line.startswith('II.') 
		or line.startswith('III.') or line.startswith('IV.') 
		or line.startswith('V.') or line.startswith('VI.'):
		# 제목에서 기호만 제외한 후 유효한 제목만 남겼습니다.
    line = line[line.find('.') + 2:]
  elif line.startswith('-'):
    continue
  elif line.startswith('§'):
    continue
	# 개행 문자 제거
  elif line == "\\n":
    continue
  elif line.startswith('('):
    line = line[4:]
  sentences.append(line)
```

그 결과 간결한 문장들로 이루어진 1.2MB의 데이터로 정리되었습니다. 일부 독일어로 된 고유명사와 콜론(:)과 같은 문장들도 있었지만, 이러한 문들을 제거할 시에 문맥이 지나치게 흐려져, 얼마 안되는 소수의 문장임을 감안하고 데이터로 활용하였습니다.

다음은 전처리가 이루어진 데이터의 도입입니다.

```
Human reason, in one sphere of its cognition, is called upon to
consider questions, which it cannot decline, as they are presented by
its own nature, but which it cannot answer, as they transcend every
faculty of the mind.
It falls into this difficulty without any fault of its own. It begins
with principles, which cannot be dispensed with in the field of
experience, and the truth and sufficiency of which are, at the same
time, insured by experience. With these principles it rises, in
obedience to the laws of its own nature, ...
```

**3. Training with Teachable-NLP**

Teachable-NLP에 위 데이터를 업로드하여 튜닝하였습니다. 이미 이전에 개인적으로 Finetuning할 시에 모델이 커질수록 글이 생성되는 데 시간이 오래 소요되는 점을 알고 있었기에, 빠른 글 생성을 위해 GPT-2 Small 모델을 활용하였습니다.

대신 정확도가 높기를 희망하여 `epoch`를 5로 설정하고 학습을 진행하였습니다.

**4. Generate Text from Ghostwriter**

만약 prefix가 'The demon'으로 주어졌다면, 아래와 같이 'The demon'으로 시작하는 문장들이 출력됩니다. 사용자가 생성되는 글의 Length를 자율적으로 선택하는 방향도 고려하였지만, 안정적인 서버와 로딩 속도를 고려하여 Length는 고정하였습니다.

<img width="1439" alt="_2021-04-14__4 48 24" src="https://user-images.githubusercontent.com/46207836/115824214-933d5880-a442-11eb-853b-733a13cded5d.png">

**5. Re-preprocessing**

2번에서 전처리를 하였음에도 불구하고 미처 발견하지 못한 인덱스(ex.1. , Remark, -)와 일부 특수문자들이 나타났습니다.

```
**Remark II.** Now with this view all empirical use of our faculty of
cognition in the determination of time is in perfect accordance.
**—**Ovid, Metamorphoses. **xiii**
The one relates to the objects of the pure understanding, and is
intended to demonstrate and to render comprehensible the objective
validity of its **_à priori_ conceptions**;
**1.** Mathematical judgements are always synthetical. Hitherto this fact,
though incontestably true and very important in its consequences,
```

세부적으로 다시 전처리를 거쳐 해당 문자들을 제거하였습니다.

추가된 코드는 다음과 같습니다.

```python
# 이전에 놓친 인덱스
elif line.startswith('Introductory'):
    continue
elif line.startswith('The remark:'):
    continue
elif line.startswith('Remark'):
		continue
# 주석을 위한 대괄호 내용 삭제
if line.find('[') > -1:
      if line.find(']') > -1:
        left = line.find('[')
        right = line.find(']')
        new = line[:left-1] + line[right+1:]
        line = new
.
.
.
# 문장 중간에 등장하는 인덱스와 기호 삭제
line = line.replace('§','')
line = line.replace('_', '')
line = line.replace(';', '. ')
line = line.replace('(a)', '')
line = line.replace('(b)', '')
line = line.replace('(c)', '')
line = line.replace('(1)', '')
line = line.replace('(2)', '')
line = line.replace('(3)', '')
line = line.replace('(4)', '')
line = line.replace('1.', '')
line = line.replace('2.', '')
line = line.replace('3.', '')
line = line.replace('4.', '')
line = line.replace('5.', '')
```



**Q. 전처리 과정 중 맞닥뜨리는 문제**

- **소괄호 처리**: 일상적인 대화를 위한 챗봇에서는 괄호를 전처리해야하나, 칸트에 관한 에세이를 작성하는 것이므로 괄호가 큰 문제가 되지 않을 것으로 판단하였습니다. 그래서 문맥을 살리고자 소괄호는 별도 처리하지 않았습니다.
- **문자를 코드로 처리하였음에도 여전히 남아있는 경우**:  눈으로 보기에 동일한 문자로 보이나  코드 상으로 다른 문자인 경우가 있습니다. 이런 경우 함수 내에 문자를 키보드 입력하기보다는 복사-붙여넣기 형식으로 적용해보면 해결됩니다.

**5. More Efficient**

위와 같은 전처리를 거치며 보다 효율적이고, 정확한 문장을 생성하는 모델로 튜닝할 수 있었습니다. 주어진 키워드를 중심으로 '순수이성비판'을 바탕으로 한 글이 생성되어 칸트에 관한 에세이를 써야한다는 부담감을 덜 수 있습니다.

**6. Finally**

이러한 과정을 거쳐 마침내 [Ghostwriter Kant](https://master-kant-dleunji.endpoint.ainize.ai)를 만나보실 수 있습니다.

대신 이 프로그램에서는 length = 200으로 고정되고, 글이 한 번에 출력되어 글의 방향을 입맛대로 설정할 수 없다는 단점이 있습니다.

**만약 글의 방향을 세심하게 다듬고 싶으시다면, 튜닝한 모델과 자동으로 연동된 [TabTab](https://kubecon-tabtab-ainize-team.endpoint.ainize.ai/?modelUrl=https://train-avgw7n5kbmsb7wrip2a8-gpt2-train-teachable-ainize.endpoint.dev.ainize.ai/predictions/gpt-2-en-small-finetune&text=I'm Kant. Please let me know how to get started!)에서 글을 작성해 보세요! TabTab에서는 짧은 문장마다 출력할 수 있으며, 5개의 후보 문장 중에 원하는 문장을 선택할 수 있으므로 글의 방향을 설정하실 수 있습니다.**

![_2021-04-15__4.02.43](/Users/ieunji/Downloads/_2021-04-15__4.02.43.png)



**7. Wrap up**

이제 더 이상 머리를 쥐어짜며 칸트 에세이를 쓰실 필요가 없습니다! 칸트가 여러분 대신 기꺼이 글을 작성해줄 거니까요. 혹여 그가 오랜 기간 무덤에서 지내면서 그의 이론을 잊었을 거라는 우려는 하실 필요 없습니다. 그는 '순수이성비판'을 철저히 복습했으니까요.

여러분도 `Teachable NLP`로 어려운 과목에 대한 레포트를 대신 작성해주는 모델을 쉽게 만들어, API로 글을 생성하실 수 있습니다.