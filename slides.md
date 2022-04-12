---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
---

# Composablesについて（後編）

---

# Agenda

- React Hooksとの違い
- Composablesのテストについて
- Componentテストについて

---

# 今日のひとこと

<div style="display: flex; justify-content: space-between">
  <div>
    <h2>"Night of the Demon"のBlu-ray購入！</h2>
    <ul>
      <li>1957年作</li>
      <li>監督: ジャック・ターナー</li>
      <li>RKOホラー立役者の偉大な監督！</li>
        <ul style="margin-left: 30px">
          <li>『キャット・ピープル』</li>
          <li>『レオパルド・マン』</li>
          <li>『わたしはゾンビと歩いた！』</li>
        </ul>
      <li>黒沢清などに多大なる影響を与えた！</li>
    </ul>
  </div>
  <img src="/night_of_the_demon_.jpg" alt="" style="width: 300px">
</div>

---

# React Hooksとは？

- React 16.8で追加された機能のこと
- クラスコンポーネント　→ 関数コンポーネント
- VueでいうとOptions → Composition的な感じ？

```js
import React, { useState } from 'react';

function Example() {
  // Declare a new state variable, which we'll call "count"
  const [count, setCount] = useState(0);

  return (
          <div>
            <p>You clicked {count} times</p>
            <button onClick={() => setCount(count + 1)}>
              Click me
            </button>
          </div>
  );
}
```

---

# vs. React Hooks

- VueのComposables概念はReactのCustom Hooksに酷似している
  - Composition APIはReactのHooksに影響を受けている（ロジックの外部切り出しなど）
  - しかし、Vueの場合はReactiveシステムに基づいており、Hooksの実行モデルとは根本的に違う

---

# vs. React Hooks

- React Hooksはコンポーネントが更新されるたびに繰り返し呼び出される（再レンダリング）
  - propsが更新されたとき
  - stateが更新されたとき
  - 親コンポーネントが再レンダリングされたとき
- 野放しにしておくとパフォーマンスに影響が生じる

---

# vs. React Hooks

- React Hooksは条件分岐などの文脈で使えない
- https://ja.reactjs.org/docs/hooks-rules.html#explanation

---

# vs. React Hooks

- 正しい依存関係をReact Hooksに渡さないとStale-Closure問題が生じる
  - 「古くなった」クロージャーのこと
  - 「外側のスコープにある変数への参照を保持できる」という関数が持つ性質
    - 関数に状態を持たせる手段として
    - 外から参照できない変数を定義する手段として
    - グローバル変数を減らす手段として
    - 高階関数の一部分として

---

# vs. React Hooks

- `useState`, `useEffect` の例
- https://dmitripavlutin.com/react-hooks-stale-closures/#3-stale-closures-of-hooks

---

# vs. React Hooks

- 高価な計算には`useMemo`で正しい依存関係の配列を手動で渡さなければならない
  - `useMemo`とはメモ化された値を返すもの
  - 高負荷な再計算を防ぐ役割のもの
  - 依存する配列のいずれかが変化した場合にのみメモ化された値を再計算する

```js
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

---

# vs. React Hooks

- 子コンポーネントに渡されるイベントハンドラーは、デフォルトで不必要な子コンポーネントの<br />更新を引き起こすので、最適化として明示的に`useCallback`を使う必要がある
  - `useCallback`はメモ化されたコールバックを返すもの
  - コールバック関数と、それが依存している値の配列を渡す
  - そのコールバックがメモ化したものを返し、依存配列が変化した場合にのみ変化する

```js
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);

```

---

# vs. React Hooks

<p>要するに,,,</p>
<h2 style="display: flex; justify-content: center">考えることが多くてめんどくさい😂</h2>

---

# 一方、Composition APIは

- `setup()`, `<script setup>`構文は一度だけ呼び出されるため、Stale-Closure問題は心配無用
- 条件付きで呼び出すことができ、React Hooksと比べて柔軟性がある（ここは別途掘り下げたい）
  - https://logaretm.com/blog/conditional-vuejs-compositions/
- VueのReactiveでは、依存関係を自動的に収集するため手動での宣言が不要
- 不必要な子の更新を避けるために、コールバック関数を手動でキャッシュする必要がない
  - 子コンポーネントは必要なときだけ更新される
  - 再レンダリングなどは開発者が懸念しなくてよい

---

# Vue側の主張

- React Hooksはリスペクトしてるが、色々問題があることは事実
- React熟練者でも混乱しやすいHooksのルールがある
- VueのReactive性が「たまたま」それらの問題を回避する方法を提供していることに気付いた

---

# vs. React Hooks

- それって本当なの？🤔
- 例えば「柔軟すぎるが故に厳密なルール化が必要」みたいなこともあるのでは？

---

# Testing Composables

- Composablesのテスト時には2つの考え方がある
  - ホストのコンポーネントインスタンスに依存するもの
  - 依存しないもの

---

# Testing Composables

ホストのコンポーネントインスタンスに依存するもの

- ライフサイクル
- Provide / Inject

---

# Testing Composables

Provideを使用している部分のテスト例

```js
// test-utils.js
import { createApp } from 'vue'

export function withSetup(composable) {
  let result
  const app = createApp({
    setup() {
      result = composable()
      // suppress missing template warning
      return () => {}
    }
  })
  app.mount(document.createElement('div'))
  // return the result and the app instance
  // for testing provide / unmount
  return [result, app]
}
```

---

# Testing Composables

Provideを使用している部分のテスト例

```js
import { withSetup } from './test-utils'
import { useFoo } from './foo'

test('useFoo', () => {
  const [result, app] = withSetup(() => useFoo(123))
  // mock provide for testing injections
  app.provide(...)
  // run assertions
  expect(result.foo.value).toBe(1)
  // trigger onUnmounted hook if needed
  app.unmount()
})
```

---

# Testing Composables

逆にReactivity APIのみの利用時（依存しない場合）は、特にラップなどは不要

```js
// counter.js
import { ref } from 'vue'

export function useCounter() {
  const count = ref(0)
  const increment = () => count.value++

  return {
    count,
    increment
  }
}
```

---

# Testing Composables

Reactivity APIのみの利用時

```js
// counter.test.js
import { useCounter } from './counter.js'

test('useCounter', () => {
  const { count, increment } = useCounter()
  expect(count.value).toBe(0)

  increment()
  expect(count.value).toBe(1)
})

```

---

# Component Testing

- コンポーネントのprops, event, 提供するslot, style, lifecycleなどに関連する問題を検出する必要
- 子コンポーネントをモックするのではなく、ユーザーと同じようにコンポーネントと<br />インタラクションを行い、相互間のインタラクションテストをしなければならない
- 内部の詳細の実装よりも、コンポーネントの公開インターフェースに焦点を当ててテストすべき（event, propsなど）

---

# Component Testing

してはいけないこと

- コンポーネントのプライベートな状態をアサートしたり、プライベートなメソッドをテストすること
  - 実装の詳細をテストすると、テストが壊れやすくなり、実装が変更されたときに更新が必要になるため
  - コンポーネントの最終的な仕事は正しいDOM出力のレンダリング
- どうしてもロジック部分をテストしたくなった場合は, utilsなど外部に切り出して行うべき

---

# Component Testing

テストツールのおすすめ（テストランナー）

- Vitest or Cypress
- Cypressはノードベースのランナーでは捕まえられない問題（スタイル、ネイティブDOMイベント、クッキーなど）をキャッチできるが、Vitestに比べて桁違いに遅くなる
- VueとしてはVitest推しっぽい？

---

# Component Testing

テストツールのおすすめ（マウンティングライブラリー）

- testing-library or test-utils
- コンポーネントのマウント、イベントのトリガー、出力に対するアサーションに利用
- Vueとしてはtesting-library推しっぽい
- テストの優先順位がしっかりしているため
- Vue固有の内部をテストする必要がある高度なコンポーネントを構築する場合のみtest-utilsを使うとのこと