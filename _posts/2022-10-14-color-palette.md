---
title: "React에서 라이브러리 없이 컬러 팔레트 구현하기"
excerpt: "색상 선택 기능과, 색상 데이터를 관리하는 법"

categories:
    - react
tags:
    - react

toc: true
toc_sticky: true
---

## 개요

현재 개인 연습용으로 진행중인 영화 카드 프로젝트에서 카드 컴포넌트의 색이 전부 같아 밋밋함을 느껴 컬러 팔레트 기능을 추가하기로 하였다. 라이브러리나 타인의 코드 없이 바닥부터 혼자 개발하는 것을 목표로 하였고, 이틀 간의 씨름 끝에 여러가지 이슈를 해결하면서 원하던 기능을 구현할 수 있었다. **전체 코드는 포스팅의 최하단에 위치시켜 놓았다.**

## 구현할 기능 : 컬러 팔레트

![팔레트](https://user-images.githubusercontent.com/97890886/195639558-86dc83c8-e371-409a-9a83-1dbb7d9941a1.PNG)


- 다양한 색깔의 `컬러 팔레트`컴포넌트를 렌더링한다. 각각의 팔레트는 단일 컴포넌트로 분리한다.
- 카드에도 선택한 컬러 팔레트와 동일한 색상의 `background`가 적용된다.
- 컬러 팔레트를 `클릭`할 시, `border` 옵션을 주어 클릭했다는 것을 사용자에게 보여준다.
- *→ 개발 중 발생한 이슈!* 두 번 클릭할 시 색상 선택이 `취소`되어 기본 색상으로 되돌아가게 한다.

## 완성된 기능

![Honeycam 2022-10-13 23-19-10](https://user-images.githubusercontent.com/97890886/195639696-e853df31-de63-4dd8-9d45-e906cd2c5883.gif)


## 구현하기

### 동적인 컬러 팔레트 만들기

```jsx
const allColors = [
        '#EAECEE',
        '#FCF3CF',
        '#FFEBEE',
        '#FAE1ED',
        '#E9F7EF',
        '#EDE7F6',
        '#F9E4B5',
        '#CAF9B5'
    ];
```

먼저, 색상 데이터 테이블을 `배열`로 만든다.

```jsx
// Create.js

const [color, setColor] = useState('');
```

각 항목의 색상을 데이터로 저장하여 활용하기 위해, 컬러 팔레트 컴포넌트의 상위 컴포넌트에서 `state`로 동적인 상태를 관리한다.

`color`를 동적인 데이터로써 다른 컴포넌트에 사용하려면 필연적으로 계층구조를 따라 아래로 내려가는 단방향 데이터 흐름을 따라야 하기 때문에, `color`은 컬러 팔레트 컴포넌트가 아닌 한 단계 위의 상위 컴포넌트에서 관리하는 것이 좋다.

```jsx
const id = useRef(0);

allColors.map((v,i) => {
		id.current++;
    return (<Color key={id.current} color={v} />);
});
```

`Array.prototype.map`함수를 사용하여 `allColors` 배열을 순회하면서 하위 컴포넌트(개별 컬러 팔레트)에 각각의 색상을 전달한다. key값은 인덱스 대신 useRef를 사용하였다. useRef를 컴포넌트 변수로 활용하는 이유에 대해서는 따로 포스팅하였다. → [useRef를 컴포넌트 내 변수로 사용해보자](https://kanghyeyoon.github.io/react/useRef/)

```jsx
// Color.js

const Color = ( { color} ) => {

    return (
        <button className="Color" style={%raw%}{{backgroundColor:`${color}`}}{%endraw%} />
    );
};

export default Color;
```

하위 컴포넌트인 `Color`은 다음과 같다. `인라인 스타일`과 `템플릿 리터럴`을 활용하여 배경색을 지정해 주었다.

이렇게 하면 정적인 컬러 팔레트는 완성된다.

![팔레트 2](https://user-images.githubusercontent.com/97890886/195639857-16ee4bb3-7b87-45e7-9d69-042b77120c23.PNG)


그러나 여기까지 하면 그냥 예쁜 스케치북과 다를 게 없다. 이제 동적으로 색깔 선택이 가능하도록 해 보자.

```jsx
const ColorPalette = ( {setColor} ) => {
// 생략
};

export default ColorPalette;
```

컬러 팔레트 컴포넌트에서 `state`인 `color`를 변경할 수 있도록 `setColor`를 `props`로 받아준다.

```jsx
return (
        <div className="ColorPalette">
            <h2>컬러 팔레트</h2>
            <div>
                {allColors.map(v => {
										id.current++;
                    return (<button key={id.current} onClick={() => {
                        setColor(v);
                    }}>
                        <Color color={v}/>
                    </button>);
                })}
            </div>
        </div>
    );
```

`onClick` 이벤트 핸들러를 할당하여 각각의 색상 팔레트를 **클릭**했을 시, `color`를 변화시킨다.

이제 상위 컴포넌트에서 `color`를 적당한 곳에 `props`로 넘겨준다면, 실시간으로 색상을 관리할 수 있게 된다. `color`를 활용하여 카드 미리보기 컴포넌트에 `background`로 적용해 보겠다.

![팔레트3](https://user-images.githubusercontent.com/97890886/195639929-3c380d9c-9253-4396-aa01-fa259e07c89a.PNG)


```jsx
// Preview.js

const Preview = ( {color} ) => {

    return (
        <div className="Preview">
            <h2>카드 미리보기</h2>
            <div className="Card" style={%raw%}{{backgroundColor:`${color}`}}{%endraw%}>
							/* contents */
            </div>
        </div>
    );
};

export default Preview;
```

`props`로 `color`을 받아 역시 인라인 스타일을 적용하였다.

### 선택 시 적용될 사용자 친화 디자인 만들기

각각의 컬러 팔레트를 선택(클릭)할 시 `border` 옵션을 주어 색상이 선택되었다는 것을 사용자로 하여금 더 확실하게 알 수 있게 해 보자.

```jsx
const [isSelected, setIsSelected] = useState('');
```

선택된 팔레트의 정보를 저장할 `state`를 추가한다.

```jsx
return (
        <div className="ColorPalette">
            <h2>컬러 팔레트</h2>
            <div className="AllColor">
                {allColors.map((v,i) => {
                    id.current++;
                    return (<button key={id.current} onClick={() => {
                        setColor(v);
                        setIsSelected(i);
                    }}>
                        <Color id={i} color={v} isSelected={isSelected} />
                    </button>);
                })}
            </div>
        </div>
    );
```

`onClick` 이벤트 핸들러에 `setIsSelected`를 추가하여 선택된 Color 컴포넌트의 정보를 실시간으로 관리할 수 있게 한다.

```jsx
// Color.js

const Color = ( {id, color, isSelected} ) => {

    return (
        <div className={`Color${isSelected === id ? 'Selected' : ''}`} style={%raw%}{{backgroundColor:`${color}`}}{%endraw%} />
    );
};

export default Color;
```

개별 컬러 팔레트 컴포넌트에서는 **자신이 선택된 색상인지에 대한 여부**를 `isSelected`로 받게 된다.

삼항연산자를 통한 `조건부 렌더링`을 통해, `isSelected`가 true인 경우에만 다른 스타일을 적용하게끔 해준다.

```css
.ColorSelected {
		/* 생략 */
    border: 3px solid #D2D2CB;
}
```

`border`을 적용해준다.

그러나 여기까지만 하면, 이미 선택한 색상을 다시 한 번 클릭했을 때 **취소가 되지 않아** 불편하다. 보통 일반적인 선택 버튼 기능의 경우, 선택한 버튼을 다시 클릭하면 초깃값으로 돌아간다는 점을 감안하여 추가적인 코드를 작성해 보겠다.

```jsx
return (
        <div className="ColorPalette">
            <h2>컬러 팔레트</h2>
            <div className="AllColor">
                {allColors.map((v,i) => {
                    id.current++;
                    return (<button key={id.current} onClick={() => {
                        setColor(v);
                        setIsSelected(i);
                        if (isSelected === i) {        // <----- !!!
                            setColor('#eeeee4');
                            setIsSelected('');   
                        }
                    }}>
                        <Color id={i} color={v} isSelected={isSelected} />
                    </button>);
                })}
            </div>
        </div>
    );
```

if 조건문을 추가하여 선택한 컴포넌트가 **바로 전에 선택한 컴포넌트와 동일한 경우** `isSelected`를 초기화시켜 1. `color` 를 기본 색상으로 초기화하고 2. `isSelected`를 빈 문자열로 초기화한다.

![팔레트 ㅋ](https://user-images.githubusercontent.com/97890886/195641150-d72dbe97-e66a-4895-99a2-05b55b0feb42.PNG)

처음 보았던 최종 결과물과 같은 컬러 팔레트가 완성되었다!

## 전체 코드

> Create.js
> 

```jsx
// Create.js

import { useState } from 'react';
import ColorPalette from '../components/ColorPalette';
import Preview from '../components/Preview';

const Create = () => {

    const [contents, setContents] = useState();
    const [color, setColor] = useState('');

    const priviewInfo = {
        name,
        star,
        contents,
        color
    };

    return (
        <div className="Create">
            <aside>
                <Preview {...priviewInfo} />
                <ColorPalette setColor={setColor} />
            </aside>
            <article>
								/* 생략 */
            </article>
        </div>
    );
};

export default Create;
```

> ColorPalette.js
> 

```jsx
// ColorPalette.js

import { useRef, useState } from "react";
import Color from "./Color";

const ColorPalette = ( {setColor} ) => {
    const [isSelected, setIsSelected] = useState('');

    const id = useRef(0);

    const allColors = [
        '#EAECEE',
        '#FCF3CF',
        '#FFEBEE',
        '#FAE1ED',
        '#E9F7EF',
        '#EDE7F6',
        '#F9E4B5',
        '#CAF9B5'
    ];

    return (
        <div className="ColorPalette">
            <h2>컬러 팔레트</h2>
            <div className="AllColor">
                {allColors.map((v,i) => {
                    id.current++;
                    return (<button key={id.current} onClick={() => {
                        setColor(v);
                        setIsSelected(i);
                        if (isSelected === i) {
                            setColor('#eeeee4');
                            setIsSelected('');   
                        }
                    }}>
                        <Color id={i} color={v} isSelected={isSelected} />
                    </button>);
                })}
            </div>
        </div>
    );
};

export default ColorPalette;
```

> Color.js
> 

```jsx
// Color.js

const Color = ( {id, color, isSelected} ) => {

    console.log(isSelected);

    return (
        <div className={`Color${isSelected === id ? 'Selected' : ''}`} style={%raw%}{{backgroundColor:`${color}`}}{%endraw%} />
    );
};

export default Color;
```

> Preview.js
> 

```jsx
const Preview = ( {color} ) => {

    return (
        <div className="Preview">
            <h2>카드 미리보기</h2>
            <div className="Card" style={%raw%}{{backgroundColor:`${color}`}}{%endraw%}>
								/* 생략 */
            </div>
        </div>
    );
};

export default Preview;
```

## 기능 구현보다 중요한 것은 데이터의 흐름

> 어떤 컴포넌트가 state를 변경하거나 **소유할 것인가?**
> 

사실 컬러 팔레트는 기능 자체만 보자면 전혀 어려운 난이도가 아니다. 오히려 단순한 라디오 버튼에 가깝다. 그러나 이 기능을 구현하면서, 나는 단순히 로직을 코딩하는 것을 넘어 **React의 본질**에 대해 생각해보게 되었다.

React는 항상 컴포넌트 계층구조를 따라 아래로 내려가는 단방향 데이터 흐름을 따른다. 그리고 어떤 컴포넌트가 어떤 state를 가져야 하는 지 바로 결정하는 것은 매우 힘들다. 이 문제로 며칠을 고민하던 도중, React의 공식 문서의 마지막 문단의 `React스럽게 생각하기`라는 내용에서 많은 아이디어를 얻을 수 있었다. 

- state를 기반으로 렌더링하는 모든 컴포넌트를 찾아라!
- 공통 소유 컴포넌트를 찾아라!
- **공통, 혹은 더 상위에 있는 컴포넌트가 state를 가져야 한다.**
- state를 소유할 적절한 컴포넌트를 찾지 못하였다면, 상위 계층에 state를 소유하는 컴포넌트를 하나 만들어서 공통 소유 컴포넌트를 상속시켜라.

컬러 팔레트에서 색상 `state`를 직접 관리하는 것이 아닌, 한 단계 위의 상위 컴포넌트에서 관리하게 함으로써 보다 쉽게 데이터를 활용할 수 있었다.

결론적으로 어떤 기능을 구현할 때에는 컴포넌트 고유의 로직을 짜는 행위 뿐만이 아닌 그 컴포넌트의 `state`를 어떻게 활용할지에 대한 고찰이 요구되는 것이다. 앞으로도 더욱 React스럽게 데이터를 다루고 싶다는 생각을 했다.

## 참고
[thinking-in-react](https://ko.reactjs.org/docs/thinking-in-react.html)