---
title: Writing resilient components - Viết component có tính mềm dẻo - linh hoạt cao.
date: 2019-12-05 15:40:43
tags:
  - technical
  - Javascript
  - React Hooks
---

Khi một ai đó bắt đầu code React, họ sẽ tìm kiếm những style guide. Bên cạnh việc chúng - style guide là một ý kiến hay để xây dựng được tính thống nhất trong một project, có nhiều thứ cần lưu ý, tuân thủ theo - điều đó khiến cho React trở nên thiếu tính flexible và strong opinion.

Bài viết được lược dịch từ Blog [overreacted.io](https://overreacted.io/writing-resilient-components/)

<!-- more -->
Bạn có thể sử dụng nhiều type system khác nhau, thích sử dụng function declaration hay arrow function, sort props theo thứ tự alphabe hay theo những gì bạn thích.

Tính linh hoạt này cho phép [tích hợp React](https://reactjs.org/docs/add-react-to-a-website.html) vào một project bất kì với những conventions có sẵn. Nhưng nó cũng tạo ra nhiều tranh cãi không hồi kết.

Có những nguyên tắc design quan trọng mà mỗi component cần phải nỗ lực để follow theo. Nhưng tôi không nghĩ style guide sẽ sao chép/capture những nguyên tắc này hiệu quả. Chúng ta sẽ nói về style guide đầu tiên, và sau đó cùng xem những nguyên tắc nào thực sự có ích.

----------------------------------

### Đừng bị mất tập trung bởi những vấn đề trong tưởng tượng.

Trước khi chúng ta nói về nguyên tắc thiết kế component, tôi muốn nói một chút về những style guides. Chúng không phải là những ý tưởng/quan điểm phổ biến nhưng ai đó cần phải nói lên diều đó.

Trong cộng đồng Javascript, có một số style guides có quan điểm chặt chẽ về linter. Theo quan điểm của tôi, chúng có lại tạo ra nhiều sự khó chịu/friction hơn là giá trị đem lại. Tôi không thể đếm được bao nhiêu lần một ai đó show cho tôi đoạn code valid tuyệt đối và nói rằng "React đang phàn nàn về đoạn code này", nhưng thực ra nó chỉ là do linter show cảnh báo/error. Điều này dẫn tới 3 issues:

* Mọi người xem linter trở thành kẻ gác cồng ồn ào hơn là một công cụ hữu hiệu. Những dòng cảnh báo có ích bị chìm nghỉm bởi một loạt style guide. Và kết quả là, người ta không đọc message từ linter khi debugging, và bỏ quên đi những tips có ích. Thêm vào đó, nhiều người ít khi phải code Javascript(như design chẳng hạn) lại càng khó khăn hơn khi làm việc với những đoạn code này.

* Người ta không học được sự khác nhau giữa những use case hợp lệ và không hợp lệ của một pattern cụ thể. Ví dụ, có một rule khá phổ biến cấm gọi `setState` trong `componentDidMount`. Nhưng nếu nó luôn luôn là ý kiến "tồi", thì React đã cấm tiệt nó rồi! Có một số case hợp lệ cho nó, đó là việc tính toán DOM node layout - ví dụ vị trí của tooltip. Chúng tôi đã thấy nhiều người "work around" rule này bằng cách add thêm `setTimeout` mà bỏ đi điểm mấu chốt này.

* Cuối cùng, người ta thường theo tư duy thi hành và chấp nhận những quan điểm về những thứ không đem lại ý nghĩa khác biệt nhưng lại dễ dàng tìm thấy trong code. "Bạn sử dụng một declare function, nhưng project của ta lại sử dụng arrow functions". Bất cứ khi nào tôi cảm thấy có sự gượng ép bởi những rule như thế, xem xét kĩ càng, sâu hơn và tôi đã phải quyết định bỏ qua nó. Nó dụ tôi có được cảm giác thành công khi giải quyết được một linter khó nhằn nhưng thực ra éo nâng cao được chất lượng dòng code tí nào.

Và tôi đang nói về việc bỏ hết cmn linting đi? Free style đi? Không hẳn thế!

Với một config tốt, một linter sẽ là tool hữu hiệu để bắt bugs trước khi chúng kịp xuất hiện. Việc tập trung vào style quá nhiều chỉ làm sao nhãng hơn mà thôi.

----------------------------------
### Marie Kondo Your Lint Config

ở đây là những gì tôi suggest bạn làm vào thứ 2 đầu tuần. Tập trung team bạn lại nửa giờ, lướt qua mỗi lint rule được bật trong project config, và tự hỏi bản thân: Đoạn rule này có giúp chúng ta bắt bug không? Nếu không, bỏ chúng đi(Bạn có thể bắt đầu một config mới toanh bằng `eslint-config-react-app` với không một styling rule nào).

Ít nhất, team của bạn có một tiến trình loại bỏ những rule không cần thiết. Đừng có tưởng rằng bất cứ cái gì bạn hay ai đó add vào config lint một năm qua là `best practice`. Hãy đặt câu hỏi về nó và tìm câu trả lời. Đừng để ai đó nói với bạn rằng bạn không đủ thông minh để chọn lint rule - cay vãi.

Nhưng formatting thì sao? Hãy sử dụng `Prettier` và quên mợ một mớ bòng bong "style nits" đi. Bạn không cần một tool hét vào mặt bạn rằng: êu, thêm 1 dấu cách, êu, thêm dấu chấm phẩy cuối dòng - nếu thay vào đó bạn chỉ cần một tool giúp bạn fix cmn code tự dộng. Sử dụng linter để tìm bugs, không phải để hoa mỹ màu mè. OK?

Tất nhiên, có nhiều khía cạnh về coding style không trực tiếp liên quan đến format nhưng nó có thể gây ra rắc rối khi không thống nhất được trong dự án.

Tuy nhiên, nhiều style guide quá tinh để bắt được những lint rule. Đó là lý do tại sao nó quan trọng để xây dựng sự tin tưởng giữa các member của team, và để chia sẻ những kiến thức hữu ích dưới dạng form của wiki page hay một đoạn design guide ngắn.

Không phải mọi thứ đều nhất thiết phải tự động hóa! Việc có tầm nhìn/ hiểu biết thực thụ lý do tại sao style guide đưa ra sẽ là giá trị hơn là chỉ follow theo những rule đó.

Nhưng nếu của phải follow theo style guide chặt chẽ một cách mê muội như vậy, điều gì thực sự quan trọng ở đây?

ĐÓ là mục tiêu của bài post này.

----------------------------------

### Writing Resilient Components

Không phải thụt đầu dòng, hay sort theo thứ tự alphabe có thể sửa được một lỗi thiết kế. Do dó thay vì tập trung vào xem đoạn code nó trong như thế nào, tôi tập trung vào tìm hiểu đoạn code nó chạy như thế nào. Có một số nguyên tắc thiết kế mà tôi cho rằng rất hữu ích:
1. Không dừng data flow
2. Luôn luôn sẵn sàng để render
3. Không component nào là nguyên mẫu
4. Giữ cho data ở local là độc lập

Ngay cả khi bạn không sử dụng React, bạn sẽ nhận ra những nguyên tắc tương tự khi thử nghiệm và sai lầm cho bất cứ một thiết kế UI component theo mô hình data flow một chiều nào - unidirectional data flow.


----------------------------------

### Nguyên tắc 1: Không dừng data flow.

Khi ai đó sử dụng component của bạn, họ mong đợi rằng có thể pass những props khác nhau vào đó, và component sẽ thay đổi theo đó:
```js
// isOk might be driven by state and can change at any time
<Button color={isOk ? 'blue' : 'red'} />
```

Thông thường, đó là điều React work default. Nếu bạn sử dụng color `props` trong `Button` component, bạn sẽ thấy giá trị được cung cấp khi render:
```js
function Button({ color, children }) {
  return (
    // ✅ `color` is always fresh!
    <button className={'Button-' + color}>
      {children}
    </button>
  );
}
```

Tuy nhiên, một lỗi thường xuyên gặp khi học React là copy props vứt vào state:
```js
class Button extends React.Component {
  state = {
    color: this.props.color
  };
  render() {
    const { color } = this.state; // 🔴 `color` is stale!
    return (
      <button className={'Button-' + color}>
        {this.props.children}
      </button>
    );
  }
}
```

Điều này nhìn có vẻ trực quan nếu bạn đã sử dụng class ngoài code React. Nhưng, việc copy prop vào state đã khiến bạn bỏ qua việc update nó.
```js
// 🔴 No longer works for updates with the above implementation
<Button color={isOk ? 'blue' : 'red'} />
```

Trong một vài case hiếm gặp, cách này là có chủ ý, do đó cần chắc chắn có thể gọi props `initialColor` hay `defaultColor` để sáng tỏ được việc có update được color hay không.

Nhưng thường bạn sẽ muốn đọc prop trực tiếp từ component của bạn và tránh copy props(hay bất cứ thứ gì cần tính toán từ props) vào state:
```js
function Button({ color, children }) {
  return (
    // ✅ `color` is always fresh!
    <button className={'Button-' + color}>
      {children}
    </button>
  );
}

```

Những value được tính toán là nguyên nhân khác mà người ta thỉnh thoảng lại thử để copy props vào state. Ví dụ, tưởng tượng chúng ta đang xác định màu text của button dựa vào việc tính toán lằng nhằng với màu nền trong tham số truyền vào:
```js
class Button extends React.Component {
  state = {
    textColor: slowlyCalculateTextColor(this.props.color)
  };
  render() {
    return (
      <button className={
        'Button-' + this.props.color +
        ' Button-text-' + this.state.textColor // 🔴 Stale on `color` prop updates
      }>
        {this.props.children}
      </button>
    );
  }
}
```

Component trên có bug nếu nó không tính toán lại `this.state.textColor` khi `color` prop thay đổi. Cách dễ để fix là chuyển `textColor` sang render method và sử dụng purecomponent:
```js
class Button extends React.PureComponent {
  render() {
    const textColor = slowlyCalculateTextColor(this.props.color);
    return (
      <button className={
        'Button-' + this.props.color +
        ' Button-text-' + textColor // ✅ Always fresh
      }>
        {this.props.children}
      </button>
    );
  }
}
```
Vấn đề đã được giải quyết. Bây giờ nếu props change, ta sẽ tính toán lại `textColor`, nhưng ta lại không tránh được những tính toán phức tạp với cùng một props.

Tuy nhiên, nếu muốn optimize chúng tốt hơn. Sẽ là gì nếu `children` prop thay đổi? Nó sẽ giống kiểu không may khi `textColor` phải tính toán lại. Cách giải quyết thứ 2 của chúng ta nhằm mục đích optimize lại nó:
```js
  class Button extends React.Component {
    state = {
      textColor: slowlyCalculateTextColor(this.props.color)
    };
    componentDidUpdate(prevProps) {
      if (prevProps.color !== this.props.color) {
        // 😔 Extra re-render for every update
        this.setState({
          textColor: slowlyCalculateTextColor(this.props.color),
        });
      }
    }
    render() {
      return (
        <button className={
          'Button-' + this.props.color +
          ' Button-text-' + this.state.textColor // ✅ Fresh on final render
        }>
          {this.props.children}
        </button>
      );
    }
  }
```

Tuy nhiên, người ta thường thêm những side effect vào đó nữa. Và điều đó thường gây ra những vấn đề sẽ được giải quyết ỏ tính năng Concurrent Rendering sắp release tới. Và có method "getDerivedStateFromProps" nhìn có vẻ dị hơn.

Hãy cùng quay lại phần trên. Để hiệu quả hơn, ta sử dụng `memoization`. Tuy nhiên, Hooks đã có bước đi vượt trội hơn, cung cấp cho chúng ta những cách built-in để memoize những tính toán phức tạp:
```js
function Button({ color, children }) {
  const textColor = useMemo(
    () => slowlyCalculateTextColor(color),
    [color] // ✅ Don’t recalculate until `color` changes
  );
  return (
    <button className={'Button-' + color + ' Button-text-' + textColor}>
      {children}
    </button>
  );
}
```

Đó là những gì bạn cần! Trong class component, bạn có thể sử dụng helper kiểu thư viện `memoize-one` để làm điều đó. Trong function component, `useMemo` từ Hook đưa cho bạn tính năng tương tự.

Bây giờ, ta có thể thấy việc optimize những tính toán phức tạp không phải là lý do chính để copy props vào state. Kết quả của hàm render nên phụ thuộc vào sự thay đổi của props.

----------------------------------

### Không dừng flow data trong side effects

Đi sâu hơn nữa, chúng ta sẽ nói về việc giữ sự thống nhất về render result khi prop thay đổi. Tránh copy prop vào state là một phần của điều đó. Tuy nhiên, điều quan trọng là side effects(như data fetching chẳng hạn) cũng là một phần của data flow.

Xem ví dụ:
```js
  class SearchResults extends React.Component {
    state = {
      data: null
    };
    componentDidMount() {
      this.fetchResults();
    }
    fetchResults() {
      const url = this.getFetchUrl();
      // Do the fetching...
    }
    getFetchUrl() {
      return 'http://myapi/results?query' + this.props.query;
    }
    render() {
      // ...
    }
  }
```

Rất nhiều component React khá tương tự như trên - nhưng nếu ta xem kĩ hơn, ta sẽ thấy có bug. `fetchResults` method sử dụng `query` prop cho việc fetch data:
```js
getFetchUrl() {
    return 'http://myapi/results?query' + this.props.query;
  }
```
Nhưng sẽ như thế nào nếu `query` prop thay đổi? Trong components, không có gì thay đổi cả. Điều đó có nghĩa là side effect trong component của chúng ta không tuân theo sự thay đổi của props. Đây thường là ngọn nguồn của bug trong React app.

Để fix điều này, ta cần:

* Xem trong `componentDidMount` và những method được gọi trong đó
  * Trong ví dụ của ta, đó là `fetchResults` và `getFetchUrl`.
* Viết tất cả prop và state được sử dụng bởi tất cả những method đó:
  * Trong ví dụ của chúng ta, đó chính là `this.props.query`.
* Đảm bảo bất cứ khi nào prosp thay đổi, ta sẽ re-run lại side effect:
  * Ta có thể làm điều đó bằng cách add thêm vào `componentDidUpdate`:

```js
class SearchResults extends React.Component {
  state = {
    data: null
  };
  componentDidMount() {
    this.fetchResults();
  }
  componentDidUpdate(prevProps) {
    if (prevProps.query !== this.props.query) { // ✅ Refetch on change
      this.fetchResults();
    }
  }
  fetchResults() {
    const url = this.getFetchUrl();
    // Do the fetching...
  }
  getFetchUrl() {
    return 'http://myapi/results?query' + this.props.query; // ✅ Updates are handled
  }
  render() {
    // ...
  }
}
```
Bây giờ code của ta sẽ tuân thủ sự thay đổi của props, kể cả side effects. Tuy nhiên, sẽ có case làm điều đó bị phá vỡ. Ví dụ, ta có thể add thêm `currentPage` vào local state và sử dụng nó trong `getFetchUrl`:
```js
class SearchResults extends React.Component {
  state = {
    data: null,
    currentPage: 0,
  };
  componentDidMount() {
    this.fetchResults();
  }
  componentDidUpdate(prevProps) {
    if (prevProps.query !== this.props.query) {
      this.fetchResults();
    }
  }
  fetchResults() {
    const url = this.getFetchUrl();
    // Do the fetching...
  }
  getFetchUrl() {
    return (
      'http://myapi/results?query' + this.props.query +
      '&page=' + this.state.currentPage // 🔴 Updates are ignored
    );
  }
  render() {
    // ...
  }
}
```
Code của ta lại có bug vì side effect không tuân thủ sự thay đổi của `currentPage`.

Props và state là một phần của React data flow. Cả việc rendering lẫn side effect đều ảnh hưởng tới data flow, không dược bỏ qua nó.

Để fix điều này, ta có thể lặp lại các bước trên:

* Xem trong `componentDidMount` và những method được gọi trong đó
  * Trong ví dụ của ta, đó là `fetchResults` và `getFetchUrl`.
* Viết tất cả prop và state được sử dụng bởi tất cả những method đó:
  * Trong ví dụ của chúng ta, đó chính là `this.props.query` và `this.state.currentPage`.
* Đảm bảo bất cứ khi nào prosp thay đổi, ta sẽ re-run lại side effect:
  * Ta có thể làm điều đó bằng cách add thêm vào `componentDidUpdate`:
```js
lass SearchResults extends React.Component {
  state = {
    data: null,
    currentPage: 0,
  };
  componentDidMount() {
    this.fetchResults();
  }
  componentDidUpdate(prevProps, prevState) {
    if (
      prevState.currentPage !== this.state.currentPage || // ✅ Refetch on change
      prevProps.query !== this.props.query
    ) {
      this.fetchResults();
    }
  }
  fetchResults() {
    const url = this.getFetchUrl();
    // Do the fetching...
  }
  getFetchUrl() {
    return (
      'http://myapi/results?query' + this.props.query +
      '&page=' + this.state.currentPage // ✅ Updates are handled
    );
  }
  render() {
    // ...
  }
}
```
Có tốt không nếu bằng cách nào đó ta có thể bắt được lỗi này một cách tự động? Có linter nào có thể bắt được lỗi này cho chúng ta?

----------------------------------

Thật không may, tự động checking tính thống nhất của một class component là quá khó. Một method có thể được gọi bởi nhiều method khác. Việc phân tích tĩnh những lần gọi từ `componentDidMount` và `componentDidUpdate` thường có rất nhiều kết quả sai.

Tuy nhiên, có một thiết kế API có thể phân tích được tính thống nhất này. Đó chính là `useEffect` của React Hooks:

```js
function SearchResults({ query }) {
  const [data, setData] = useState(null);
  const [currentPage, setCurrentPage] = useState(0);

  useEffect(() => {
    function fetchResults() {
      const url = getFetchUrl();
      // Do the fetching...
    }

    function getFetchUrl() {
      return (
        'http://myapi/results?query' + query +
        '&page=' + currentPage
      );
    }

    fetchResults();
  }, [currentPage, query]); // ✅ Refetch on change

  // ...
}
```

Chúng ta truyền logic vào effect, và điều đó trở nên dễ dàng hơn khi thấy những giá trị nào từ React data flow mà nó phụ thuộc. Những giá trị đó gọi là dependencies và trong ví dụ trên là `[currentPage, query]`

Để ý rằng mảng trên không phải là concept gì mới mẻ. Trong class, ta có thể search những "dependencies" đó thông qua tất cả những method được gọi. `useEffect` API chỉ làm cho concept đó trở nên rõ ràng hơn mà thôi.

Hãy nhớ rằng quan trọng là bạn phải tuân thủ tất cả props và state update cho effect mà bạn đang viết cho dù là class component hay function component.

Với class API, bạn phải tự mình tìm được tính thống nhất, xác nhận sự thay đổi thông qua prop, state được handle bởi `componentDidUpdate`. Mặt khác, component của bạn không có tính mềm dẻo với sự thay đổi của props và state. Điều đó không chỉ là vấn đề của React. Nó áp dụng cho tất cả những thư viện UI nào cho phép bạn xử lý "creation" và "updates" riêng biệt.

`useEffect` API đảo lộn mặc định đó bằng cách khuyến khích tính thống nhất. Nó có thể hơi lạ một chút ở những lần đầu, nhưng kết quả là component của bạn trở nên resilient hơn với những thay đổi của logic. Và từ khi những"dependencies" trở nên rõ ràng, ta có thể xác định được effect một cách thống nhất bằng lint rule. Chúng ta đang sử dụng linter để bắt bug.

----------------------------------

### Không dừng data flow khi tối ưu

Không có case nào mà bạn bỏ qua sự thay đổi props một cách ngẫu nhiên cả. NHững sai lầm này thường xảy ra khi bạn optimize component bằng tay.

Hãy nhớ cách tiếp cận optimization này sử dụng shallow equality như `PureComponent` hay `React.Memo` để so sánh.

Tuy nhiên, nếu bạn thử optimize một component bằng cách tự so sánh, bạn có thể gặp lỗi khi compare props:
```js
class Button extends React.Component {
  shouldComponentUpdate(prevProps) {
    // 🔴 Doesn't compare this.props.onClick 
    return this.props.color !== prevProps.color;
  }
  render() {
    const onClick = this.props.onClick; // 🔴 Doesn't reflect updates
    const textColor = slowlyCalculateTextColor(this.props.color);
    return (
      <button
        onClick={onClick}
        className={'Button-' + this.props.color + ' Button-text-' + textColor}>
        {this.props.children}
      </button>
    );
  }
}
```

Dễ dàng để gặp phải lỗi này vì với class, bạn thường phải pass function xuống component con, và chúng thường như nhau:
```js
class MyForm extends React.Component {
  handleClick = () => { // ✅ Always the same function
    // Do something
  }
  render() {
    return (
      <>
        <h1>Hello!</h1>
        <Button color='green' onClick={this.handleClick}>
          Press me
        </Button>
      </>
    )
  }
}
```
Do đó việc optimize của bạn không break mọi thứ ngay tức khắc. Tuy nhiên, nó vẫn có thể thấy ngay bằng giá trị `onClick` cũ nếu nó thay đổi nhưng props khác lại không:
```js
class MyForm extends React.Component {
  state = {
    isEnabled: true
  };
  handleClick = () => {
    this.setState({ isEnabled: false });
    // Do something
  }
  render() {
    return (
      <>
        <h1>Hello!</h1>
        <Button color='green' onClick={
          // 🔴 Button ignores updates to the onClick prop
          this.state.isEnabled ? this.handleClick : null
        }>

          Press me
        </Button>
      </>
    )
  }
}
```

Trong ví dụ này, click vào button sẽ disable nó - nhưn nó không xảy ra vì `Button` component bỏ qua update `onClick` prop.

Điều đó có thể trở nên rối rắm hơn nếu function được xác định dựa vào thứ gì đó có thể thay đổi theo thời gian, như `draft.content`:
```js
   drafts.map(draft =>
    <Button
      color='blue'
      key={draft.id}
      onClick={
        // 🔴 Button ignores updates to the onClick prop
        this.handlePublish.bind(this, draft.content)
      }>
      Publish
    </Button>
  )
```