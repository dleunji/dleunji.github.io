---
Title : "Ghostwriter Kant Data Preprocess"
layout : post
date : 2021-04-23
category : Internship
blog : true
author : dleunji
description : Network Layer
---

### Ghostwriter Data preprocess

Teachable-NLP는 코드를 따로 작성할 필요없이 **하나의 텍스트 파일**만으로 GPT-2을 튜닝할 수 있는 프로그램입니다.

이를 활용하여 철학과 학부 시절 자주하던 상상을 실현하고자 칸트의 '순수이성비판'을 활용하여 칸트식의 문장을 생성할 수 있는 모델을 생성하기를 마음 먹었습니다. 단순히 문장을 생성하는 것보다는 스토리라인을 덧붙여 마치 칸트가 대필작가(Ghostwriter)가 되어 학생들 대신 글을 써주는 방식으로 프로그램, [Ghostwriter Kant](https://ainize-kant-dleunji.endpoint.ainize.ai)를 구상하였습니다.

1. **Acquiring the data**

칸트의 가장 대표적인 저서 중 '순수이성비판(The Critique of Pure Reason) 은 [Project Gutenberg](https://www.gutenberg.org)에서 무료로 txt 형식으로 구할 수 있었습니다. 원문은 독일어로 작성되었으나, 원활한 학습과 생성된 글의 활용성을 고려하여 영어 번역본을 데이터로 선정하였습니다. 그리고 이는 고전 도서로, Public Domain License 상태였습니다. 이로써 학습에 활용하여도 저작권적인 문제가 발생하지 않음을 확인하였습니다.

![kant_gut](https://user-images.githubusercontent.com/46207836/115824821-866d3480-a443-11eb-8be7-a583724c8d43.png)

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

이러한 과정을 거쳐 마침내 [Ghostwriter Kant](https://ainize-kant-dleunji.endpoint.ainize.ai)를 만나보실 수 있습니다.

대신 이 프로그램에서는 length = 200으로 고정되고, 글이 한 번에 출력되어 글의 방향을 입맛대로 설정할 수 없다는 단점이 있습니다.

**만약 글의 방향을 세심하게 다듬고 싶으시다면, 튜닝한 모델과 자동으로 연동된 [TabTab](https://kubecon-tabtab-ainize-team.endpoint.ainize.ai/?modelUrl=https://train-avgw7n5kbmsb7wrip2a8-gpt2-train-teachable-ainize.endpoint.dev.ainize.ai/predictions/gpt-2-en-small-finetune&text=I'm Kant. Please let me know how to get started!)에서 글을 작성해 보세요! TabTab에서는 짧은 문장마다 출력할 수 있으며, 5개의 후보 문장 중에 원하는 문장을 선택할 수 있으므로 글의 방향을 설정하실 수 있습니다.**

![tabtab화면](https://user-images.githubusercontent.com/46207836/115824889-a3a20300-a443-11eb-94a6-5a42e414e417.png)



**7. Wrap up**

이제 더 이상 머리를 쥐어짜며 칸트 에세이를 쓰실 필요가 없습니다! 칸트가 여러분 대신 기꺼이 글을 작성해줄 거니까요. 혹여 그가 오랜 기간 무덤에서 지내면서 그의 이론을 잊었을 거라는 우려는 하실 필요 없습니다. 그는 '순수이성비판'을 철저히 복습했으니까요.

여러분도 `Teachable NLP`로 어려운 과목에 대한 레포트를 대신 작성해주는 모델을 쉽게 만들어, API로 글을 생성하실 수 있습니다.



[English ver.]

**Teachable-NLP** is a GPT-2 Finetuning program with a text(.txt) file without writing NLP codes. As soon as I recognized it, I planned to make a model creating Kantian sentences.

I dreamed a someone who writes an Philosophy essay in place of me when I majored in Philophy. So I came up with the idea that I tranined GPT-2 with 'The Critique of Pure Reason' of Kant and Kant becomes Ghostwriter and does writes Kantian essay in place of students.

1. **Acquiring the data**

I acquired the text(.txt) file of 'The Critique of Pure Reason' from [Project Gutenberg](https://www.gutenberg.org). The original text is in Deutsch, but Teachable NLP works only in English. So I used the English version.

Moreover, I had to check the copyright status of text file. It published hundreds years ago, so the copyright was Public domain. It makes me free to copyright issue even if I edit the text and use the text to machine learning.

![kant_gut](https://user-images.githubusercontent.com/46207836/115824821-866d3480-a443-11eb-8be7-a583724c8d43.png)

**2. Preprocessing the data**

There were unwanted data in the text file (e.g. headings like SECOND', 'I.').

It might cause `garbage out`, so I removed them.

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

Also the diagrams which explains the Kantian theories will disrupt the context with the irregular space to Teachable-NLP. After skmming the text, I found all the diagrams and annotations started with spaces and other regular sentences didn't. I removed the unnecessary data by using the pattern.

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

It was simple data preprocessing with `Python` .

With a `for loop`, I checked every line ( `f.readlines()`)and add only valid data to the new data file. For that, I examined the sentence carefully and tried to find out patterns.

The text in front of PREFACE consists of publication and irrelevant with Kant. So I ignored the data before a sentence starting with PREFACE. And then, I filtered out the headings and gap in the intervals.

As a result, I found out the pattern of headings (e.g. Section, Chaper, Introduction) and the terminologies of Logic, Roman numerals. And I tried to make use of the contents in the parentheses by substring the text and kept in context.

```python
file_name = "The Critique of Pure Reason.txt"
f = open(file_name, "rt", encoding='utf-8')
file = f.readlines()
f.close()
sentences = []
start = 0
for line in file: 
  if line.startswith('PREFACE'): 
	# In front of PREFACE, no values
	# var start is used to flag
    start = 1
    continue
  elif start != 1:
    continue
  elif line.startswith('End'):
	# And vice versa, behind END, no values
	# So break the loop
    break
  elif line.startswith(' '):
	# Remove the diagrams and annotations
    continue
	# Remove headings
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
		# Remove only the Symbols and keep the valid long titles
    line = line[line.find('.') + 2:]
  elif line.startswith('-'):
    continue
  elif line.startswith('§'):
    continue
	# Remove CRLF
  elif line == "\\n":
    continue
  elif line.startswith('('):
    line = line[4:]
  sentences.append(line)
```

Finally I got a 1.2MB data with simple sentences. Some Deutsch proper noun and colons are still there. If I remove them, however, it doesn't make sense. So I kept them given the rarely small amounts.

[preprocess1.txt](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/45f457ef-8708-4f73-9a74-a25f63e19959/preprocess1.txt)

This is the introduction of data after preprocess.

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

I uploaded the data to Teachable-NLP and set the option to finetune. I've already finetuned the model by myself and noticed it takes longer time to finetune larger model. I focused on reducing the loading, so I chose `small model`. But I chose `epoch 5` to ensure least at least accuracy.

**4. Generate Text from Ghostwriter**

The user should give him a prefix of essay. If prefix is 'The demon', The sentences prints out as below. I considered the user can control the length of essay, however, I fixed the length to 200 for GPU utilzation rate.

<img width="1439" alt="_2021-04-14__4 48 24" src="https://user-images.githubusercontent.com/46207836/115824214-933d5880-a442-11eb-853b-733a13cded5d.png">

**5. Re-preprocessing**

Even though I've already preprocessed in No.2, some symbols and headings appear because I missed them (e.g.1. , Remark, -) at that time.

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

So I preprocess data again and clearly remove them. For that, I added the code as below.

```python
# missed headings
elif line.startswith('Introductory'):
    continue
elif line.startswith('The remark:'):
    continue
elif line.startswith('Remark'):
		continue
# remove the bracket
if line.find('[') > -1:
      if line.find(']') > -1:
        left = line.find('[')
        right = line.find(']')
        new = line[:left-1] + line[right+1:]
        line = new
.
.
.
# remove index and symbols in the middle of sentence
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

**Q. Problems**

- **Parentheses** : It is ought to be removed for routine dialogue. But for essay, the parentheses doesn't matter.
- **Some texts are stiil alive** : There are some texts that seems same, but they are actually different in code. In case of that, you can copy & paste the ambigous text to the preprocessing code, not just type.

**5. More Efficient**

Over the double preprocessing, the GPT-2 model became fine-tuned to more accurate. Finally with the prefix, the Kantian essay is created automatically based on 'The Critique of Pure Reason' and it relieves the students who have to write the long essay.

**6. Finally**

**The program '[Ghostwriter Kant](https://ainize-kant-dleunji.endpoint.ainize.ai)' fixed the length of essay and the text is printed only at once. If you want to elaborate the essay, write in [TabTab](https://kubecon-tabtab-ainize-team.endpoint.ainize.ai/?modelUrl=https://train-avgw7n5kbmsb7wrip2a8-gpt2-train-teachable-ainize.endpoint.dev.ainize.ai/predictions/gpt-2-en-small-finetune&text=I'm Kant. Please let me know how to get started!) that is linked to the Fine-tuned model! In [TabTab](https://kubecon-tabtab-ainize-team.endpoint.ainize.ai/?modelUrl=https://train-avgw7n5kbmsb7wrip2a8-gpt2-train-teachable-ainize.endpoint.dev.ainize.ai/predictions/gpt-2-en-small-finetune&text=I'm Kant. Please let me know how to get started!), you can print out short sentence, and select a sentence in 5 candidate sentences.**

![tabtab화면](https://user-images.githubusercontent.com/46207836/115824889-a3a20300-a443-11eb-94a6-5a42e414e417.png)

**7. Wrap up**

Now you don't have to rack your brain to write Kantian essay for Philosophical assignments. Ghostwriter Kant would write it in place of you. Perhaps you might to worry he forgot his theory while he lay in the tomb. But you don't have to worry about that! Because he reviewed 'The Critique of Pure Reason' thoroughly!

You can also easily create a model that writes a essay on difficult subjects with `Teachable NLP` and generate articles with Model APIs.