# Makers' Principle: Rhseung's Design System (RDS)

RDS는 단순한 컴포넌트 모음이 아닌, **선언적인 UI 문법(Declarative UI Syntax)**입니다. 
개발자가 UI의 **구조(Structure)**를 선언하면, 시스템이 **디테일(Detail)**과 **인접 처리(Adjacency)**를 책임집니다.

## For React

### 1. 선언적 UI와 시스템 책임 (Declarative by Default)
개발자는 "무엇을 만들고 싶은지"만 명확히 적습니다. "어떻게 구현할지"는 시스템의 몫입니다.

* **디테일 자동화**: `radius`, `border`, `spacing` 등을 개발자가 직접 계산하지 않습니다.
* **인접 처리(Adjacency)**: 컴포넌트가 서로 맞닿을 때 발생하는 스타일 수정은 시스템이 내부적으로 처리합니다.

```tsx
// ✅ 개발자는 구조만 선언합니다.
<ButtonGroup variant="outline">
    <Button>왼쪽</Button>
    <Button>가운데</Button>
    <ButtonGroup.Separator />
    <Button>오른쪽</Button>
</ButtonGroup>

/**
 * [시스템의 책임]
 * 1. 첫 번째 버튼: 오른쪽 border-radius 제거
 * 2. 중간 버튼: 양쪽 border-radius 제거 및 좌측 border 중첩 해결
 * 3. 구분선(Separator) 뒤의 버튼: 간격 및 경계면 재설정
 */
```



### 2. 구조적 슬롯 (Structural Children over Prop-Dumping)

특정 UI 기능을 위해 `props`를 무분별하게 나열하거나 객체 형태로 주입하는 방식을 지양합니다.

#### 2.1 기존 방식의 문제 (Prop-Dumping)

MUI와 같은 기존 라이브러리는 adornment를 추가할 때 코드가 비대해지고 구조 파악이 어렵습니다.

```tsx
// ❌ 비권장: Prop-Dumping (코드가 길어지고 구조가 감춰짐)
<TextField
  label="검색"
  InputProps={{
    startAdornment: (
      <InputAdornment position="start">
        <SearchIcon />
      </InputAdornment>
    ),
    endAdornment: (
      <IconButton onClick={clear}><ClearIcon /></IconButton>
    ),
  }}
/>
```

#### 2.2 RDS의 해결책: Structural Slots

`children`을 '내용'이 아닌 '구조'로 취급합니다. 슬롯을 밖으로 드러내어 직관적인 레이아웃을 구성합니다.

```tsx
// ✅ 권장: RDS의 선언적 슬롯 (구조가 한눈에 보이며 확장이 자유로움)
<TextField placeholder="검색어를 입력하세요">
    <SearchIcon />
    <TextField.Inner />
    <ClearIcon onClick={clear} />
</TextField>
```



### 3. 시스템 속성의 전파 (System Prop Propagation)

`size`, `intent`와 같은 시스템 `prop`은 하위 요소로 자동 전파되어 코드 반복을 줄입니다.

* **자동 감지**: 부모의 속성을 자식이 Context를 통해 감지하여 스스로를 최적화합니다.
* **우선순위**: 자식 요소에 명시된 속성이 부모의 전파값보다 항상 우선합니다.

```tsx
// ✅ IconSun은 Button의 'md' 사이즈를 자동으로 감지합니다.
<Button size="md">
    <IconSun /> 
    라이트 모드
</Button>

// 복합 구조에서의 전파
<ButtonGroup size="lg">
    <Button>대형 버튼</Button>
    <Button size="sm">나만 소형 버튼</Button> {/* 자식 설정이 우선됨 */}
</ButtonGroup>
```



### 4. 관심사의 분리 (View vs Control)

View 컴포넌트가 트리거(제어 로직)를 소유하지 않도록 분리하여 책임 소재를 명확히 합니다.

* **View**: "어떻게 보이는가"만 책임집니다.
* **Control**: "언제 열리고 닫히는가"는 비즈니스 로직(예: Overlay 매니저)이 결정합니다.

```tsx
// ❌ 비권장: Modal이 트리거까지 책임지는 구조
<Modal.Root>
    <Modal.Trigger><Button>열기</Button></Modal.Trigger>
</Modal.Root>

// ✅ 권장: 트리거는 외부 비즈니스 로직에서 관리
const { open } = useOverlay();
<Button onClick={() => open(() => <MyModal />)} />
```



### 5. 레이아웃 시스템 (Layout Primitives)

의미 없는 `div`를 줄이고, 레이아웃의 의도를 명확히 드러내는 프리미티브를 사용합니다.

* **Zero Margin Rule**: 모든 원자 컴포넌트(Button, Input 등)는 외곽 마진이 0입니다. 간격은 레이아웃 컴포넌트가 책임집니다.
* **Primitives**: `Box`, `HStack/VStack`, `Flex`, `Grid`, `Group`.



### 6. 브랜드 독립적 설계 (Brand-agnostic)

특정 브랜드 색상에 의존하지 않고 시스템적으로 안정적인 시각 피드백을 제공합니다.

* **Semantic Tokens**: `bg-[#58c1c8]` 같은 하드코딩 대신 `bg-primary` 토큰만 사용합니다.
* **State Layering**: hover/active 상태 시 색상을 직접 변경하지 않고, 그 위에 **State Mask(투명도 레이어)**를 얹습니다. 이는 어떤 브랜드 색상 위에서도 일관된 상태 표현을 가능하게 합니다.



### 7. 타입 안전성과 네임스페이스 (Type System)

RDS는 개발자 경험(DX)을 위해 TypeScript의 **Namespace Merging**을 표준으로 사용합니다.

* **Public Contract**: 모든 컴포넌트는 `{Component}.Props`와 `{Component}.State`를 공개합니다.
* **탐색성(Discoverability)**: IDE에서 `TextField.`을 입력하는 것만으로 사용 가능한 모든 서브 컴포넌트와 타입을 탐색할 수 있습니다.

```ts
export function Button(props: Button.Props) { ... }

export namespace Button {
    export type Props = { size?: 'sm' | 'md' | 'lg'; ... };
    export type State = { pressed: boolean; ... };
}
```



### 8. 다형성 표준 (Polymorphism: asChild)

타입 안전성을 저해하고 `props` 충돌을 일으키는 `as` 속성 대신, **`asChild`** 패턴을 사용합니다.

```tsx
// ✅ <a> 태그의 기능을 유지하면서 Button의 스타일을 안전하게 입힘
<Button asChild>
    <a href="/docs">문서 읽기</a>
</Button>
```



### 9. 하이엔드 기본 경험 (Native-first)

* **HTML 준수**: `type="password"`, `type="date"` 등 기본 HTML 속성을 가리지 않고 지원합니다.
* **Smart Enhancement**: 인터페이스는 유지하되, 필요시 내부적으로 더 고도화된 UI(예: Custom DatePicker)로 대체 렌더링합니다.

## For Flutter

## For SwiftUI
