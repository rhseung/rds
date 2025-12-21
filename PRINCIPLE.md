# Makers' Principle
## Rhseung's Design System (RDS)

RDS는 그냥 컴포넌트 모음이 아니에요.
RDS는 **선언적인 UI 문법(Declarative UI, 선언적 UI)**이에요.

RDS는 이렇게 작동해요.

- UI의 **구조(structure, 구조)**는 개발자가 선언해요.
- UI의 **디테일(detail, 디테일)**과 **맞물림(adjacency, 인접 처리)**은 시스템이 책임져요.
- 어떤 브랜드 색상(brand color, 브랜드 색)이어도 안정적으로 동작해요. (Brand-agnostic, 브랜드 독립적)
- 타입 안정성(type safety, 타입 안전성)을 기본으로 가져요.

이 문서는 RDS를 설계하고 구현할 때 따라야 하는 **기준(standard, 기준)**이에요.

## 1. 선언적 UI를 기본으로 써요 (Declarative by Default)

RDS에서는 “어떻게 구현하느냐”를 코드에 최대한 적지 않아요.
대신 “무엇을 만들고 싶은지”만 또렷하게 적어요.

- 개발자는 **무엇을 보여줄지**만 선언해요.
- **어떻게 보일지**는 시스템이 책임져요.

예시:

```tsx
<ButtonGroup>
	<Button />
	<Button />
	<ButtonGroup.Separator />
	<Button />
</ButtonGroup>
```

이 코드에서 `radius`, `border`, `spacing` 같은 건 개발자가 신경 쓰지 않아도 돼요.
버튼끼리 맞닿는 부분의 디테일은 시스템이 알아서 처리해요.

## 2. `props`를 나열해서 UI를 만들지 않아요 (No Prop-Dumping)

RDS에서는 특정 UI를 만들기 위해 `props`를 계속 덧붙이는 방식을 지양해요.
이 방식은 시간이 지나면 API가 무거워지고, 사용하기 어려워져요.

핵심은 이거예요.

- 구조는 `children`으로 표현해요.
- 구현 디테일은 `props`로 노출하지 않아요.

### 2.1 구현 디테일 `props`는 지양해요 (Implementation-detail `props`)

**Implementation-detail `props`(구현 디테일 속성)**은 컴포넌트 내부 레이아웃/위치/스타일 같은 구체적인 구현을 직접 조절하게 만들어요.

예를 들면 이런 것들이에요.

- `startIcon`, `endIcon`
- `leftAdornment`, `rightAddon`
- `hasIcon`, `withX` 같은 shortcut `props`

이런 `props`는 선언적 UI를 깨요.
컴포넌트가 “무엇인지”보다 “어떻게 배치되는지”를 강요하기 때문이에요.

### 2.2 구조는 `children`으로 표현해요 (Structural `children`)

RDS에서는 `children`을 **콘텐츠(content, 내용)**가 아니라 **구조(structure, 구조)**로 봐요.

- `children`으로 슬롯(slot, 자리)을 선언해요.
- 컴포넌트는 `children`을 해석해서 적절한 레이아웃과 스타일을 만들어요.

이 방식은 API를 가볍게 만들어요.
그리고 확장성(extensibility, 확장성)도 좋아져요.

참고로, 확장은 보통 이런 도구로 해결해요.

- `children`
- render `prop` (렌더 `prop`)
- compound component (복합 컴포넌트)

### 2.3 `TextField.Inner`는 구조적 슬롯이에요 (`TextField.Inner` as a Structural Slot)

`TextField.Inner`는 `TextField` 안에서 **입력 필드(input field, 입력 필드)**가 들어갈 핵심 슬롯이에요.

RDS에서는 입력 영역을 `props`로 조립하지 않아요.
대신 **슬롯을 `children`으로 드러내요.**

이렇게 하면 이런 점이 좋아요.

- “어디가 입력 영역인지”가 코드에서 바로 보여요.
- adornment(장식 요소) 개수/위치가 늘어나도 `props`가 폭발하지 않아요.
- 아이콘이 아니라 임의의 요소도 자연스럽게 들어가요.

예시:

```tsx
<TextField>
	<SearchIcon />
	<TextField.Inner />
	<ClearIcon onClick={clear} />
</TextField>
```

### 2.4 `children`은 구조예요 (`children` are Structure)

RDS에서 `children`은 “내용”이 아니라 “구조”예요.
컴포넌트는 `children`을 분석하고, 필요한 구조로 재배치해요.

이것이 RDS의 핵심 원칙 중 하나입니다: **`children` = 구조(structure)**.

## 3. 선언적 UI는 컴포넌트를 과하게 쪼개는 게 아니에요 (Declarative UI ≠ Over-Componentization)

선언적이라는 이유로 모든 책임을 컴포넌트 트리 안에 넣지 않아요.
RDS는 책임(responsibility, 책임)을 분리해요.

### 3.1 View와 Control을 분리해요 (View vs Control Responsibility Split)

- View(뷰)는 “보여주는 것”만 책임져요.
- Control(제어)은 “열고/닫고/전환하는 것”을 책임져요.

트리거(trigger, 트리거)는 보통 비즈니스 로직에 가까워요.
그래서 View 컴포넌트가 트리거를 소유하지 않아요.

### 3.2 왜 `Modal.Trigger`는 권장하지 않아요? (Why `Modal.Trigger` Is Discouraged)

비권장:

```tsx
<Modal.Root>
	<Modal.Trigger>
		<Button />
	</Modal.Trigger>
</Modal.Root>
```

권장:

```tsx
<Button onClick={() => overlay.open(() => <Modal />)} />
```

`Modal.Root` 같은 View 컴포넌트가 트리거까지 책임지면, Modal이 너무 많은 책임을 가져요.
RDS는 “뷰는 뷰답게” 유지해요.

## 4. `props`는 의미(semantic)만 담아요 (Semantic-only `props`)

RDS에서 `props`는 **의미(semantic, 의미)**와 **시스템 제어(system control, 시스템 제어)**만 담아요.

허용하는 `props` 예시:

- `variant`
- `size`
- `intent` / `tone`
- `disabled`, `loading`, `invalid`, `readOnly`
- `value`, `onChange`, `name`
- `aria-*`
- `className` (보강 용도)
- `style` (런타임 측정/애니메이션 등 불가피한 경우만)

지양하는 `props` 예시:

- 위치/레이아웃/구현 디테일을 제어하는 `props`
- 조합을 대신하는 shortcut `props`

## 5. 시스템 `prop`은 자동으로 전파돼요 (System Prop Propagation)

Section 7에서는 Section 4에서 정의한 시스템 `props`의 런타임 동작 방식을 설명해요.

### 5.1 `size` 전파 규칙이에요 (Size Propagation Rules)

`size`는 시스템 `prop`이에요.
그래서 하위 요소로 자동 전파돼요.

예시:

```tsx
<Button size="md">
	<IconSun />
	라이트 모드
</Button>
```

- child가 `size`를 직접 적으면 child가 우선이에요.

### 5.2 전파는 경계가 있어요 (Propagation Boundaries)

전파는 아무 데나 퍼지지 않아요.
RDS가 관리하는 컴포넌트에만 전파돼요.

## 6. 의미 없는 `div`는 줄여요 (Layout System)

`div`로만 레이아웃을 쌓으면 의도가 사라져요.
RDS는 의도를 드러내는 레이아웃 컴포넌트를 제공해요.

### 6.1 레이아웃 프리미티브예요 (Layout Primitives)

- `Box` — 최소 단위 래퍼
- `HStack` / `VStack` / `ZStack` — 흐름 / 겹침
- `Flex` — 일반 flexbox
- `Grid` — 2D 레이아웃

### 6.2 책임을 나눠요 (Responsibility Split)

- `Box`: 단일 영역 스타일
- `Stack`: 1D 흐름
- `Flex`: 일반 flex
- `Grid`: 2D 배치
- `Group`: 인접 관계 규칙
- `Composite`: 구조적 UI

## 7. 스타일은 Tailwind로만 써요 (Tailwind-first Styling)

### 7.1 기본은 Tailwind예요 (Tailwind-first)

RDS는 Tailwind CSS만 사용해요.
CSS-in-JS, inline `style`은 기본적으로 쓰지 않아요.

### 7.2 `style`은 예외적으로만 써요 (Runtime Style Exceptions)

아래 경우에만 `style`을 허용해요.

- 런타임 측정 값
- 애니메이션 중간값
- canvas / overlay positioning

## 8. variant 시스템은 `tailwind-variants(tv)`만 써요 (Variant System)

- 클래스 조합은 `tailwind-variants(tv)`만 사용해요.
- `cva`는 쓰지 않아요.
- `variant` / `size` / `state`는 전부 `tv()` 안에서 정의해요.

## 9. 브랜드 색상에 상관없이 예쁘게 만들어요 (Brand-agnostic by Construction)

### 9.1 색상은 semantic token만 써요 (Semantic Tokens Only)

컴포넌트는 브랜드 색을 직접 참조하지 않아요.
오직 semantic token만 사용해요.

허용:

- `bg-primary`
- `text-primary-foreground`

금지:

- `bg-[#58c1c8]`
- `text-[var(--brand)]`

### 9.2 상태는 색을 바꾸지 않아요 (State Layer over Color Mutation)

hover / active / pressed 상태는 base color를 바꾸지 않아요.

대신 이렇게 해요.

- 동일한 색 위에 state mask layer를 얹어요.
- `opacity`로 강도만 조절해요.

이 방식이 좋은 이유는 간단해요.

- 브랜드 색이 바뀌어도 상태 표현이 일관돼요.
- state별 bg/fg 토큰이 끝없이 늘어나는 걸 막아요.

### 9.3 강제로 지켜요 (Lint & Enforcement)

- `eslint-plugin-tailwindcss`
- `tailwindcss/no-arbitrary-value` 활성화
- `arbitrary color value` 전면 금지

## 10. 타입은 완전히 안전해야 해요 (Type System Principle)

### 10.1 완전한 타입 안정성이 기본이에요 (Fully Type-safe)

RDS 컴포넌트는 전부 타입 안전하게 만들어야 해요.

### 10.2 타입은 컴포넌트에 붙어 있어야 해요 (Component-scoped Types)

모든 컴포넌트는 아래 타입을 공개해요.

- `{Component}.Props`
- `{Component}.State`

이건 RDS의 공개 타입 계약(public type contract, 공개 타입 계약)이에요.

### 10.3 네임스페이스 병합으로 타입을 붙여요 (Namespace Merging)

RDS는 TypeScript의 **namespace merging(네임스페이스 병합)**을 사용해서,
컴포넌트 식별자(identifier, 식별자)에 타입을 붙여요.

이 방식이 좋은 이유예요.

- 타입을 따로 export/import 하지 않아도 돼요.
- IDE 자동 완성이 잘 돼요. (discoverability, 탐색성)
- 런타임 코드는 단순해요.
- DX(Developer Experience, 개발자 경험)가 좋아져요.

예시:

```ts
export function Button(props: Button.Props) {
	// ...
	return null
}

export namespace Button {
	export type Props = {
		size?: 'sm' | 'md' | 'lg'
		disabled?: boolean
	}

	export type State = {
		focused: boolean
		pressed: boolean
	}
}

type P = Button.Props
type S = Button.State
```

이 패턴은 “구현 트릭”이 아니에요.
RDS의 공개 API 계약이에요.

## 11. `Skeleton`은 단순하게도, 편하게도 써요 (Skeleton)

`Skeleton`은 두 방식 모두 지원해요.

- `width` / `height` / `shape`로 단순하게 쓸 수 있어요.
- `children`을 넣으면 그 형태를 그대로 skeleton으로 만들어요.

`children` 기반 방식에서는,
원본 요소를 `invisible`로 렌더링해서 레이아웃을 잡아요.

예시:

```tsx
<Skeleton>
	<Button size="md">로딩</Button>
</Skeleton>
```

## 12. Storybook으로 품질을 확인해요 (Quality Gate)

모든 public 컴포넌트는 Storybook을 지원해야 해요.

- `size` / `variant` / `state` 조합을 보여줘야 해요.
- light / dark / brand 테마로 시각 회귀 테스트를 해야 해요.

## 13. 진입장벽은 낮아야 해요 (Low Entry Barrier & Native-first)

RDS는 배우기 어렵지 않아야 해요.
그래서 HTML/React의 기본 mental model을 최대한 살려요.

### 13.1 HTML 기능은 그대로 지원해요 (Native-first)

HTML primitive의 기능을 숨기거나 이름을 바꾸지 않아요.

예시:

```tsx
<TextField type="email" />
<TextField type="password" />
<TextField type="date" />
```

### 13.2 `type`으로 더 좋은 UI를 제공해요 (Smart Enhancement via `type`)

`type`만으로는 UX가 부족한 경우가 있어요.
그럴 때 RDS는 내부 구현을 더 좋은 컴포넌트로 바꿔요.

예:

- `type="date"` → `DateTimePicker`
- `type="color"` → `ColorPicker`

API는 그대로 유지해요.
구현만 더 똑똑해져요.

### 13.3 자동화는 항상 끌 수 있어요 (Opt-out & No Surprise)

자동화는 편의 기능이에요.
그래서 언제든 opt-out할 수 있어야 해요.
그리고 동작이 예상 가능해야 해요.

## 14. 다형성은 `asChild`로 해요 (Polymorphism: `asChild`)

이 패턴은 Section 3에서 말한 책임 분리 원칙의 구체적인 구현 방식이에요.

`as`는 타입 안전성을 해치기 쉬워요.
그래서 RDS는 `asChild` 패턴을 표준으로 사용해요.

예시:

```tsx
<Button asChild>
	<a href="/docs">문서</a>
</Button>
```

이 방식은 좋아요.

- child 타입을 그대로 유지해요.
- 커스텀 컴포넌트도 자연스럽게 지원돼요.
- `props` 충돌이 없어요.
- `ref` 전달도 안정적이에요.

`as="a"` 같은 API는 RDS 표준에서 사용하지 않아요.

## 15. 마지막 원칙이에요 (Final Principle)

RDS는 사용자를 귀찮게 하지 않아요.
디테일은 시스템이 책임져요.
사용자는 선언만 하면 돼요.