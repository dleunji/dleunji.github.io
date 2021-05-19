---
Title : "Styled Components"
layout : post
date : 2021-05-16
category : React
blog : true
author : dleunji
description : 컴포넌트 스타일링
---

회사의 크롬 확장 프로그램에는 살펴보니 Styled Components 스타일로 컴포넌트가 스타일링 되어있었다. 아무래도 별도의 파일을 생성하지 않아도 되서 아닐까 싶다.

그래서 리액트 중에서 `Styled Components`를 집중적으로 실습하고 공부하였다.

기본적인 부분들은 CSS와 매우 유사하게 작동하므로 특이한 개념만 기록한다.

```scss
import styled from 'styled-components';

const Thing = styled.div.attrs(()=>
    ({tabIndex: 0}))`
    color : blue;

    &:hover {
        color: red; // <Thing> when hovered
    }

    & ~ & {
        background : tomato;
        //<Thing> as a sibling of <Thing>,
        // but maybe not directly next to it
    }

    & + & {
        background: lime;
        //<Thing> next to <Thing>
    }

    &.something {
        background: orange;
        //<Thing> tagged with an additional CSS class ".something"
    }

    .something-else & {
        border: 1px solid;
        // <Thing> inside another elemnt labeled ".something-else"
    }
`
export default Thing;
```

한 파일에 javascript랑 styled-components가 섞여있어서 평소와 다르게 어떤 언어를 블록 언어 처리해야하는지 난감하였다;;;

이러한 점을 Keep in mind하고 추가적인 스타일링 살펴보자!

아까 위 파일에서는 자바스크립트 파일에서 코딩되었다면 아래 코드의 형식은 `scss`이다. 유사해보여도 다른 형식이니 주의하자.

```scss
.TodoListItem {
    padding: 1rem;
    display: flex;
    align-items: center;//세로 중앙 정렬
    &:nth-child(even){
        background: #f8f9fa;
    }
    .checkbox {
        cursor: pointer;
        flex: 1;//차지할 수 있는 영역 모두 차지
        display: flex;
        align-items: center;
        svg {
            //아이콘
            font-size: 1.5rem;
        }
        .text {
            margin-left: 0.5rem;
            flex: 1;//차지할 수 있는 영역 모두 차지
        }
        //체크되었을 때 보여줄 스타일
        &.checked {
            svg{
                color: #22b8cf;
            }
            .text {
                color: #adb5bd;
                text-decoration: line-through;
            }
        }
    }
    .remove {
        display: flex;
        align-items: center;
        font-size: 1.5rem;
        color: #ff6b6b;
        cursor: pointer;
        &:hover{
            color: #ff8787;
        }
    }

    //엘리먼트 사이사이에 테두리를 넣어줌
    & + & {
        border-top: 1px solid #dee2e6;
    }
}
```

**세상에 스타일링이 제일 어려웠어요...**

