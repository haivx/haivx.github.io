---
title: React lifecycle purecomponent and its problems
date: 2018-09-05 14:36:54
tags:
  - technical
  - ReactJS
---
Bài viết là những lượm lặt từ trang chủ React.

## Constructor
> Nếu bạn không cần khởi tạo state hoặc bind method thì không cần phải implement constructor cho React component

<!-- more -->

Constructor trong React Component được gọi trước khi component được mount. Trong các constructor của subclass extends từ React.Component, ta cần call **super(props)** trước khi sử dụng. Nếu không, this.props sẽ trả về undefined.

Thường thì React constructor chỉ có 2 nhiệm vụ chính:

  - Khởi tạo local state và assign tới this.state
  - Binding event handler methods tới instance

1.<strong>Set State:</strong>

Không nên call setState() trong constructor. Thay vì đó, nếu comnponent cần sử dụng local state, asign state khởi tạo trực tiếp tới this.state trong construtor.
```js
    constructor(props) {
        super(props);
        // Don't call this.setState() here!
        this.state = { counter: 0 };
        this.handleClick = this.handleClick.bind(this);
    }
```
Tránh sử dụng các action **side-effects** hay **subscriptions** trong constructor. Thay vì sử dụng ở đó, hãy dùng componentDidMount.

2. <strong>Lỗi thường gặp</strong>
```js
    constructor(props) {
        super(props);
        // Don't do this!
        this.state = { color: props.color };
    }
```

Chỉ sử dụng cách trên nếu bạn MUỐN bỏ qua việc props update. Trong case này, nó sẽ coi như là việc mình set defaultColor. Bạn có thể bắt buộc component reset internal state bằng change key của nó nếu cần.

## II. componentDidMount()

Hook này được gọi ngay khi component được mount(DOM đc insert vào tree node). Những công việc khởi tạo cần có DOM nodes nên được set tại đây.

  - có thể set subscription tại hook này
  - call API tại hook này

1. <strong>Set State</strong>
Có thể dùng setState tại đây. Nó sẽ trigger một hành động render page lần nữa, nhưng nó sẽ xảy ra trước khi browser update. Điều này đảm bảo việc render gọi hai lần, và người dùng không thấy được sự thay đổi. Tuy nhiên nó thường gây issues perfomance.

Nó có thể nên dùng khi cần thiết, trong những case như modals hoặc tooltips khi bạn cần tính toán DOM node trước khi render component phụ thuộc vào size hay position.

## III. componentDidUpdate()

> componentDidUpdate(prevProps, prevState, snapshot). Hook này được gọi sau khi component đã update.

Nó có thể sử dụng trong trường hợp call API dựa vào việc compare curent props và previous props.
```js
    componentDidUpdate(prevProps) {
        // Typical usage (don't forget to compare props):
        if (this.props.userID !== prevProps.userID) {
            this.fetchData(this.props.userID);
        }
    }
```

1. <strong>Set State</strong>

Có thể gọi setState tuy nhiên cần check điều kiện(chẳng hạn ví dụ trên) nếu không nó sẽ gây ra vòng lặp vô hạn.

## IV. componentWillUnmount()

Dùng để clean các method đã set ở đầu lifecycle như cancel request API, clean subscribtion, clear timeout…

1. <strong>Set state</strong>
 Tất nhiên là không vì component sẽ không re-render lần nào nữa.

## V. shouldComponentUpdate() and its problems
> This method only exists as a performance optimization. Do not rely on it to “prevent” a rendering, as this can lead to bugs.

Nên dùng `pureComponent` để `shallow compare` props và state thay vì compare bằng tay.

Nếu tự tin bạn có thể compare bằng tay. Tuy nhiên việc return false không prevent việc child component re-render khi state changes.

Không khuyến khích check deep shallow như **JSON.stringify()** vì chúng không hiệu quả và gây performance issues.

**React.PureComponent** sử dụng shallow comparison. Do đó, nó cũng ko giải quyết được vấn đề trong trường hợp data structures phức tạp. Bạn cần hiểu rõ shallow compare như thế nào. Và đây là ví dụ:
```js
    class ListOfWords extends React.PureComponent {
        render() {
            return <div>{this.props.words.join(',')}</div>;
        }
    }

    class WordAdder extends React.Component {
        constructor(props) {
            super(props);
            this.state = {
            words: ['marklar']
            };
            this.handleClick = this.handleClick.bind(this);
        }

        handleClick() {
            // This section is bad style and causes a bug
            const words = this.state.words;
            words.push('marklar');
            this.setState({words: words});
        }

        render() {
            return (
            <div>
                <button onClick={this.handleClick} />
                <ListOfWords words={this.state.words} />
            </div>
            );
        }
    }
```

–> Vấn đề ở đây là việc push value vào mảng mặc dù value của state - là object mới có thay đổi tuy nhiên React lại so sánh bằng shallow compare. Tức với object nó chỉ so sánh reference thôi chứ không so sánh value của properties.

Do đó React xem như 2 object đó là như nhau. Vậy nên, tránh dùng method push ở trường hợp này nhé.

### <strong>The Power Of Not Mutating Data:</strong>
> The simplest way to avoid this problem is to avoid mutating values that you are using as props or state.
```js
    handleClick() {
        this.setState(state => ({
            words: state.words.concat(['marklar'])
        }));
    }
```

hoặc
```js
    handleClick() {
        this.setState(state => ({
            words: [...state.words, 'marklar'],
        }));
    };
```

Một ví dụ setState() trong demo [Stocklist](https://github.com/haivx/stocklist);

Bạn cũng nên viết code theo phong cách avoid mutation. Ví dụ thay vì:
```js
    function updateColorMap(colormap) {
        colormap.right = 'blue';
    }
```
Ta lại dùng:
```JS
    function updateColorMap(colormap) {
        return Object.assign({}, colormap, {right: 'blue'});
    }
```

Hoặc
```js
    function updateColorMap(colormap) {
        return {...colormap, right: 'blue'};
    }
```

## Câu hỏi đặt ra: Tại sao phải dùng cách tiếp cận IMMUTABLE:

React so sánh state bằng shallow compare, nếu state có structure phức tạp, có nested object, hoặc được modify đi thì object mới và object cũ đều có chung reference. Và kết quả là React coi như chúng như nhau. Và không được update.

Vì vậy, cần phải tạo ra một object mới khi setState, khi tạo object mới, reference tất nhiên là mới. Và React hiểu bạn đang set lại State mới => Nó sẽ update.

Be good. Sleep well. And enjoy coding.
