---
title: A Complete Guide to useEffect
date: 2019-11-29 15:40:43
tags:
  - technical
  - Javascript
  - React Hooks
---
<p>
Bạn đã sử dụng React Hooks trong các dự án của mình. Kể cả những app nhỏ hay lớn. Hầu hết đều cảm thấy hài lòng. Bạn thoải mái với những API và có được một vài tricks để code hiệu quả hơn với Hooks. <br>Thậm chí bạn có thể viết được `custom hooks` để reuse code hiệu quả và show nó với đồng nghiệp. "Great job" - they said.</p>

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

### 1. TLDRV

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

### 2. Mỗi lần render đều có một giá trị props và state mới

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

### 3. Mỗi lần render đều có một event handler riêng

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

### 4. Mỗi lần render đều có một Effects riêng

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
*Câu hỏi đặt ra là, effect sẽ đọc giá trị state `count` như thế nào?*

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

### 5. Mỗi lần render đều có mọi thứ riêng nó

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

### 6. Còn về Cleanup
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

Điều đó cho phép React xử lý effect một cách hiệu quả sau khi painting - và giúp app nhanh hơn. Props cũ vẫn ở đó nếu đoạn code cần nó.

### 7. Đồng bộ, không phải vòng đời - lifecycle

Một trong những thứ ưa thích của tôi về React chính là nó thống nhất việc khởi tạo giá trị ban đầu. Nó giúp [reduces the entropy](https://overreacted.io/the-bug-o-notation/) cho chương trình của bạn.

Chẳng hạn component của tôi như này:

```js
  function Greeting({ name }) {
    return (
      <h1 className="Greeting">
        Hello, {name}
      </h1>
    );
  }
```

Không quan trọng rằng nếu tôi render `<Greeting name="Dan" />` và sau đó `<Greeting name="Yuzhi" />` hay nếu tôi chỉ render `<Greeting name="Yuzhi" />`. Kết quả cuối cùng, chúng ta sẽ thấy “Hello, Yuzhi” ở cả 2 trường hợp.

Người ta thường chém, Thành công là đường đi, không phải là dích đến. Điều đó là sự khác biệt giữa `$.addClass` và `$.removeClass` khi gọi Jquery code(our journey) và specifying những CSS class nào trong React code(our destination).

React đồng bộ DOM theo props và state hiện tại. Không có sự khác biệt giữa `mount` hay `update` khi rendering.

Bạn nên nghĩ về effect như vậy. `useEffect` giúp bạn đồng bộ mọi thứ bên ngoài React tree theo props và state.

```js
function Greeting({ name }) {
  useEffect(() => {
    document.title = 'Hello, ' + name;
  });
  return (
    <h1 className="Greeting">
      Hello, {name}
    </h1>
  );
}
```

Có sự khác biệt nhỏ về tư tưởng thiết kế nên mount/update/unmount. Điều đó là quan trọng để nắm rõ bản chất của nó. Nếu bạn cố gắng viết efect có behaves khác nhau phụ thuộc vào việc component renders vào lần đầu tiên hay không, bạn đang đi ngược lại nguyên lý của mô hình thiết kế Hooks. Chúng ta đang sai lầm khi đồng bộ nếu kết quả phụ thuộc vào "journey" hơn là "destination".

Không quan trọng rằng ta đã render với props A, B, và C hay nếu ta chỉ render C ngay tức thì mà thôi. Trong khi có một số sự khác biệt tạm thời(Khi call API), kết quả cuối cùng đều là như nhau.

Hơn nữa, chạy tất cả effect trong một lần render có thể không hiệu quả(và một số trường hợp nó gây ra vòng lặp vô hạn). Vậy bằng cách nào ta có thể fix nó.

### 8. Hãy chỉ cho React biết phân biệt effects

Chúng ta đã được học về DOM. Thay vì thao tác với nó mỗi lần re-render, React chỉ update những phần DOM thực sự thay đổi.

Khi bạn đang update:
```js
h1 className="Greeting">
  Hello, Dan
</h1>
```
thành:
```js
<h1 className="Greeting">
  Hello, Yuzhi
</h1>
```
React chỉ thấy 2 objects:
```js
const oldProps = {className: 'Greeting', children: 'Hello, Dan'};
const newProps = {className: 'Greeting', children: 'Hello, Yuzhi'};
```

Chúng check qua props và xác định rằng children đang thay đổi và cần update lại DOM, nhưng `className` thì không. So bạn có thể:
```js
  domNode.innerText = 'Hello, Yuzhi';
  // No need to touch domNode.className
```
ta có thể làm tương tự như trên với effect không? Có cách nào thật nai xừ để tránh việc re-render chúng khi sử dụng effect không cần thiết không?

Ví dụ, có thể component sẽ re-render vì state đã change:
```js
  function Greeting({ name }) {
    const [counter, setCounter] = useState(0);

    useEffect(() => {
      document.title = 'Hello, ' + name;
    });

    return (
      <h1 className="Greeting">
        Hello, {name}
        <button onClick={() => setCounter(count + 1)}>
          Increment
        </button>
      </h1>
    );
  }
```

Nhưng effect lại éo sử dụng state. `Effect đồng bộ document.title` với props `name`, nhưng `name` lại không đổi. Việc re-assign lại `document.title` mỗi lần counter change trông có vẻ éo hay ho lắm.

OK. So React có thề phân biệt effect chứ bồ?
```js
  let oldEffect = () => { document.title = 'Hello, Dan'; };
  let newEffect = () => { document.title = 'Hello, Dan'; };
  // Can React see these functions do the same thing?
```

Không hẳn thế. React không thể đoán được function nào nếu không gọi nó(Trong source code không chứa giá trị specific, chỉ gần với name `props` thôi).

Điều đó lý giải tại sao bạn muốn tránh chạy lại effect nếu không cần thiết, bạn có thể thêm một dependency array - gọi tắt là deps là tham số thứ 2 trong effects.
```js
useEffect(() => {
    document.title = 'Hello, ' + name;
  }, [name]); // Our deps
```

Kiểu: Êy, tôi biết bạn éo biết cái éo gì trong function, nhưng tôi hứa nó chỉ sử dụng `name` và éo có gì khác luôn trong render.

Nếu mỗi giá trị giữa các effect như nhau, sẽ không có action synchronize và React có thể skip effect đó:

```js
const oldEffect = () => { document.title = 'Hello, Dan'; };
const oldDeps = ['Dan'];

const newEffect = () => { document.title = 'Hello, Dan'; };
const newDeps = ['Dan'];

// React can't peek inside of functions, but it can compare deps.
// Since all deps are the same, it doesn’t need to run the new effect.
```
Nếu ngay cả một trong những giá trị trong `deps` khác nhau giữa các lần render, ta biết rằng effect sẽ không thể skip được. Synchronize tất cả mọi thứ!

### 9. Đừng nói dối React về dependency - Sống có tí đạo đức đê

Nếu bạn làm thế, hậu quả nghiêm cmn trọng. Trực giác cho thấy, rất nhiều thanh niên trai tráng đang cố sử dụng `useEffect` với tư tưởng từ class và cố gắng cheat game(Mịa tôi cũng cheat mấy lần nên tôi biết).

```js
function SearchResults() {
  async function fetchData() {
    // ...
  }

  useEffect(() => {
    fetchData();
  }, []); // Is this okay? Not always -- and there's a better way to write it.

  // ...
```

"Nhưng mà tui chỉ muốn nó chạy lúc mount thôi!". Hãy nhớ rằng: Nếu bạn chỉ rõ trong deps, tất cả value nằm trong component có sử dụng effect đầu phải ở đó. Bao gồm, props, state, functions - mọi thứ trong component.

Thỉnh thoảng khi bạn làm thế, nó sẽ gây ra vài vấn đề. Ví dụ, có thể bạn gặp việc fetching lặp vô hạn hay socket bị tạo thường xuyên. `Giải pháp cho vấn đề này là không remove dependency`. Ta sẽ xử nó sau, nhưng trước hết cần hiểu rõ vấn đề này trước đã.

<b>Điều gì xảy ra khi bạn ăn kem trước cổng - dependencies bạn truyền vào không đúng</b>

```js
useEffect(() => {
    document.title = 'Hello, ' + name;
  }, [name]);
```
Tất cả dependencieskhacs, nó sẽ re-run lại effect.

Nhưng khi ta để mảng rỗng, new effect sẽ không chạy. Vì éo có gì khác nhau á.

Trong case này, vấn đề rất rõ ràng. Nhưng trực giác có thể làm bạn bị đui ở những trường hợp nơi mà giải pháp dùng class nhảy tung tăng trong đầu bạn.

Ví dụ, ta đang viết một counter tăng mỗi giây. Với class, ta thường làm kiểu: Set up interval một lần và destroy nó đi. Khi ta convert nó sang `useEffect`, theo bản năng ta thêm vào deps một empty array `[]` với thông điệp: Tao muốn nó chạy 1 lần thôi. OK?

 ```js
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);

  return <h1>{count}</h1>;
}
```
Nếu tư tưởng của bạn là deps giúp tôi đặt điều kiện để chạy effect, và bạn muốn nó chạy một lần vì nó là interval. Và tại sao nó lại gây ra vấn đề?

Tuy nhiên, điều này khá make sense nếu bạn biết rằng deps là gợi ý của ta với React về tất cả mọi thứ mà effect sử dụng từ render scope. Nó sử dụng `count` nhưng ta đã lie rằng nó không đi cùng `[]`.

Trong lần render đầu, `count` là 0. Theo đó, `setCount(count+1)` trong effect của render đầu nghĩa là `setCount(0+1)`. Từ đó, ta sẽ không re-run effect lần éo nào nữa vì đã cho deps là mảng rỗng rồi, nó sẽ gọi `setCount(0+1)` mỗi giây:

```js
// First render, state is 0
function Counter() {
  // ...
  useEffect(
    // Effect from first render
    () => {
      const id = setInterval(() => {
        setCount(0 + 1); // Always setCount(1)
      }, 1000);
      return () => clearInterval(id);
    },
    [] // Never re-runs
  );
  // ...
}

// Every next render, state is 1
function Counter() {
  // ...
  useEffect(
    // This effect is always ignored because
    // we lied to React about empty deps.
    () => {
      const id = setInterval(() => {
        setCount(1 + 1);
      }, 1000);
      return () => clearInterval(id);
    },
    []
  );
  // ...
}
```
Ta đã nói dối React về việc effect không phụ thuộc vào một giá trị nằm trong component, khi mà thực sự là có. Effect sử dụng `count` - một giá trị nằm trong component(nhưng nằm outside effect):
```js
  const count = // ...

  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);
  ```
  Do đó, việc chỉ rõ deps là `[]` sẽ tạo bug. React sẽ so sánh deps và bỏ qua việc update effect này.

  Issues kiểu này khá là khó để nhận ra. Do đó, tôi khuyến khích anh em luôn tỉnh táo trước các thế lực thù địch, phải khai đúng cái gì cần được cho vào deps. Chúng tôi đã thêm ESlint cho cái rule này.

### 10. 2 điều thực sự đúng về dependencies

Có hai chiến lược chính về deps. Bạn nên bắt đầu bằng cách 1, sau đó có thể áp dụng cách 2 nếu cần.

Chiến lược 1 là fix mảng deps include tất cả những value có trong component sử dụng trong effect. Let's do it:
```js
  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1);
    }, 1000);
    return () => clearInterval(id);
  }, [count]);
```
Điều đó giúp deps array chính xác hơn. Nó có thể không hẳn là lý tưởng nhưng đó là issue đầu tiên ta cần giải quyết. Bây giờ một sự thay đổi `count` sẽ re-run lại effect, với mỗi interval tiếp theo sẽ referencing `count` từ render trong `setCount(count+1)`:
```js
// First render, state is 0
function Counter() {
  // ...
  useEffect(
    // Effect from first render
    () => {
      const id = setInterval(() => {
        setCount(0 + 1); // setCount(count + 1)
      }, 1000);
      return () => clearInterval(id);
    },
    [0] // [count]
  );
  // ...
}

// Second render, state is 1
function Counter() {
  // ...
  useEffect(
    // Effect from second render
    () => {
      const id = setInterval(() => {
        setCount(1 + 1); // setCount(count + 1)
      }, 1000);
      return () => clearInterval(id);
    },
    [1] // [count]
  );
  // ...
}
```
Điều đó có thể fix được vấn đề, nhưng interval sẽ bị clear và set lại mỗi khi `count` changes. ĐÓ là điều không mong muốn.

<b>Chiến lược thứ 2 là thay đổi code trong effect để nó không cần value thay đổi thường xuyên như ta muốn nữa</b>. Ta không muốn nói dối về deps - ta chỉ thay đổi effect để phụ thuộc ít hơn vào chúng.

Lét go:

### 11. Viết hàm effect độc lập

Chúng ta đều muốn bỏ biến `count` trong deps:
```js
  useEffect(() => {
      const id = setInterval(() => {
        setCount(count + 1);
      }, 1000);
      return () => clearInterval(id);
    }, [count]);
```
để làm diều đó, ta cần tự hỏi: Chúng ta sử dụng `count` để làm gì? Có vẻ như ta chỉ sử dụng nó với `setCount`. Trong case đó, ta không phải lúc nào cũng cần `count` trong scope. Khi chúng ta muốn update state dựa vào previous state, chúng tôi có thể sử dụng [functional updater form](https://reactjs.org/docs/hooks-reference.html#functional-updates) trong setState.

```js
  useEffect(() => {
      const id = setInterval(() => {
        setCount(c => c + 1);
      }, 1000);
      return () => clearInterval(id);
    }, []);
```
Tôi coi những case như này là "false dependencies". Vâng, `count` là một deps cần thiết vì ta viết `setCount(count +1)` trong effect. Tuy nhiên, ta chỉ thực sự cần `count` để transform nó thành `count + 1` và "send it back" to React. Nhưng React cũng biết `count` hiện tại là bao nhiêu. <strong>Tất cả chúng ta cần là nói cho React tăng state lên - làm bất cứ điều gì ngay bây giờ </strong>

*Hãy nhớ ta đang thực hiện việc remove deps. Ta không cheat game. Effect function không đọc value của `count` từ render scope nữa*

Ngay cả khi function effect chỉ chạy 1 lần, interval callback ở lần render đầu tiên là đủ thông tin khi trả về update instruction `c = c + 1` mỗi khi interval được fires. Không cần thiết để biết giá trị `counter` trong state hiện tại là bao nhiêu. React biết điều đó.

### 12. Functional update và Google docs

Hãy nhớ ta đã nói về đồng bộ - synchronization là tư thưởng thiết kế cho effect? Một khía cạnh thú vụ của đồng bộ là bạn thường muốn keep lại "message" giữa các lần `systems untangled` từ state của chúng. Ví dụ, việc editing một document trên GOogle docs không gửi cả page tới server. Điều đó rõ ràng là không hiệu quả. Thay vào đó, chúng chỉ gửi một bản miêu tả của những điều mà user đang thử làm.

Trong thực tế các use case là khác nhau, nhưng chúng đều áp dụng một nguyên lý chung cho effects. Chúng giúp cho ta chỉ gửi một phần thông tin nhỏ bên trong effect tới component. Updater form kiểu `setCount(c => c + 1)` truyền đạt thông tin chặt chẽ hơn là `setCount(count + 1)` vì nó không bị "tainted" - "liên quan" bởi giá trị `count` hiện tại. Hãy nghĩ về việc React đang xử lý theo hướng - tìm một minimal state. Chúng đều có nguyên lý giống nhau, nhưng cho update.

Việc Encoding những ý định/mục tiêu (hơn là kết quả) cũng giống như cách Google docs giải quyết vấn đề collaborative editing. While this is stretching the analogy, functional update có vai trò tương tự như trong React. Chúng đảm bảo việc update từ nhiều nguồn khác nhau(event handler, effect subscriptions, etc) có thể được áp dụng chính xác trong một lần(in a batch) và trong một hướng có thể dự đoán được.

Tuy nhiên, ngay cả việc dùng `setCount(c => c + 1)` cũng không hẳn là hay. Nó trông có chút không tự nhiên, và bị giới hạn những gì có thể làm. Ví dụ, nếu ta có 2 biến state có giá trị phụ thuộc lẫn nhau, hay nếu ta cần tính toán state tiếp theo dựa vào props, điều này không giúp ta giải quyết được. May mắn thay, `setCount(c => c + 1)` có một pattern mạnh mẽ hơn. Chúng gọi là `useReducer`.

### 12A. Decoupling Updates from Actions - Tách update ra trong actions. 

Hãy sửa đoạn code trên thành 2 biến state:`count` và `step`. Function interval lúc này sẽ tăng count bằng giá trị của `step`:
```js
function Counter() {
  const [count, setCount] = useState(0);
  const [step, setStep] = useState(1);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + step);
    }, 1000);
    return () => clearInterval(id);
  }, [step]);

  return (
    <>
      <h1>{count}</h1>
      <input value={step} onChange={e => setStep(Number(e.target.value))} />
    </>
  );
}
```
Hãy nhớ rằng ta không cheat. Từ khi tôi bắt đầu sử dụng `step` bên trong effect, tôi sẽ add thêm nó vào deps. Và điều đó giúp code chạy chính xác.

Behaviour hiện tại của example này là thay đổi `step` sẽ restart lại interval - vì nó là một trong các dependencies. Và trong nhiều trường hợp, điều đó đúng với những gì bạn cần! Chả có gì sai nếu bạn chia nhỏ effect và set up một cái mới, và chúng ta không cần tránh điều đó nếu như không có lí do.

Tuy nhiên, hãy nói về việc ta muốn interval không reset khi thay đổi `step`. Bằng cách nào ta có thể remove được `step` trong dependency của effect?

<strong>Khi set up một biến state phụ thuộc vào giá trị hiện tại của biến state khác, bạn có thể thử dùng chúng với `useReducer`</strong>

Khi bạn viết `setSomething(something => ...)`, đó là thời điểm tốt để sử dụng một reducer để thay thế. Một reducer cho phép bạn chia nhỏ "actions" xảy ra trong component theo cách mà state update để phản hồi lại action đó.

Cùng đổi `step` bằng `dispatch` trong effect:
```js
const [state, dispatch] = useReducer(reducer, initialState);
const { count, step } = state;

useEffect(() => {
  const id = setInterval(() => {
    dispatch({ type: 'tick' }); // Instead of setCount(c => c + step);
  }, 1000);
  return () => clearInterval(id);
}, [dispatch]);
```
Bạn có thể hỏi tôi: "Nó tốt hơn như thế nào?" Câu trả lời là <strong>React đảm bảo rằng `dispatch` function luôn không đổi trong suốt lifetime của component. So ví dụ trên sẽ không cần việc resubscribe lại interval</strong>. Và chúng ta đã giải quyết được bài toán.

Bạn có thể omit `dispatch`, `setState`, `useRef` container values từ deps vì React đảm bảo chúng là static. Nhưng nó cũng không làm sao nếu ta modify chúng.

THay vì đọc state trong effect, nó sẽ dispatch một action mã hóa thông tin về `what happened`. Điều đó cho phép effect được chia nhỏ ra từ `step` state. Effect không care việc bạn update state, nó chỉ nói cho chúng ta `what happened`. Và reducer là nơi tập trung logic update đó:
```js
  const initialState = {
    count: 0,
    step: 1,
  };

  function reducer(state, action) {
    const { count, step } = state;
    if (action.type === 'tick') {
      return { count: count + step, step };
    } else if (action.type === 'step') {
      return { count, step: action.step };
    } else {
      throw new Error();
    }
  }
```
### 14. Tại sao useReducer là Cheat mode của Hooks?

Chúng ta đã thấy cách remove deps khi một effect caaf setstate dựa vào state trước đó, hay dựa vào biến state khác. Nhưng sẽ là gì nếu ta cần props để tính toán trong state tiếp theo? Ví dụ, có thể API là `<Counter step={1} />`. Chắc chắn trong case này ta không thể tránh khỏi việc sử dụng props.step trong deps.?

Thực tế là có thể. Ta có thể truyền reducer vào trong component để đọc props.
```js
function Counter({ step }) {
  const [count, dispatch] = useReducer(reducer, 0);

  function reducer(state, action) {
    if (action.type === 'tick') {
      return state + step;
    } else {
      throw new Error();
    }
  }

  useEffect(() => {
    const id = setInterval(() => {
      dispatch({ type: 'tick' });
    }, 1000);
    return () => clearInterval(id);
  }, [dispatch]);

  return <h1>{count}</h1>;
}
```
Pattern này sẽ disable một số optimization, do đó đừng cố gắng sử dụng chúng ở mọi nơi, nhưng bạn có thể access vào props từ reducer nếu cần.

Ngay cả trong những trường hợp `dispatch` chính nó vẫn là stable giữa quá trình re-render. Bạn có thể omit nó từ efect deps nếu cần. Nó không gây ra việc effect re-run.

Bạn có thể thắc mắc: Làm sao nó có thể làm việc? Bằng cách nào reducer biết được props khi chúng được gọi bên trong effect ở lần render khác?
Câu trả lời là khi bạn `dispatch`, React chỉ nhớ actions, nhưng nó sẽ gọi reducer trong lần render tiếp theo. Khi đó, props mới sẽ nằm trong scope và bạn sẽ không ở trong effect.

Điều đó giải thích tại sao tôi cho rằng `useReducer` là cheat mode của Hooks. Nó cho phép ta chia nhỏ logic update những thay đổi đang xảy ra. Điều đó, giúp tôi remove được những deps không cần thiết trong effect và tránh việc re-run chúng nhiều lần.

### 15. Chuyển function vào effects

Một lỗi thường gặp là nghĩ rằng function không phải là dependencies. VÍ dụ đoạn code dưới này vẫn chạy ngon:
```js
function SearchResults() {
  const [data, setData] = useState({ hits: [] });

  async function fetchData() {
    const result = await axios(
      'https://hn.algolia.com/api/v1/search?query=react',
    );
    setData(result.data);
  }

  useEffect(() => {
    fetchData();
  }, []); // Is this okay?

  // ...
```
Đoạn code trên vẫn chạy. Nhưng vấn đề với việc omit local function là nó khó để biết chúng ta có xử lý được tất cả các trường hợp hay không khi component grows.

Hãy tưởng tưởng code của ta sẽ chia nhỏ và mỗi function to gấp 5 lần: 
```js
function SearchResults() {
  // Imagine this function is long
  function getFetchUrl() {
    return 'https://hn.algolia.com/api/v1/search?query=react';
  }

  // Imagine this function is also long
  async function fetchData() {
    const result = await axios(getFetchUrl());
    setData(result.data);
  }

  useEffect(() => {
    fetchData();
  }, []);

  // ...
}
```
Bây giờ, ta sẽ sử dụng một số state và props trong một trong những function:
```js
function SearchResults() {
  const [query, setQuery] = useState('react');

  // Imagine this function is also long
  function getFetchUrl() {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }

  // Imagine this function is also long
  async function fetchData() {
    const result = await axios(getFetchUrl());
    setData(result.data);
  }

  useEffect(() => {
    fetchData();
  }, []);

  // ...
}
```

Nếu ta quên update deps, effects sẽ không đồng bộ những thay đổi từ props và state. Điều đó không hay lắm.

May mắn thay, có một easy solution cho problem này. Nếu bạn chỉ sử dụng một số function trong effect, chuyển nó hẳn vào effect:
```js
  function SearchResults() {
    // ...
    useEffect(() => {
      // We moved these functions inside!
      function getFetchUrl() {
        return 'https://hn.algolia.com/api/v1/search?query=react';
      }
      async function fetchData() {
        const result = await axios(getFetchUrl());
        setData(result.data);
      }

      fetchData();
    }, []); // ✅ Deps are OK
    // ...
  }
```

Lợi ích của điều này là gì? Ta không cần phải suy nghĩ nhiều về việc bắc cầu - transitive dependencies. Mảng deps không bao giờ nói dối: Ta thực sự không sử dụng thứ gì từ outer scope của component trong effect.

Nếu sau đó ta có edit function `getFetchUrl` để sử dụng `query` state, ta nên nhớ rằng ta đang sửa nó trong effect - do đó, ta cần add query vào deps:
```js
function SearchResults() {
  const [query, setQuery] = useState('react');

  useEffect(() => {
    function getFetchUrl() {
      return 'https://hn.algolia.com/api/v1/search?query=' + query;
    }

    async function fetchData() {
      const result = await axios(getFetchUrl());
      setData(result.data);
    }

    fetchData();
  }, [query]); // ✅ Deps are OK

  // ...
}
```

Bằng cách add thêm deps, ta không chỉ "appeasing React". Điều đó giống với việc refetch lại data khi query thay đổi. Thiết kế của `useEffect` buộc bạn phải nhớ sự thay đổi của data flow và chọn cách mà effect đồng bộ nó - thay vì bỏ qua nó đến khi dính bug ở prod.

Hiện tại có lint rules là exhaustive-deps từ plugin `eslint-plugin-react-hooks`, với nó bạn có thể phân tích được effect và nhận được suggest những deps nào đang miss. Nói cách khác, lint sẽ nói với bạn data flow nào thay đổi mà không được xử lý chính xác.

<strong>Nhưng tôi có thể put function vào trong effect</strong>
Thỉnh thoảng bạn không muốn move function vào bên trong effect. Ví dụ, một vài effect trong cùng components có thể gọi chung một function, và bạn không muốn copy paste logic. Hay có thể là prop.

Bạn có nên bỏ qua function như vậy trong deps? Tôi nghĩ là không. Effect không nói dối về deps của bạn. Luôn có những solution hay hơn. Một quan niệm sai lầm là một function không bao giờ thay đổi. Nhưng ta đã học được rằng, nó không phải lúc nào cũng đúng. THay vào đó, một function được define trong một component có thể thay đổi qua các lần render.
Và đó là vấn đề. Ví dụ 2 effect cùng gọi `getFetchUrl`:
```js
function SearchResults() {
  function getFetchUrl(query) {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }

  useEffect(() => {
    const url = getFetchUrl('react');
    // ... Fetch data and do something ...
  }, []); // 🔴 Missing dep: getFetchUrl

  useEffect(() => {
    const url = getFetchUrl('redux');
    // ... Fetch data and do something ...
  }, []); // 🔴 Missing dep: getFetchUrl

  // ...
}
```

Trong trường hợp này, bạn không muốn move `fetFetchUrl` vào trong effect nếu bạn không định dùng chung logic.

Nói cách khác, đây là vấn đề. Nếu cả 2 effect đều phụ thuộc vào một deps, mảng dependency sẽ là vô dụng:
```js
function SearchResults() {
  // 🔴 Re-triggers all effects on every render
  function getFetchUrl(query) {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }

  useEffect(() => {
    const url = getFetchUrl('react');
    // ... Fetch data and do something ...
  }, [getFetchUrl]); // 🚧 Deps are correct but they change too often

  useEffect(() => {
    const url = getFetchUrl('redux');
    // ... Fetch data and do something ...
  }, [getFetchUrl]); // 🚧 Deps are correct but they change too often

  // ...
}
```

Một giải pháp hay ho là skip `getFetchUrl` function trong deps list. Tuy nhiên tôi không nghĩ đó là vấn đề hay. Nó làm ta khó nhận biết việc khi nào data flow thay đổi cần được handle trong effect. Dẫn đến issues kiểu không update lại interval như đã nói ở trên:

THay vào đó, có 2 solution đơn giản hơn.

Đầu tiên, nếu một function không sử dụng bất cứ gì từ component scope, bạn có thể hoist nó ra ngoài component và sử dụng thoải mái trong effect:
```js
// ✅ Not affected by the data flow
function getFetchUrl(query) {
  return 'https://hn.algolia.com/api/v1/search?query=' + query;
}

function SearchResults() {
  useEffect(() => {
    const url = getFetchUrl('react');
    // ... Fetch data and do something ...
  }, []); // ✅ Deps are OK

  useEffect(() => {
    const url = getFetchUrl('redux');
    // ... Fetch data and do something ...
  }, []); // ✅ Deps are OK

  // ...
}
```

Không cần phải truyền nó vào deps vì nó không nằm trong render scope và không chịu ảnh hưởng bởi data flow. Nó không phụ thuộc vào state hay props. 

Cách hay ho khác là ta có thể dùng nó với `useCallback`:
```js
unction SearchResults() {
  // ✅ Preserves identity when its own deps are the same
  const getFetchUrl = useCallback((query) => {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }, []);  // ✅ Callback deps are OK

  useEffect(() => {
    const url = getFetchUrl('react');
    // ... Fetch data and do something ...
  }, [getFetchUrl]); // ✅ Effect deps are OK

  useEffect(() => {
    const url = getFetchUrl('redux');
    // ... Fetch data and do something ...
  }, [getFetchUrl]); // ✅ Effect deps are OK

  // ...
}
```
`useCallback` giống như một layer khác của dependency. Nó giải quyết vấn đề theo cách khác - đó là thay vì tránh dùng function trong deps, ta khiến cho function đó chỉ thay đổi khi cần thiết.

Hãy xem tại sao cách tiếp cận này là hiệu quả. Trước đây, example của ta đang thể hiện 2 kết quả search(cho `react` và `redux`). Nhưng ta muốn add một input để có thể search query tùy ý. Thay vì dùng `query` như một argument - đối số(đối số hay argument là giá trị truyền vào khi gọi hàm còn tham số hay paramenter là giá trị khi định nghĩa một hàm), `getFetchUrl` bây giờ sẽ đọc nó ra từ local state.

Ta sẽ thấy ngay việc miss `query` deps:
```js
  function SearchResults() {
    const [query, setQuery] = useState('react');
    const getFetchUrl = useCallback(() => { // No query argument
      return 'https://hn.algolia.com/api/v1/search?query=' + query;
    }, []); // 🔴 Missing dep: query
    // ...
  }
```

Nếu tôi fix `useCallback` deps - thêm vào `query`, bất cứ effect nào với `getFetchUrl` sẽ re-run khi `query` thay đổi.

```js
  function SearchResults() {
    const [query, setQuery] = useState('react');

    // ✅ Preserves identity until query changes
    const getFetchUrl = useCallback(() => {
      return 'https://hn.algolia.com/api/v1/search?query=' + query;
    }, [query]);  // ✅ Callback deps are OK

    useEffect(() => {
      const url = getFetchUrl();
      // ... Fetch data and do something ...
    }, [getFetchUrl]); // ✅ Effect deps are OK

    // ...
  }
```

Với `useCallback`, nếu `query` không đổi, `getFetchUrl` cũng không đổi, effect sẽ không chạy lại. Nhưng nếu `query` thay đổi, `getFetchUrl` sẽ thay đổi và khi đó ta sẽ re-fetch data. Nó khá giống việc ta sửa một cell nào trong spreadsheet excel, cell khác cũng tự động tính toán lại.

Đó chỉ là kết quả khi ta nắm gọn data flow và tư duy đồng bộ. Giải pháp tương tự đều work với function prop khi pass từ parents:

```js
function Parent() {
  const [query, setQuery] = useState('react');

  // ✅ Preserves identity until query changes
  const fetchData = useCallback(() => {
    const url = 'https://hn.algolia.com/api/v1/search?query=' + query;
    // ... Fetch data and return it ...
  }, [query]);  // ✅ Callback deps are OK

  return <Child fetchData={fetchData} />
}

function Child({ fetchData }) {
  let [data, setData] = useState(null);

  useEffect(() => {
    fetchData().then(setData);
  }, [fetchData]); // ✅ Effect deps are OK

  // ...
}
```

`fetchData` chỉ thay đổi trong `Parent` khi `query` state change, `Child` sẽ không refetch data khi không cần thiết.

### 15. Function có phải là một phần của data flow?

Thật thú vị, pattern này bị phá vỡ bởi cách dùng class khi thể hiện sự khác nhau giữa effect và lifecycle paradigms. Nếu dùng class:
```js
class Parent extends Component {
  state = {
    query: 'react'
  };
  fetchData = () => {
    const url = 'https://hn.algolia.com/api/v1/search?query=' + this.state.query;
    // ... Fetch data and do something ...
  };
  render() {
    return <Child fetchData={this.fetchData} />;
  }
}

class Child extends Component {
  state = {
    data: null
  };
  componentDidMount() {
    this.props.fetchData();
  }
  render() {
    // ...
  }
}
```

Bạn có thể nghĩ rằng: thôi nào `Dan`, ta đều biết rằng `effect` work tương tự như `componentDidMount` hay `componentDidUpdate` combined vs nhau thôi,. Yep nhưng nó không làm việc ngay với cả `componentDidUpdate`:

```js
  class Child extends Component {
    state = {
      data: null
    };
    componentDidMount() {
      this.props.fetchData();
    }
    componentDidUpdate(prevProps) {
      // 🔴 This condition will never be true
      if (this.props.fetchData !== prevProps.fetchData) {
        this.props.fetchData();
      }
    }
    render() {
      // ...
    }
  }
```

Rõ ràng `this.props.fetchData` luôn bằng `prevProps.fetchData` và ta sẽ không refetch. Hay remove nó thử:
```js
componentDidUpdate(prevProps) {
    this.props.fetchData();
  }
```
Nhưng nếu làm thế này thì cứ lúc nào render thì nó đều fetchdata. Thử bind query vào và check thử xem:
```js
render() {
    return <Child fetchData={this.fetchData.bind(this, this.state.query)} />;
  }
```

Tuy nhiên điều kiện `this.props.fetchData !== prevProps.fetchData` luôn luôn đúng ngay cả khi `query` không change. Do đó ta luôn refetch.

Chỉ có một giải pháp khi dùng class là pass query vào `Child` component. `Child` không sử dụng `query` nhưng nó có thể trigger refetch khi thay đổi. 
```js
class Parent extends Component {
  state = {
    query: 'react'
  };
  fetchData = () => {
    const url = 'https://hn.algolia.com/api/v1/search?query=' + this.state.query;
    // ... Fetch data and do something ...
  };
  render() {
    return <Child fetchData={this.fetchData} query={this.state.query} />;
  }
}

class Child extends Component {
  state = {
    data: null
  };
  componentDidMount() {
    this.props.fetchData();
  }
  componentDidUpdate(prevProps) {
    if (this.props.query !== prevProps.query) {
      this.props.fetchData();
    }
  }
  render() {
    // ...
  }
}
```
Qua một thời gian dài làm việc với class trong React, tôi đã quen với việc pass những props không cần thiết và phá vỡ tính đóng gói của parent component mà tôi chỉ nhận ra một tuần trước đây khi làm nó.

<strong>Với classes, function props - chính nó không thực sự là một phần của data flow.</strong>. Những methods đều dùng cách sửa đổi biến `this` do đó ta không thể dựa vào chúng để nắm được những thứ khác. Vì vậy, ngay cả khi ta chỉ muốn sử dụng function, ta phải pass những data mục đích để so sánh chúng như `query` ở trên. Ta không biết liệu rằng `this.props.fetchData` pass từ parent có phụ thuộc vào state hay không, liệu nó có change hay không. 

> Đây cũng là một trong những issues người viết băn khoăn nhất trong hơn 2 năm làm việc với React lifecycle.

Với `useCallback`, function có thể hoàn toàn tham gia vào data flow. Chúng ta có thể nói rằng nếu input vào function thay đổi, function sẽ thay đổi, nhưng nếu không, chúng sẽ giữ nguyên. Cảm ơn sự "granularity" mà `useCallback` đem lại, những thay đổi props như `props.fetchData` có thể tự động được đẩy lên.

TƯơng tự, `useMemo` giúp ta xử lý những object phức tạp:
```js
  function ColorPicker() {
    // Doesn't break Child's shallow equality prop check
    // unless the color actually changes.
    const [color, setColor] = useState('pink');
    const style = useMemo(() => ({ color }), [color]);
    return <Child style={style} />;
  }
```

<strong>Tôi muốn nhấn mạnh rằng việc sử dụng `useCallback` mọi nơi trông sẽ rất vụng</strong>. Nó không phải là lối thoát tuyệt vời và hữu dụng khi một function phải pass down và được gọi từ trong effect của một số children. Hay nếu bạn thử prevent việc memoization của child component. Nhưng hook cho chúng một cách [avoiding passing callbacks down](https://reactjs.org/docs/hooks-faq.html#how-to-avoid-passing-callbacks-down).

Ở ví dụ trên, tôi thích hơn nhiều nếu `fetchData` trong effect(nó có thể extract bằng cusom hooks) hay import từ top level. Tôi muốn giữ cho effect đơn giản nhất có thể, và callback không giúp ta điều đó.(Sẽ có gì xảy ra nếu `props.onComplete` callback thay đổi khi request đang được gọi?)

### 16. Race conditions:

Một ví dụ fetching data dùng class kiểu:
```js
  class Article extends Component {
    state = {
      article: null
    };
    componentDidMount() {
      this.fetchData(this.props.id);
    }
    async fetchData(id) {
      const article = await API.fetchArticle(id);
      this.setState({ article });
    }
    // ...
  }
```

Như bạn đã biết, đoạn code trên có bug. Nó không handle việc update. Ví dụ thứ hai là ví dụ bạn có thể tìm thấy trên mạng:
```js
class Article extends Component {
  state = {
    article: null
  };
  componentDidMount() {
    this.fetchData(this.props.id);
  }
  componentDidUpdate(prevProps) {
    if (prevProps.id !== this.props.id) {
      this.fetchData(this.props.id);
    }
  }
  async fetchData(id) {
    const article = await API.fetchArticle(id);
    this.setState({ article });
  }
  // ...
}
```

Đoạn code trên có vẻ ổn, nhưng vẫn có thể dính bug. Nguyên nhân chính là request có thể đến sau việc order. Tức là khi tôi đang fetching `{id: 10}`, thì tôi chuyển mợ sang `{id: 20}`, nhưng request của `{id: 20}` lại đến đầu, và request bắt đầu sớm hơn nhưng lại kết thúc muộn hơn có thể ghi đè state.

Đó gọi là race condition, và nó là điển hình cho việc mix `async/await`(giả lập rằng thứ gì đó đang chờ kết quả trả về) với `top-down` data flow(props hay state có thể thay đổi khi chúng ta đang ở giữa async function).

Effect không giải quyết vấn đề này một cách kì diệu, mặc dầu chúng sẽ cảnh báo bạn nếu bạn thử pass trực tiếp một async function vào effect.
Nếu cách tiếp cận async bạn sử dụng support cancel, tuyệt vời. Bạn có thể cancel request async khi cleanup function.

CÓ một lựa chọn, cách dễ nhất là dùng điểm dừng để track với một biến boolean:
```js
  function Article({ id }) {
    const [article, setArticle] = useState(null);

    useEffect(() => {
      let didCancel = false;

      async function fetchData() {
        const article = await API.fetchArticle(id);
        if (!didCancel) {
          setArticle(article);
        }
      }

      fetchData();

      return () => {
        didCancel = true;
      };
    }, [id]);

    // ...
  }
```
[Bài viết này](https://www.robinwieruch.de/react-hooks-fetch-data) sẽ đi vào chi tiết cách bạn có thể handle err hay loading state, hay có thể extract logic ra custom hooks. Tôi recommend bạn check it out.

> Đây cũng là một trong những issues người viết băn khoăn nhất trong hơn 2 năm làm việc với React lifecycle. Có giải pháp là dùng biến isMounted nhưng trông code không clean. Còn giải pháp dùng promise thì lại phức tạp hơn. Đây cũng là một issuse trên github mà ngay tại thời điểm dịch cũng chưa có cách giải quyết thật sự hợp lý khi dùng class.

### 17. Nâng tầm - Raising the Bar:

Với tư duy class lifecyle, side effects sẽ có behaviour khác khi render. Rendering UI được định hướng bởi props và state, và có đảm bảo sự thống nhất giữa chúng, nhưng side effects lại không. Đó thường là ngọn nguồn của bug.

Với mindset `useEffect`, mọi thứ được đồng bộ theo mặc định. Side effect trở thành một phần của data flow trong React. Với mỗi lần gọi `useEffect`, một khi bạn làm đúng, component của bạn sẽ handle được những case khù khoằm tốt hơn. 

Tuy nhiên, điều đó tốn nỗ lực hơn. Chúng có thể annoying. Viết code đồng bộ để có thể handles nhiều case vốn khó hơn là firing một side effects không thống nhất trong render.

Điều này có thể cần xem xét nếu `useEffect` là tool mà bạn sẽ sử dụng trong hầu hết mọi thời điểm. Tuy nhiên, nó ở tầng thấp. Nó có thể được demo rất dễ theo dạng tutorial. Trên thực tế, có vẻ cộng đồng sẽ bắt đầu chuyển sang dạng Hooks ở level cao hơn như là API tốt hơn kiểu như custom hooks.

Tôi đã thấy nhiều app tạo ra những custom hooks riêng kiểu `useFetch` đóng gói một số logic phân quyền hay `useTheme` sử dụng theme context. Một khi bạn có một toolbox, bạn sẽ không phụ thuộc quá nhiều vào `useEffect` nữa. Nhưng, tất cả đều được base trên nó cả.

Hơn nữa, `useEffect` là một common case để fetch data. Nhưng data fetching không phải chính xác là vấn đề đồng bộ. Nó có vẻ rõ ràng vì bạn phải xử lý deps. Vậy những gì mà chúng ta đang đồng bộ trong một React app?

Trong thời gian tới, `Suspense for Data fetching` sẽ cho phép thư viện từ bên thứ 3 có thể giao tiếp với React để suspend rendering cho đến khi mọi thứ async(mọi thứ ở đây là code, data, images) đã sẵn sàng.

Suspense dần sẽ cover được nhiều use case data fetching hơn, tôi dự đoán rằng `useEffect` sẽ quay về phía background như là một tool vô cùng mạnh mẽ cho bạn khi đồng bộ props và state. Không giống data fetching, nó handle những case thông thường vì nó được design để làm điều đó. Nhưng cho đến khi đấy, custom Hooks vẫn là một cách hay để reuse lại logic data fetching.

### Tổng kết:

Bây giờ bạn đã biết khá nhiều về useEffect. Nếu không hiểu thì vui lòng xem lại phần dịch ở trên hoặc bài viết gốc. Have a good day! 

Happy hacking!