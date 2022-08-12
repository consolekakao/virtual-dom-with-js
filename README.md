# Vanilla JavaScript로 Virtual DOM 구현하기

```
// 실행 방법!
npm install
npm run babel
npm run start
```

## 📚 Virtual DOM을 구현하기 위한 정리 📚
> - React Element
> - JSX vs JS
> - Virtual DOM

---

## React Element
- React 앱의 가장 작은 단위.
- 컴포넌트의 구성 요소.
- 브라우저 DOM element 와 다름.
  - Document안의 모든 객체가 상속하는 제일 범용적인 기반 클래스.
  - 공통 메서드와 속성만 가짐.
  - 특정 요소를 더 상세하게 표현하는 클래스가 element를 상속.  
    ex. HTMLElement 인터페이스 : HTML 요소의 기반 인터페이스.  
    ex. SVGElement 인터페이스 : 모든 SVG 요소의 기초.


#### Root DOM Node
- React로 구현된 애플리케이션은 일반적으로 하나의 루트(root) DOM 노드가 있음.
- 이 안에 들어가는 모든 element를 React Dom 에서 관리함.
```
<div id="root"></div>
```
- React element를 루트 DOM 노드에 렌더링하려면 둘 다 ReactDOM.render()로 전달.
```
const element = <h1>Hello, world</h1>;
ReactDOM.render(element, document.getElementById('root'));
```

---

## JSX vs JS
- JSX : JavaScript를 확장한 문법. React element를 생성함.
- 각 JSX element 는 `React.createElement(component, props, ...children)` 를 호출하기 위한 Syntax Sugar.
- JSX 구문 자체는 브라우저에서 읽을 수 없음. 일반 JavaScript로 변환해야 함.
- [Babel](https://babeljs.io/repl/#?browsers=defaults%2C%20not%20ie%2011%2C%20not%20ie_mob%2011&build=&builtIns=false&corejs=3.21&spec=false&loose=false&code_lz=GYVwdgxgLglg9mABACwKYBt1wBQEpEDeAUIogE6pQhlIA8AJjAG4B8AEhlogO5xnr0AhLQD0jVgG4iAXyJyGzFqPEs5RCOgCGAZ22IOmOIlQAPKKjD09AJVSboAOgDCcALYAHBBaiES5C_SoZHi-pKQUVDSICqwGXARQyDDaDu5kcO4pUHAA6siaUNLKilKksrJEtvZQACIA8gCyDhSWQdh-tHFG2XkFALwARDl8AgOIIiwANH70cBAgrt4OAOaUAKLoqItgUABCAJ4AkvTYAOTpcFCnuES4UvIg6IgaOtoAcpqLg-jJUAOqpFoPxYMHMrkQAEZRMCOsDQVtEAAmaEwVSiR4sIA&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=react&prettier=false&targets=&version=7.18.12&externalPlugins=&assumptions=%7B%7D)
  : JSX를 React.createElement 함수를 호출하는 JavaScript로 변환.
```
// JSX
class Hello extends React.Component {
  render() {
    return <div>Hello {this.props.toWhat}</div>;
  }
}

ReactDOM.render(
  <Hello toWhat="World" />,
  document.getElementById('root')
);
```
```
// JS
class Hello extends React.Component {
  render() {
    return React.createElement('div', null, `Hello ${this.props.toWhat}`);
  }
}

ReactDOM.render(
  React.createElement(Hello, {toWhat: 'World'}, null),
  document.getElementById('root')
);
```
```
<ul className="list">
  <li>item 1</li>
  <li>item 2</li>
</ul>

// Babel JSX 문서를 보면, Babel이 위의 코드를 아래와 같이 번역함

React.createElement('ul', { className: 'list' },
  React.createElement('li', {}, 'item 1'),
  React.createElement('li', {}, 'item 2'),
);
```

---

## Virtual DOM (VDOM)
- DOM 형태를 본따 만든 객체 덩어리.

#### 1.DOM Tree 표현하기.
- DOM Tree를 JavaScript Object(Virtual DOM)로 메모리에 저장하기.
```
// 예를 들어 아래의 Tree를 구현했다고 가정하자
<ul class="list">
  <li>item 1</li>
  <li>item 2</li>
</ul>

// 위의 DOM 요소를 JS Object로 표현하면 아래와 같음
{ 
  type: 'ul', props: { 'class': 'list' }, children: [
    { type: 'li', props: {}, children: ['item 1'] },
    { type: 'li', props: {}, children: ['item 2'] }
  ]
}
// type, props, children 반복. -> 큰 규모의 Tree를 만들기는 힘듦
```
```
// 헬퍼 함수 생성 후 사용
function h(type, props, ...children) {
  return { type, props, children: children.flat() }
}

h('ul', { 'class': 'list' },
  h('li', {}, 'item 1'),
  h('li', {}, 'item 2'),
);
// JSX와 모양이 비슷?!

// jsx pragma를 통해 React.createElement를 h함수로 대체하자
// pragma(프라그마) : 컴파일러 지시문. 컴파일러에게 파일 내용을 처리하는 방법을 알려줌.
// 1. Babel 플러그인에 옵션 추가. 2. 모듈 시작 부분에 pragma 주석 설정.
```
```
// jsx pragma. React.createElement 대신에 h를 사용하라고 Babel에게 전달
/** @jsx h */ 

const a = (
  <ul className="list">
    <li>item 1</li>
    <li>item 2</li>
  </ul>
);

// 위의 코드를 Babel이 아래와 같이 변환

const a = (
  h('ul', { className: 'list' },
    h('li', {}, 'item 1'),
    h('li', {}, 'item 2'),
  );
);

// h함수(직접 생성한 헬퍼 함수)를 실행하면 JavaScript Object(Virtual DOM 표현)가 반환됨

const a = (
  { type: 'ul', props: { className: 'list' }, children: [
    { type: 'li', props: {}, children: ['item 1'] },
    { type: 'li', props: {}, children: ['item 2'] }
  ] }
);
```

#### 2. JavaScript Object(Virtual DOM)로 Real DOM 표현하기
- Virtual DOM을 Real DOM으로 변경하기.


- 아래 코드의 3가지 규칙.
  - 모든 변수를 "$"로 시작하는 Real DOM(element, text node)로 작성.
  - Virtual DOM 표현은 node 변수에 포함.
  - 하나의 root node만 가짐. 모든 다른 node는 root node 안에 위치.
  - props은 제외. (속성은 Virtual DOM의 기본 개념을 이해하는데 필요하지 않음)
```
// createElement 함수 : Virtual DOM Node를 가져와서 Real DOM Node 반환
function createElement(node) {
  if (typeof node === 'string') {
    // TextNode(JavaScript 문자열)
    return document.createTextNode(node);
  }

  // { type: '…', props: { … }, children: [ … ] } 형식의 JavaScript Object
  const $el = document.createElement(node.type);
  node.children
    .map(createElement)
    .forEach($el.appendChild.bind($el));
  return $el;
}
```

#### 3. 변경사항 처리
- Virtual Tree의 변화 감지.
- 뷰(HTML)에 변화가 있을 때, 구 가상돔(Old Node)과 새 가상돔(New Node)을 비교하여 변경된 내용만 DOM에 적용.
```
function changed(node1, node2) {
  return typeof node1 !== typeof node2 ||
         typeof node1 === 'string' && node1 !== node2 ||
         node1.type !== node2.type
}


// updateElement 함수 : parent, newNode, oldNode, index 파라미터 받음
// parent : Virtual Node의 Real DOM element 부모
// index : 부모 element에 있는 node의 위치
function updateElement($parent, newNode, oldNode, index = 0) {
  
  // newNode는 있는데 oldNode가 없는 경우 (node가 새로 추가된 경우)
  if (!oldNode) {
    $parent.appendChild(
      createElement(newNode)
    );

  
  // oldNode는 있는데 newNode이 없는 경우 (node를 삭제해야 하는 경우)
  } else if (!newNode) {
      $parent.removeChild(
        $parent.childNodes[index]
      );
    }


  // oldNode와 newNode가 다른 경우 (node가 변경된 경우)
  // $parent(부모 node)에서 index(현재 node의 인덱스)로 newNode(새로 생성된 node) 대체
  } else if (changed(newNode, oldNode)) {
    $parent.replaceChild(
      createElement(newNode),
      $parent.childNodes[index]
    );


  // newNode와 oldNode이 동일하므로 두 노드의 모든 children(자식 노드) 비교
  // 실제로 각 node마다 updateElement를 호출해야 함 (재귀)
  // 고려사항
  //   - text node는 자식(children)을 가질 수 없기 때문에, element node만 비교
  //   - 현재 node에 대한 참조를 부모로 전달
  //   - 모든 자식(children)을 하나씩 비교
  //   - 인덱스(i) : 자식(children) 배열의 child node의 인덱스
  } else if (newNode.type) {
      const newLength = newNode.children.length;
      const oldLength = oldNode.children.length;
      for (let i = 0; i < newLength || i < oldLength; i++) {
        updateElement(
          $parent.childNodes[index],
          newNode.children[i],
          oldNode.children[i],
          i
        );
      }
    }
}
```
