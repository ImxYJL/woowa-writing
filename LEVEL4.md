이 글은 React를 사용하는 프론트엔드 개발 초보를 위해 React와 TypeScript를 활용한 공통 컴포넌트를 만드는 법을 안내합니다.
여기서 다루는 내용은 기초적인 수준으로, 공통 컴포넌트의 지원 범위를 설정하는 것이나 웹 접근성 심화처럼 깊은 주제 대신 '재사용 가능한 공통 컴포넌트'를 만드는 방법을 다룹니다.
React를 써 보긴 했지만 공통 컴포넌트를 어떻게 만들어야 할지 막막한 분들에게 추천드립니다.

# **컴포넌트는 무엇일까**

## **통상적인 의미의 컴포넌트**

컴포넌트는 재사용할 수 있는 코드 블럭입니다. 꼭 컴포넌트라는 단어를 쓰지 않아도 이렇게 재사용가능한 코드 조각이라는 개념은 프로그래밍 전반에서 통용되는 중요한 개념입니다.

**리액트 공식 문서의 컴포넌트**

그렇다면 FE 개발, 나아가 리액트에서의 컴포넌트는 무엇일까요?
사람에 따라 정의하는 방식이 조금씩 다르지만 여기서는 컴포넌트란 화면에 그려질 무언가를 리턴하는 JavaScript 함수라고 하겠습니다.

# **공통 컴포넌트란?**

공통 컴포넌트는 컴포넌트의 의미 중 ‘재사용성’에 집중한 컴포넌트입니다.

## **특화된 컴포넌트**

앞선 통상적인 의미의 컴포넌트에서 컴포넌트는 재사용 가능하다고 했으나, 어떤 컴포넌트는 비교적 특정 기능에 특화되어 재사용성이 떨어지기도 합니다.
예를 들어 아래와 같은 컴포넌트는 비밀번호 입력에 특화된 상태입니다. 

```tsx
interface PasswordInputProps {
  password: string;
  setPassword: (value: string) => void;
}

const PasswordInput = ({ password, setPassword }: PasswordInputProps) => {
  const handleChange = (event: ChangeEvent<HTMLInputElement>) => {
    setPassword(event.target.value);
  };

  return (
    <input 
      id="password-input" 
      value={password} 
      type="password" 
      onChange={handleChange} 
    />
  );
};

```

비밀번호 외의 다른 정보(사용자 이름, 주소 등)를 입력받는 `input`이 필요할 때 이 `PasswordInput`을 재활용할 수 없기 때문입니다. 리턴되는 input의 `id`가 `password-input`이고, `type` 또한 `password`라서 비밀번호가 아닌 일반 텍스트 형태로 입력을 받고 싶다면 또다른 `input` 컴포넌트를 만들어야 합니다.

## **공통 컴포넌트**

반면 공통 컴포넌트는 컴포넌트의 재사용성에 집중해 다양한 상황에서 재활용할 수 있는 컴포넌트입니다.
아래와 같은 `input` 컴포넌트는 `props`를 이용해 `input` 컴포넌트 외부에서 상황별로 커스텀할 수 있는, 즉 더 자유도가 높은 컴포넌트입니다.

```tsx
interface InputProps {
  type: React.InputHTMLAttributes<HTMLInputElement>['type'];
  value: string;
  onChange: (event: React.ChangeEvent<HTMLInputElement>) => void;
  id?: string;
}

const Input = ({ id, value, onChange, type }: InputProps) => {
  return (
    <input
      id={id}
      value={value}
      type={type}
      onChange={onChange}
    />
  );
};
```

이전 코드와의 차이점이 무엇일까요?
이전 코드에서는 `input`의 `id`와 `type`을 `Input` 컴포넌트 안에서 고정해버렸는데, 이제는 외부에서 자유롭게 지정할 수 있습니다.
이를테면 `id`를 `username-input` 등으로 설정하고 `type`도 `text`로 넘겨주면 사용자의 이름을 입력하는 `Input` 컴포넌트로 만들 수 있고, 아까처럼 `type`이 `password`인 비밀번호 입력 `Input`으로도 활용할 수 있습니다.
이런 방식으로 보다 다양한 상황에서의 재활용 가능성을 고려한 컴포넌트가 공통 컴포넌트입니다. 

# **공통 컴포넌트에 타입 적용하기**

타입스크립트를 사용하는 경우, `props`에 타입을 지정하지 않으면 타입이 `any`로 인식되므로 설정에 따라 타입 에러가 발생합니다. 앞선 예시들에서 인터페이스로 타입을 지정해 준 이유도 그 때문입니다. 이번에는 공통 컴포넌트에 타입을 어떻게 지정하는지를 알아보겠습니다.

## **사용할 타입만 명시하기**

첫 번째 방법은 보수적으로 사용할 속성들만 명시해주는 방법입니다. 앞서 보여드린 예시에서도 이 방법을 사용했습니다.

```tsx
interface InputProps {
  type: React.InputHTMLAttributes<HTMLInputElement>['type'];
  value: string;
  onChange: (event: React.ChangeEvent<HTMLInputElement>) => void;
  id?: string;
}

const Input = ({ id, value, onChange, type }: InputProps) => {
  return (
    <input
      id={id}
      value={value}
      type={type}
      onChange={onChange}
    />
  );
};
```

`Input` 컴포넌트의 `props`로 올 수 있는 값을 `InputProps` 인터페이스로 명시했으므로, 이 인터페이스에 명시되지 않은 속성을 `props`로 넘겨준다면 타입 에러가 납니다.

## **해당 태그에서 사용할 수 있는 모든 타입 받아오기**

두 번째 방법은 해당 태그에서 지원하는 모든 속성들을 한 번에 가져올 수 있는 방법입니다.
바로 `React.InputHTMLAttributes<HTMLInputElement>` 타입을 사용하면 됩니다.

```tsx
const Input = ({
  id,
  value,
  onChange,
  type,
}: React.InputHTMLAttributes<HTMLInputElement>) => {
  return (
    <Input
      id={id}
      value={value}
      type={type}
      onChange={onChange}
    />
  );
};
```

`React.InputHTMLAttributes<HTMLInputElement>`는 React에서 `input` 요소에 적용할 수 있는 속성들을 타입으로 정의한 것입니다. 이 타입은 `input` 요소의 모든 표준 HTML 속성을 포함하고 있습니다.

현재 `props`로 받아오고 있는 모든 속성은 `React.InputHTMLAttributes<HTMLInputElement>`에 포함된 속성들이므로 타입 에러가 나지 않고, 위에서처럼 별도의 인터페이스 등으로 이 속성들을 지정해줄 필요도 없습니다.

또한 `rest props`와 결합해 코드를 더 간편하게 작성할 수도 있습니다.
```tsx
const Input = ({ ...rest }: React.InputHTMLAttributes<HTMLInputElement>) => {
  return <input {...rest} />;
};
```

이외에도 `React.InputHTMLAttributes<HTMLInputElement>`에서 특정 속성만을 제외하고 싶다면 `Omit`을 통해 제거하는 등 다양한 타입 커스텀을 할 수 있습니다.
이렇게만 보면 타이핑도 적고 유연한 후자가 더 편리해보이지만, 공통 컴포넌트의 자유도를 어디까지 둘 것인지에 따라 어떤 방법을 선택할지가 결정됩니다. 높은 자유도가 항상 좋기만 한 건 아니니까요. 
이와 관련된 내용은 본 글에서 다루는 범위를 넘기 때문에 [이 내용을 다루는 글](https://fe-developers.kakaoent.com/2024/240116-common-component/)을 소개하고 넘어가겠습니다. 지금은 공통 컴포넌트에 typescript를 도입했을 때 어떻게 타입을 지정할 수 있는지만 알고 넘어가도 좋을 것 같습니다.

---

# 공통 Checkbox 컴포넌트 만들기

이제 본격적으로 `Checkbox` 기능을 공통 컴포넌트로 만들어 보겠습니다.
아시다시피 `Checkbox`는 아래처럼 어떤 것을 선택할 수 있는 기능을 제공하는 컴포넌트입니다.

## 기능 상세
이번에 만들 Checkbox 컴포넌트는 다음과 같은 요구사항을 만족하는 컴포넌트입니다.
- 같은 코드를 여러 번 사용하지 않도록 편의성을 제공해야 한다. (재사용 가능한 컴포넌트)
- 기본적으로 제공하는 스타일이 있지만 외부에서 크기 등을 변경할 수 있어야 한다.

## 태그 결정하기: 시맨틱 마크업

### button과 input

많은 요구사항들이 있지만, 가장 먼저 해야 할 일은 역시 컴포넌트에서 사용할 태그를 결정하는 일입니다.
`Checkbox` 컴포넌트는 어떤 식으로 마크업해야 할까요?

사용자와 상호작용하는 버튼이니까 `button` 태그가 가장 먼저 떠오릅니다.
실제로 `button` 태그를 이용해서 `Checkbox` 기능을 만들 수 있습니다.

```tsx
interface CheckboxProps {
  id: string;
  isChecked: boolean;
  onChange: (event: ChangeEvent<HTMLInputElement>) => void;
}

const Checkbox = ({ id, isChecked, onChange }: CheckboxProps) => {
  return (
    <button id={id} type="button" onChange={onChange }>
      {isChecked ? <img src={checked} /> : <img src={unchecked} />}
    </button>
  );
};

export default Checkbox;
```

CSS 작업 대신 사용할 `Checkbox` 이미지도 `button` 태그의 자식으로 두면 쉽게 `Checkbox` 컴포넌트를 구현할 수 있습니다. 
이 체크박스는 잘 동작하지만(단 자연스럽게 보이려면 별도의 스타일 작업이 필요합니다), 시맨틱 마크업 관점에서 문제가 있습니다.

웹 페이지를 만들 때는 잘 동작하는 기능도 중요하지만 의미론적인 요소도 함께 챙겨 줘야 하기 때문입니다.
시맨틱 마크업을 통해 의미를 살려주는 작업은 웹 접근성 측면에서도 중요하지만 다른 개발자가 코드를 보고 이 컴포넌트가 무슨 역할을 하는지를 쉽게 알아볼 수 있다는 면에서도 중요합니다. 

시맨틱 마크업에서 `Checkbox`는 `button` 태그가 아닌 `input` 태그에 `type="checkbox"`를 기입하는 방식으로 만들어야 합니다. 
위 코드에서 태그를 `input`으로 수정해보겠습니다.

```tsx
const Checkbox = ({ id, isChecked, onChange }: CheckboxProps) => {
  return (
    <input id={id} type="checkbox" onChange={onChange }>
      {isChecked ? <img src={checked} /> : <img src={unchecked} />}
    </input>
  );
};
```

그런데 이 코드에서는 에러가 발생합니다.

> Uncaught Error: input is a void element tag and must neither have `children` nor use `dangerouslySetInnerHTML`.


즉 `input` 태그는 자식 요소를 가질 수 없다는 뜻입니다. 
`input` 태그는 별도의 `src` 속성을 지원하지 않는데, `Checkbox` 이미지를 어떻게 사용하면 좋을까요?

### input과 label

이럴 때는 `label`을 사용해서 `input`과 `img`를 연결해주면 됩니다. 
좁은 의미의 `label`은 `input` 에 대한 설명을 제공하는 태그지만, 넓게 봤을 때는 `for`(리액트에서는 `htmlFor`)를 통해 `input`과 다른 요소들을 이어 `input`과 관련된 정보를 명시적으로 전달하는 역할을 합니다.
적용하는 방법은 두 가지인데, 둘 다 간단합니다.

#### 직접적으로 label 명시하기

1. `input`을 감싸는 상위 태그를 만듭니다. 여기서는 `div`를 사용했습니다.
2. 상위 태그 안에 `input`과 `label`을 넣습니다. (즉 둘은 서로 형제 관계)
    이때 `label`의 `htmlFor`값은 `input`의 `id`값입니다.
3. `label`의 자식으로 `img`를 넣어줍니다.

```tsx
const Checkbox = ({ id, isChecked, onChange }: CheckboxProps) => {
  return (
    <div>
      <input id={id} type="checkbox" checked={isChecked} onChange={onChange} />
      <label htmlFor={id}>
        <img src={isChecked ? checked : unchecked} />
      </label>
    </div>
  );
};
```

#### 간접적인 label 명시

`input`을 `label`의 자식 요소로 넣는 방법으로도 암시적으로 둘의 관계를 표현할 수 있습니다. 이 방법에서는 `label`에 `htmlFor`를 사용할 필요가 없습니다.

```tsx
return (
    <S.CheckboxContainer>
      <S.CheckboxLabel>
        <img src={isChecked ? checked : unchecked} alt="checkbox" />
        <input
          id={id}
          type="checkbox"
          checked={isChecked}
          onChange={onChange}
        />
      </S.CheckboxLabel>
    </S.CheckboxContainer>
  );
```

이렇게 코드를 작성하면 위와 같은 에러를 방지할 수 있습니다.
두 방법은 취향 차이라고 생각하는데, 이 글에서는 첫 번째 방법을 사용하도록 하겠습니다.

이제 스타일을 입혀볼 차례입니다. 
자잘한 스타일 조정들이 많은데, 가장 중요한 건 html에서 기본으로 보여주는 `input` 대신 체크박스 이미지를 사용해야 하니 `input`의 `display`를 `none`으로 설정해 화면에서 없애주는 것입니다. (라이브러리는 `@emotion/styled`를 사용했습니다.)

```tsx
export const CheckboxContainer = styled.div`
  width: 2.7rem;
  height: 2.7rem;
  padding: 0;
  background-color: transparent;
  border: none;  
`;

export const Checkbox = styled.input`
	/* 주의: 키보드 접근성 이슈가 있습니다. */
  position: absolute;
  clip: rect(1px, 1px, 1px, 1px);
  clip-path: inset(100%);
`;

export const CheckboxLabel = styled.label`
  cursor: pointer;
  display: inline-block;
  width: 100%;
  height: 100%;

  img {
    width: 100%;
    height: 100%;
  }
`;
```

```tsx
const Checkbox = ({ id, isChecked, onChange }: CheckboxProps) => {
  return (
    <S.CheckboxContainer>
      <input id={id} type="checkbox" checked={isChecked} onChange={onChange} />
      <S.CheckboxLabel htmlFor={id}>
        <img src={isChecked ? checked : unchecked} />
      </S.CheckboxLabel>
    </S.CheckboxContainer>
  );
};

```

스타일을 입히니 이제 기본 `input` 요소는 사라지고 우리가 사용할 체크박스 이미지만 남게 되었습니다.
  
## 속성 열어두기

이제 스타일도 유연하게 적용할 수 있는 `Checkbox`를 만들어봅시다.
앞서 적용한 스타일을 보면 `Checkbox`의 사이즈가 `2.7rem`으로 고정되어 있습니다.

```tsx
export const CheckboxContainer = styled.div`
  width: 2.7rem;
  height: 2.7rem;
  ...
`;
```

`Checkbox`는 명색이 공통 컴포넌트인데 어디서나 고정된 크기로 사용해야 한다면 불편하겠죠? 
기본적으로 설정된 크기로 불러오되 원하는 경우 사이즈를 직접 명시할 수 있게 수정해보겠습니다.

```tsx
export interface CheckboxStyleProps {
  $style?: React.CSSProperties;
}
```

`CheckboxStyleProps` 라는 새로운 인터페이스를 만들었습니다. 굳이 새로 만든 이유는 스타일링을 위한 속성과 일반 속성을 구별하기 위함입니다. 
`$style`으로  `React.CSSProperties`를 받아와서 사용처에서 어떤 CSS 코드라도 사용할 수 있게 열어뒀습니다. 
(여기서는 모든 스타일을 전부 열었지만, 원하는 스타일 타입만 받아올 수도 있습니다. 계속 언급하고 있긴 하지만 이런 사소한 방식의 차이는 상황에 맞게 활용하시면 됩니다.)

이때 `props`에 `$`를 사용했는데요, 이는 DOM에 불필요한 속성(React에서만 사용하는 `props` 등)을 전달하지 않기 위해서입니다. 간략하게는 유효 속성만 전달하기 위한 구분자라고 생각하면 됩니다.
이제 상속을 통해 `CheckboxStyleProps`와 `CheckboxProps`를 연결해줍니다.

```tsx
interface CheckboxProps extends CheckboxStyleProps {
  id: string;
  isChecked: boolean;
  onChange: (event: ChangeEvent<HTMLInputElement>) => void;
}
```

그리고 `$style` 속성을 `props`로 받아 사이즈를 결정하는 컴포넌트에 전달해줍니다.

```tsx
const Checkbox = ({ id, isChecked, onChange, $style }: CheckboxProps) => {
  return (
    <S.CheckboxContainer $style={$style}>
      ...
    </S.CheckboxContainer>
  );
};
```

```tsx
export const CheckboxContainer = styled.div<CheckboxStyleProps>`
  width: 2.7rem;
  height: 2.7rem;
  padding: 0;
  background-color: transparent;
  border: none;

  ${({ $style }) => $style && { ...$style }};
`;
```

이렇게 스타일을 열어두면 `Checkbox`를 사용하는 쪽에서 원하는 크기를 인라인으로 넘길 수 있습니다.

```
<Checkbox
  id="1"
  isChecked={isChecked}
  onChange={handleChangeCheckbox}
  $style={{ width: "6rem", height: "6rem" }}
/>
```

이제 재사용 가능한 공통 `Checkbox` 컴포넌트를 완성했습니다 👍

## 참고한 글
- [input](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input)
- [label](https://developer.mozilla.org/ko/docs/Web/HTML/Element/label)
- [더 가치 있는 공통 컴포넌트 만들기](https://fe-developers.kakaoent.com/2024/240116-common-component/)
- [웹 접근성](https://www.w3.org/WAI/fundamentals/accessibility-intro/)
