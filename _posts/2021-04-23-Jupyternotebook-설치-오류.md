---
Title : "[python] Jupyter Notebook 설치 오류"
layout : post
date : 2021-04-17
category : Blog
blog : true
author : dleunji
description : Jupyter Notebook
---

#### Summary

- 파이썬 패키지 많이 써야할 때는 번거로워도 `가상환경` 구축하자!
- 충돌현상 발생하면 더 머리 아프다!

---

이제 조금 더 본격적인 데이터 처리를 위해 드.디.어. 내 노트북에 Jupyter notebook을 설치해보려고 하는데, 바로 막혔다..

우선 python2가 디폴트여서 3으로 선언해주는 부분을 오랜만이어서 깜빡했다..

1. `pip3 install --user jupyter notebook`
2. `python3 -m notebook`

나는 더이상 텐서플로우, 케라스와 같은 패키지를 더 이상 깔고 싶지 않아서 주피터를 쓰는 거기 때문에 만약 앞으로 추가 설치가 필요한 경우 docker로 돌릴란다..

공부에 참고하는 자료는 다음과 같다.

나처럼 바짝 자연어 공부를 하실 분들은 참고하시길

https://wikidocs.net/book/2155











