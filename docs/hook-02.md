# フック早わかり

https://ja.reactjs.org/docs/hooks-overview.html

## 📌 ステートフック

この例ではカウンターを表示します。ボタンをクリックすると、カウンターの値が増えます：

```jsx
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

この例のuseStateがフック（この単語の意味するところはすぐ後で説明します）です。関数コンポーネントの中でローカルなstateを使うために呼び出しています。このstateは以降の再レンダーの間もReactによって保持されます。useStateは現在のstateの値と、それを更新するための関数とをペアにして返します。この関数はイベントハンドラやその他の場所から呼び出すことができます。クラスコンポーネントにおけるthis.setStateと似ていますが、新しいstateが古いものとマージされないという違いがあります。（useStateとthis.stateの違いについてはステートフックの利用で例を挙げて説明します）useStateの唯一の引数はstateの初期値です。上記の例では、カウンターがゼロからスタートするので0を渡しています。this.stateと違い、stateはオブジェクトである必要はないことに注意してください（オブジェクトにしたい場合はそうすることも可能です）。引数として渡されたstateの初期値は最初のレンダー時にのみ使用されます。

## 複数の state 変数の宣言

1つのコンポーネント内で2回以上ステートフックを使うことができます：

```jsx
function ExampleWithManyStates() {
  // Declare multiple state variables!
  const [age, setAge] = useState(42);
  const [fruit, setFruit] = useState('banana');
  const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);
  // ...
}
```

配列の分割代入構文を使うことで、useStateを呼び出して宣言したstate変数に、異なる名前をつけることができます。これらの名前自体はuseStateのAPIの一部ではありません。その代わりに、ReactはuseStateを何度も呼び出す場合は、それらが全てのレンダー間で同じ順番で呼び出されるものと仮定します。この後で、どうしてこれが上手く動作し、どのような場合に便利なのか改めて説明します。

## 要するにフックとは？

フックとは、関数コンポーネントにstateやライフサイクルといったReactの機能を“接続する(hook into)”ための関数です。フックはReactをクラスなしに使うための機能ですので、クラス内では機能しません。今すぐに既存のコンポーネントを書き換えることはお勧めしませんが、新しく書くコンポーネントで使いたければフックを利用し始めることができます。ReactはuseStateのような幾つかのビルトインのフックを提供します。異なるコンポーネント間でステートフルな振る舞いを共有するために自分自身のフックを作成することもできます。まずは組み込みのフックから見ていきましょう。

## ⚡️ 副作用フック

これまでにReactコンポーネントの内部から、外部データの取得や購読(subscription)、あるいは手動でのDOM更新を行ったことがおありでしょう。これらの操作は他のコンポーネントに影響することがあり、またレンダーの最中に実行することができないので、われわれはこのような操作を“副作用(side-effects)“、あるいは省略して“作用(effects)”と呼んでいます。useEffectは副作用のためのフックであり、関数コンポーネント内で副作用を実行することを可能にします。クラスコンポーネントにおけるcomponentDidMount,componentDidUpdateおよびcomponentWillUnmountと同様の目的で使うものですが、1つのAPIに統合されています。（これらのメソッドとuseEffectとの違いについては副作用フックの利用法で例を挙げて説明します）例えば、このコンポーネントはReactがDOMを更新した後で、HTMLドキュメントのタイトルを設定します。

```jsx
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

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

useEffectを呼ぶことで、DOMへの更新を反映した後にあなたが定義する「副作用関数」を実行するようにReactに指示します。副作用はコンポーネント内で宣言されるので、propsやstateにアクセスすることが可能です。デフォルトでは初回のレンダーも含む毎回のレンダー時にこの副作用関数が呼び出されます。（クラスにおけるライフサイクルメソッドとの比較は副作用フックの利用法で詳しく述べます）副作用は自分を「クリーンアップ」するためのコードを、オプションとして関数を返すことで指定できます。例えば、以下のコンポーネントでは副作用を利用して、フレンドのオンラインステータスを購読し、またクリーンアップとして購読の解除を行っています。

```jsx
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

この例では、コンポーネントがアンマウントされる時や再レンダーによって副作用が再実行される時にChatAPIの購読解除を行っています。（必要なら、ChatAPIに渡すためのprops.friend.idが変わっていない場合には毎回購読しなおす処理をスキップする方法があります）useStateの場合と同様、1つのコンポーネント内で2つ以上の副作用を使用することが可能です。

```jsx
function FriendStatusWithCounter(props) {
  const [count, setCount] = useState(0);
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }
  // ...
```

フックを利用することで、ライフサイクルメソッドの場合は分離して書かざるを得なかったコンポーネント内の副作用を、関連する部分（リソースの購読とその解除、など）同士で整理して記載することが可能になります。useEffectについての更なる詳細は副作用フックの利用法を参照してください。

## ✌️ フックのルール

フックは JavaScript の関数ですが、2 つの追加のルールがあります。

- フックは関数のトップレベルのみで呼び出してください。ループや条件分岐やネストした関数の中でフックを呼び出さないでください。
- フックは React の関数コンポーネントの内部のみで呼び出してください。通常の JavaScript 関数内では呼び出さないでください（ただしフックを呼び出していい場所がもう 1 カ所だけあります — 自分のカスタムフックの中です。これについてはすぐ後で学びます）。

これらのルールを自動的に強制するためのlinterpluginを用意しています。これらのルールは最初は混乱しやすかったり制限のように思えたりするかもしれませんが、フックがうまく動くためには重要なものです。これらのルールについての詳細はフックのルールを参照してください。

## 💡 独自フックの作成

stateを用いたロジックをコンポーネント間で再利用したいことがあります。これまでは、このような問題に対して2種類の人気の解決方法がありました。高階コンポーネントとレンダープロップです。カスタムフックを利用することで、同様のことが、ツリー内のコンポーネントを増やすことなく行えるようになります。このページの上側で、useStateとuseEffectの両方のフックを呼び出してフレンドのオンライン状態を購読するFriendStatusというコンポーネントを紹介しました。この購読ロジックを別のコンポーネントでも使いたくなったとしましょう。まず、このロジックをuseFriendStatusというカスタムフックへと抽出します。

```jsx
import React, { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
```

これはfriendIDを引数として受け取り、フレンドがオンラインかどうかを返します。これで両方のコンポーネントからこのロジックが使えます：

```jsx
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}

function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```

これらのコンポーネントのstateは完全に独立しています。フックはstateを用いたロジックを再利用するものであって、stateそのものを再利用するものではありません。実のところ、フックのそれぞれの呼び出しが独立したstateを持っていますので、全く同じカスタムフックを1つのコンポーネント内で2回呼び出すことも可能です。カスタムフックは、機能というよりはむしろ慣習のようなものです。関数の名前が”use”から始まって、その関数が他のフックを呼び出しているなら、それをカスタムフックと言うことにする、ということです。このuseSomethingという命名規約によって、我々のlinterプラグインはフックを利用しているコードのバグを見つけることができます。カスタムフックは、フォーム管理や、アニメーションや、宣言的なデータソースの購読や、タイマーや、あるいはおそらく我々が考えたこともない様々なユースケースに利用可能です。Reactのコミュニティがどのようなカスタムフックを思いつくのか楽しみにしています。カスタムフックについての詳しい情報は独自フックの作成を参照してください。

## 🔌 その他のフック

その他にもいくつか、使用頻度は低いものの便利なフックが存在しています。例えば、useContext を使えば React のコンテクストをコンポーネントのネストなしに利用できるようになります：

```jsx
function Example() {
  const locale = useContext(LocaleContext);
  const theme = useContext(ThemeContext);
  // ...
}
```

またuseReducerを使えば複雑なコンポーネントのローカルstateをリデューサ(reducer)を用いて管理できるようになります：

```jsx
function Todos() {
  const [todos, dispatch] = useReducer(todosReducer);
  // ...
```

全てのビルトインフックについての詳細はフック API リファレンスを参照してください。

## 次のステップ

かなり駆け足の説明でしたね！まだよく分からないことがあった場合や、より詳しく学びたいと思った場合は、ステートフックから始まるこの先のページに進んでください。フックAPIリファレンスとよくある質問も参照してください。最後になりましたが、フックの紹介ページもお忘れなく！そこではなぜ我々がフックを導入することにしたのか、またアプリ全体を書き換えずにクラスと併用してどのようにフックを使っていくのか、について説明しています。
