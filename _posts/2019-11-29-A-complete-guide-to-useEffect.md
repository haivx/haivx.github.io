---
title: A Complete Guide to useEffect
date: 2019-11-29 15:40:43
tags:
  - technical
  - Javascript
  - React Hooks
---

Bạn đã sử dụng React Hooks trong các dự án của mình. Kể cả những app nhỏ hay lớn. Hầu hết đều cảm thấy hài lòng. Bạn thoải mái với những API và có được một vài tricks để code hiệu quả hơn với Hooks. Thậm chí bạn có thể viết được `custom hooks` để reuse code hiệu quả và show nó với đồng nghiệp. "Great job" - they said.

Nhưng đôi khi sử dụng `useEffect`, có những case có vẻ không được mượt mà cho lắm. Bạn bắt đầu cảm thấy cằn nhằn khó chịu rằng mình đã miss điều gì đó....

Bài viết được lược dịch từ Blog [overreacted.io](https://overreacted.io/a-complete-guide-to-useeffect/)

<!-- more -->
 Có vẻ nó giống lifecycle nhưng có thật sự là thế không? Bạn tự đặt câu hỏi cho bản thân kiểu như:

  - Làm cách nào tôi có thể sử dụng được `componentDidMount` bằng `useEffect`.
  - Cách fetch data trong `useEffect` đúng nhất là như thế nào? `[]` là cái gì vậy?
  - Liệu tôi có thể dùng một function như là effect dependency hay không?
  - Tại sao thỉnh thoảng tôi lại dính vụ lặp infinite fetching data?
  - Tại sao thi thoảng tôi lại get được state cũ hay prop cũ trong `effect`?

Khi tôi bắt đầu sử dụng Hooks, tôi cũng như bạn, cũng thăc mắc như vậy. Ngay cả khi viết những dòng này, tôi cũng không thể hiếu thấu được tất cả các khía cạnh của nó. Và tôi đã có những lúc phải kêu lên `aha` - điều tôi muốn chia sẻ với bạn. Dưới đây sẽ là những câu trả lời cho bạn.

> Chỉ đến khi dừng việc coi `useEffect` thông qua lăng kính của class lifecycle method, mọi thứ mới tỏ tường hơn với tôi.

> Hãy quên đi những thứ bạn đã học - Yoda

**1. TLDR**

<b>Questions: Làm cách nào tôi có thể sử dụng được `componentDidMount` bằng `useEffect`.<b>

Khi bạn sử dụng `useEffect(fn, [])`, nó không tương đương như nhau. Không giống như `componentDidMount`, nó sẽ `capture` lại props và state. Do đó, trong callback, bạn sẽ chỉ thấy được initial props và initial state. Nếu bạn muốn get được những giá trị mới nhất, bạn có thể viết nó với ref. Những thường sẽ có cách đơn giản hơn. Luôn giữ trong đầu rằng tư tưởng thiết kế `effects` sẽ khác so với việc sử dụng lifecycle, và việc thử tìm những cách tương đương nhau sẽ khiến bạn trở nên confuse hơn. Để hiệu quả hơn, hãy nghĩ theo hướng mới - `effects`, và tư tưởng thiết kế của chúng sẽ giống với hướng thực thi đồng bộ hơn là các sự kiện lifecycle.

<b>Questions: Cách fetch data trong `useEffect` đúng nhất là như thế nào? `[]` là cái gì vậy?<b>

`[]` nghĩa là effect sẽ không sử dụng bất kì giá trị nào trong flow data của React, và nó là cách an toàn để thực thi effect một lần. Sẽ có một vài bug khi giá trị được sử dụng. Bạn sẽ học một số giải pháp kiểu như `useReducer` hay `useCallback` để có thể loại bỏ sự cần thiết các dependency đó thay vì omit chúng.

<b>Questions: Liệu tôi có thể dùng một function như là effect dependency hay không?<b>

Điều khuyên dùng là hoist function không cần props và state ra khỏi component của bạn, và pull chúng về để sử dụng như là một effect trong effect đó. Sau đó, effect của bạn sẽ sử dụng function trong render scope(bao gôm function trong props), wrap chúng bằng `useCallback`, và tiếp tục. Tại sao nó lại hiệu quả hơn? Function có thể get được value từ props và state - do đó chúng sẽ tham gia vào data flow của react.

<b>Questions: Tại sao thỉnh thoảng tôi lại dính vụ lặp infinite fetching data<b>

Điều này có thể xảy ra kếu bạn fetching data trong effect mà không sử dụng dependency argument thứ hai. Nếu không có nó, effect sẽ được chạy mỗi khi render - và việc set state sẽ trigger effect thực hiện lại. Một vòng lặp vô hạn sẽ xảy ra nếu giá trị trong dependency array luôn thay đổi. Bạn có thể biết được thanh niên nào đang phá hoại bằng cách remove dần từng thanh niên một ra khỏi mảng. Tuy nhiên việc remove dependency bạn sử dụng hoặc sử dụng mảng rỗng thường là cách fix sai. THay vào đó, hãy xử lý vấn đề ngay trong source code. Ví dụ, function có thể gây ra vấn đề này, bỏ chúng vào effects, hoist lên đầu hoặc là wrapping chúng bằng `useCallback`. Để tránh việc `re-creating` objects, `useMemo` có thể là một lựa chọn.

<b>Questions: Tại sao thi thoảng tôi lại get được state cũ hay prop cũ trong `effect`<b>

Effects luôn luôn `see` props và state từ render nơi mà chúng được định nghĩa. Nó giúp ngăn chặn một vài bugs nhưng trong một vài trường hợp nó lại làm phức tạp vấn đề. Với những case như này, bạn cần sử dụng chính xác một số giá trị trong mutable ref. Nếu bạn cho rằng bạn đang get phải props hay state cũ, có thể bạn đang thiếu một vài dependency. 

*2. Mỗi lần render đều có một giá trị props và state mới*

Trước khi nói về Effects, ta cần nói về rendering.
Dưới đây là đoạn code về counter:
```js
  function Counter() {
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
Đoạn click count có nghĩa là gì? Vó phải biến `count` sẽ theo dõi sự thay đổi của state và update tự động? ĐÓ thường là suy nghĩ đầu tiên khi bạn học React, nhưng nó không phải chính xác tư tưởng của mô hình thiết kế này.

Trong ví dụ trên, `count` chỉ là một con số. Nó không phải là thứ `data binding` thần kì, `watcher`, `proxy` hay thứ nào đó. Nó chỉ là một con số cũ kiểu:
 ```js
  const count = 42;
  // ...
  <p>You clicked {count} times</p>
  // ...
 ```

 Thời điểm render đầu tiên, biến `count` ta get từ state sẽ là `0`. Khi ta gọi `setCount(1)`, React gọi lại component. Tại lúc này, `count` có giá trị là `1`. Và:
 ```js
  // During` first render
  function Counter() {
    const count = 0; // Returned by useState()
    // ...
    <p>You clicked {count} times</p>
    // ...
  }

  // After a click, our function is called again
  function Counter() {
    const count = 1; // Returned by useState()
    // ...
    <p>You clicked {count} times</p>
    // ...
  }

  // After another click, our function is called again
  function Counter() {
    const count = 2; // Returned by useState()
    // ...
    <p>You clicked {count} times</p>
    // ...
  }`
 ```
Khi ta update state, React gọi lại components. Mỗi lần render kết quả của `counter` trong state là một hằng số nằm trong function.

Và đoạn code dưới đây không thực hiện action data binding nào cả:
```js
  <p>You clicked {count} times</p>
```
Chỉ khi nhúng một số vào render output. Con số đó mới được gọi bởi React. Khi ta `setCount`, React gọi component lại với một giá trị `count` khác. Sau đó React update DOM để matching với lần render mới nhất.

`Key` ở đây chính là việc `count` - hằng số trong mỗi lần render không thay đổi qua thời gian. Mỗi làn component được gọi lại - là một lần render `see` được giá trị `count` khác nhau giữa các lần render. 

*3. Mỗi lần render đều có một event handler riêng*

Xem ví dụ dưới đây. Nó show alert với biến `count` sau mỗi 3s:

```js
  function Counter() {
    const [count, setCount] = useState(0);

    function handleAlertClick() {
      setTimeout(() => {
        alert('You clicked on: ' + count);
      }, 3000);
    }

    return (
      <div>
        <p>You clicked {count} times</p>
        <button onClick={() => setCount(count + 1)}>
          Click me
        </button>
        <button onClick={handleAlertClick}>
          Show alert
        </button>
      </div>
    );
  }
```
B1: Tăng counter lên 3
B2: Nhấn nút show alert
B3: Tăng lên 5 trước khi timeout được chạy

Bạn nghĩ alert sẽ show gì? Nó có show 5 - gia trị của counter lúc alert được bật. Hay là show 3 - state khi tôi click.

Nếu behaviour này không có meaning gì lắm, hãy tưởng tượng ví dụ thực tế hơn. [Bài viết này](https://overreacted.io/how-are-function-components-different-from-classes/) sẽ giải thích lý do như vậy nhưng đáp án đúng là 3. Alert sẽ `capture` lại state lúc tôi click button.

<i>(There are ways to implement the other behavior too but I’ll be focusing on the default case for now. When building a mental model, it’s important that we distinguish the “path of least resistance” from the opt-in escape hatches.)
</i>

Nhưng nó làm việc như thế nào?

Chúng ta đã bàn luận về giá trị `count` là hằng số không thay đổi trong mỗi lần gọi function. Cần nhấn mạnh lại - Function của chúng ta sẽ được gọi nhiều lần(trong mỗi lần render), nhưng mỗi một lần đó giá trị `count` sẽ không thay đổi và được set làm state trong render.

Điều đó không phải chỉ mỗi React - một function bình thường cũng làm việc tương tự:

```js
  function sayHi(person) {
    const name = person.name;
    setTimeout(() => {
      alert('Hello, ' + name);
    }, 3000);
  }

  let someone = {name: 'Dan'};
  sayHi(someone);

  someone = {name: 'Yuzhi'};
  sayHi(someone);

  someone = {name: 'Dominic'};
  sayHi(someone);
```
Trong ví dụ trên, biến outer `someone` được gán lại một vài lần. Giống như React, component sẽ có state thay đổi. Tuy nhiên, bên trong function `sayHi`, nơi có hằng số `name` - giá trị này luôn liên kết với `person` trong mỗi lần gọi. Giá trị này là local, và giá trị đó luôn độc lập trong mỗi lần call. Kết quả là, mỗi lần gọi timeout, mỗi lần alert đều có name của chính nó.

Điều này giải thích event handler capture lại giá trị `count` mỗi thời điểm click. Nếu chúng ta áp dụng nguyên tắc tương tự, mỗi lần render ta sẽ get được giá trị `count` riêng:

```js
  // During first `render
  function Counter() {
    const count = 0; // Returned by useState()
    // ...
    function handleAlertClick() {
      setTimeout(() => {
        alert('You clicked on: ' + count);
      }, 3000);
    }
    // ...
  }

  // After a click, our function is called again
  function Counter() {
    const count = 1; // Returned by useState()
    // ...
    function handleAlertClick() {
      setTimeout(() => {
        alert('You clicked on: ' + count);
      }, 3000);
    }
    // ...
  }

  // After another click, our function is called again
  function Counter() {
    const count = 2; // Returned by useState()
    // ...
    function handleAlertClick() {
      setTimeout(() => {
        alert('You clicked on: ' + count);
      }, 3000);
    }
    // ...
  }`
```
Mỗi lần render đều trả về một "version" của function `handleAlertClick`. Mỗi version đó đều có một giá trị `count` riêng.
```js
// During first render
function Counter() {
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + 0);
    }, 3000);
  }
  // ...
  <button onClick={handleAlertClick} /> // The one with 0 inside
  // ...
}

// After a click, our function is called again
function Counter() {
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + 1);
    }, 3000);
  }
  // ...
  <button onClick={handleAlertClick} /> // The one with 1 inside
  // ...
}

// After another click, our function is called again
function Counter() {
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + 2);
    }, 3000);
  }
  // ...
  <button onClick={handleAlertClick} /> // The one with 2 inside
  // ...
}

```
Đó chính là lý do trong [Demo](https://codesandbox.io/s/w2wxl3yo0l) event handlers đều thuộc về mỗi lần render riêng, và khi bạn click, nó tiếp tục sử dụng state `counter` từ lần render đó.

Trong mỗi lần render cụ thể, props và state đều không đổi. Nhưng nếu props và state bị cô lập giữa các lần render, nếu có bất cứ giá trị nào sử dụng chúng(bao gồm cả event handlers). Giá trị của chúng sẽ phụ thuộc vào từng lần render cụ thể. Và cả các function không đồng bộ trong event handlers cũng tương tự.

*4. Mỗi lần render đều có một Effects riêng*

Cùng xem ví dụ dưới đây:

```js
function Counter() {
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
*5. Câu hỏi đặt ra là, effect sẽ đọc giá trị state `count` như thế nào?*

Có phải rằng, có một thứ gọi là "data binding" hay "watching" khiến cho biến `count` update trong effect function? Có thể `count` là một biến có thể mutable mà React sets trong component và Effect có thể get được giá trị mới nhất?

<b>NOPE</b>

Chúng ta đều biết `count` là một constant trong mỗi lần component render cụ thể. Event handler sẽ "see" giá trị state `count` mỗi lần render mà chúng thuộc về, bởi vì `count` là một biến có scope của riêng nó. Effect cũng thế!

<b>Không phải `count` bằng cách nào đó thay đổi trong Effect. Mà effect function - chính nó đều khác mỗi lần render</b>

Ở mỗi version `count` đều có giá trị thuộc về lần render đó:
```js
// During first render
function Counter() {
  // ...
  useEffect(
    // Effect function from first render
    () => {
      document.title = `You clicked ${0} times`;
    }
  );
  // ...
}

// After a click, our function is called again
function Counter() {
  // ...
  useEffect(
    // Effect function from second render
    () => {
      document.title = `You clicked ${1} times`;
    }
  );
  // ...
}

// After another click, our function is called again
function Counter() {
  // ...
  useEffect(
    // Effect function from third render
    () => {
      document.title = `You clicked ${2} times`;
    }
  );
  // ..
}
```
React sẽ nhớ những function effect mà bạn viết ra, chạy nó mỗi khi flushing DOM và để browser paint ra màn hình.

So nếu chúng ta nói về khái niệm effect ở đây, nó đại diện cho từng function khác nhau trong mỗi lần render - và mỗi effect function đều "see" props và state riêng trong mỗi lần render cụ thể nào đó.

<b>Về mặt khái niệm, bạn có thể tưởng tượng effect giống như một phần kết quả của mỗi lần render</b>

Nếu nói chặt chẽ hơn, chúng không như thế. Nhưng về tư tưởng thiết kế mà chúng tôi build lên, effect function thuộc về từng lần render cụ thể - cách tương tự như event handler ở trên.

Để đảm bảo bạn đã hiểu vấn đề, hãy cùng recap first render:
* React: Give tôi UI khi state = 0 nha
* Component:
  - Đây là kết quả render đầu tiên: `<p>You clicked 0 times</p>`
  - Nhớ chạy function effect khi xong nha: `() => { document.title = 'You clicked 0 times' }`
* React: Sure. Đang update UI. Hey browser, Tôi đang add some stuff vào DOM đó.
* Browser: Cool. Tôi sẽ paint nó ra screen.
* React: OK, bây giờ tôi sẽ chạy function efect bạn đưa tớ
  - Chạy function `() => { document.title = 'You clicked 0 times' }`

  --------------------------------
Và bây giờ recap lại những gì xảy ra khi ta click:

* Component: Hey React, set state của tôi là 1 đi
* React: Hãy trả cho tôi UI khi state bằng một nhóe
* Component:
  - Đây là kết quả render: `<p>You clicked 1 times</p>`
  - Nhớ chạy function effect sau khi xong nha: `() => { document.title = 'You clicked 1 times' }`
* React: Sure. Đang update UI. Hey browser, Tôi đang add some stuff vào DOM đó.
* Browser: Cool. Tôi sẽ paint nó ra screen.
* React: OK, bây giờ tôi sẽ chạy function efect bạn đưa tớ
  - Chạy function `() => { document.title = 'You clicked 1 times' }`
  
  --------------------------------

*4. Mỗi lần render đều có mọi thứ riêng nó*
Chúng ta đều biết effect chạy sau mỗi lần render, props và state cũng sẽ phụ thuộc vào từng lần render cụ thể.

```js
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    setTimeout(() => {
      console.log(`You clicked ${count} times`);
    }, 3000);
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
Nếu tôi click một vài lần với delay một tẹo, cái gì sẽ được log ra??? Bạn có thể thử tại [Đây](https://codesandbox.io/s/lyx20m1ol)
Nếu đã thử, bạn có thể nghĩ: "Tất nhiên nó vẫn work bình thường mà nhể".

Nhưng thực sự cách nó hoạt động lại không giống như với class. Rất dễ để sai sót khi implement tương đương với class:
```js
  componentDidUpdate() {
      setTimeout(() => {
        console.log(`You clicked ${this.state.count} times`);
      }, 3000);
    }
```

Tuy nhiên, `this.state.count` luôn luôn trỏ về giá trị count latest hơn là phụ thuộc vào mỗi lần render. DO đó bạn sẽ thấy log ra giá trị `5` mỗi lần render.


Tôi nghĩ rằng, Hooks dựa rất nhiều vào cái gọi là Javascript closures, và việc implement bằng class lại gặp phải issue gọi là [setTimeout trong vòng lặp](https://wsvincent.com/javascript-closure-settimeout-for-loop/) - một example kinh điển để test các ứng viên JS về closures hehe. Điều đó bởi vì giá trị gây nhầm lẫn trong ví dụ trên là thay đổi(React mutate `this.state` trong class để update về state mới nhất) và không phải closures của chúng.

<b>Closures là một cách tuyệt vời đề khiến cho value của bạn không bao giờ thay đổi. Điều đó khiến chúng dễ dàng nhận biết bởi vì bạn đang refer tới constant</b> và như đã thảo luận, state và props never change trong mỗi lần render. Tuy nhiên, với class ta cũng có [cách giải quyết bằng closure](https://codesandbox.io/s/w7vjo07055)

*6. Quay ngược lại vấn đề*

Ở điểm này, điều quan trọng là chúng ta gọi chúng một cách rõ ràng: Mỗi function trong component render(bao gồm event handler, effects, timeout, API calls) đều capture lại props và state tại thời điểm render mà chúng được define.

và 2 ví dụ dưới đây là tương đương nhau:
```js
function Example(props) {
  useEffect(() => {
    setTimeout(() => {
      console.log(props.counter);
    }, 1000);
  });
  // ...
}
```
```js
function Example(props) {
  const counter = props.counter;
  useEffect(() => {
    setTimeout(() => {
      console.log(counter);
    }, 1000);
  });
  // ...
}
```

Không quan trọng rằng bạn get props và state sớm hay muộn trong component. Chúng không đổi! Trong mỗi scope của lần render cụ thể, props và state đều giữ nguyên.

Tất nhiên, thỉnh thoảng bạn chỉ muốn get giá trị latest hơn là giá trị được capture lại trong callback được defined trong effect. Cách dễ nhất là sử dụng refs, được hướng dẫn ở phần cuối của bài viết [này](https://overreacted.io/how-are-function-components-different-from-classes/).

Luôn aware rằng khi bạn muốn đọc "future" props và state từ function trong "past" render, bạn đang `swimming against the tide`. Điều đó không phải là sai(Nhất là một vài case cụ thể) nhưng nó có vẻ không được clean - nó phá vỡ tính thống nhất của mô hình. Cái này gọi là cố ý gây hậu quả vì chúng giúp hight light được những đoạn code đang bị `fragile` và phụ thuộc vào `timing`. Trong class, điều đó ít nhận thấy khi nó xảy ra:

Đây là ví dụ [class dùng counter](https://codesandbox.io/s/rm7z22qnlp): 
```js
  function Example() {
    const [count, setCount] = useState(0);
    const latestCount = useRef(count);

    useEffect(() => {
      // Set the mutable latest value
      latestCount.current = count;
      setTimeout(() => {
        // Read the mutable latest value
        console.log(`You clicked ${latestCount.current} times`);
      }, 3000);
    });
```

Trông có vẻ kì quặc khi mutate something trong React. Tuy nhiên, đó chính xác là những gì React reassigns lại `this.state` trong class. Không giống với props, state đã được capture lại, bạn không có gì để đảm bảo rằng `latestCount.current` sẽ đem lại cho bạn cùng một giá trị trong mỗi callback cụ thể. Theo định nghĩa, bạn có thể mutate chúng bất cứ thời gian nào. Điều đó là lý do nó không phải là default, bạn phải lựa chọn để thực hiện nó.

*7. Còn về Cleanup*
Như đã đề cập trên docs, một số effect có một cleanup phase. Cụ thể hơn, mục đích là undo lại effect ở một số case như subscriptions.

Ví dụ:
```js
useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.id, handleStatusChange);
    };
  });
```
Khi `props` là `{id: 10} trong lần render đầu tiên, và `{id: 20}` trong lần render thứ hai. Bạn có thể nghĩ something like that:
* React clean up effect cho `{id: 10}`
* React render UI cho `{id: 20}`
* React clean up effect cho `{id: 20}`

ĐIều này không đúng.

Với tư tưởng thiết kế này, bạn có thể nghĩ việc cleanup sẽ "see" props cũ bởi vì chúng chạy trước khi ta re-render, và sau đó effect mới sẽ "see" props mới bởi vì chúng chạy sau khi render. Tư tưởng đó được triển khai từ hướng class lifecycle, nhưng nó không chính xác. 

React chỉ chạy effect sau khi browser paint. Nó khiến cho app của bạn nhanh hơn như với các effect không gây block screen update. Effect cleanup cũng bị delay đi. Effect trước đó sẽ bị clean up sau khi re-render với props mới

* React render UI cho `{id: 20}`
* Browser paint. Ta sẽ thấy UI cho `{id: 20}` trên màn hình
* React cleanup effect cho `{id: 10}`
* React chạy effect cho `{id: 20}`

Bạn có thể thắc mắc bằng cách nào cleanup previous effect lại thấy được `{id: 10}` props nếu chúng đã bị change thành `{id: 20}`?

Như đã biết: 
> Every function inside the component render (including event handlers, effects, timeouts or API calls inside them) captures the props and state of the render call that defined it.

Và câu trả lời đã rõ ràng! Effect clean up không đọc `latest` props. Nó đọc props của lần render để define chính nó.

```js
/ First render, props are {id: 10}
function Example() {
  // ...
  useEffect(
    // Effect from first render
    () => {
      ChatAPI.subscribeToFriendStatus(10, handleStatusChange);
      // Cleanup for effect from first render
      return () => {
        ChatAPI.unsubscribeFromFriendStatus(10, handleStatusChange);
      };
    }
  );
  // ...
}

// Next render, props are {id: 20}
function Example() {
  // ...
  useEffect(
    // Effect from second render
    () => {
      ChatAPI.subscribeToFriendStatus(20, handleStatusChange);
      // Cleanup for effect from second render
      return () => {
        ChatAPI.unsubscribeFromFriendStatus(20, handleStatusChange);
      };
    }
  );
  // ...
}
````