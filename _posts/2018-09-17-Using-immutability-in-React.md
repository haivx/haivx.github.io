---
title: Using immutability in React
date: 2018-09-17 14:23:37
tags:
  - technical
  - ReactJS
---

Bài viết được dịch từ [blog http://reactkungfu.com](http://reactkungfu.com/2015/08/pros-and-cons-of-using-immutability-with-react-js/)
React không chỉ là công nghệ sử dụng để tạo ra giao diện người dùng có tính tương tác cao. Chắc chắn rằng một điều, tính flexible của React cực kì cao, nó không quan tâm tới cách bạn tổ chức cấu trúc front end application như thế nào.

Nhưng điều đó không có nghĩa là không có ý tưởng hay guideline nào giúp bạn tạo ra một app hoàn chỉnh cho mình.

Có nhiều ý tưởng và công nghệ được áp dụng hiệu quả vơi ReactJS. Một trong những idea đó là tính bất biến của dữ liệu - data immutability....
<!-- more -->
Nó đến từ thế giới của FP- functional programming - và được áp dụng để design ra một app frontend với nhiều điểm benefit.

## I. What is it?

Hãy xem ví dụ sau:
```js
  var x = 12,
      y = 12;

  var object = { x: 1, y: 2 };
  var object2 = { x: 1, y: 2 };
```
như bạn thấy, **object** và **object2** có cùng values. Nếu nhìn vào mỗi key và value tương ứng của 2 object này, chúng là như nhau. Nhưng nếu bạn check điều kiện equal thì kết quả lại khác biệt:

```js
  object == object2 // false
  object === object2 // fals
```

Tại sao vậy? Có `HAI` cách để so sánh "bằng”:

  - So sánh qua reference - reference equality
  - So sánh qua value - value equality

## II. So sánh qua reference

Object là một cấu trúc data structure hơi phức tạp. Nó có thể có nhiều key, những key này chỉ về những value tùy ý. Những value đó có thể là object. Vì vậy object có thể nested-lồng object trong object.

Nếu tôi đưa ra một ví dụ, bạn có thể nghĩ về những chiếc xe. Hãy tưởng tượng bạn có một chiếc xe màu đỏ, hàng xóm của bạn cũng có chiếc xe đó. Chúng cùng màu, cùng hãng, cùng công nghệ. Nếu một người lạ cho nhận xét: Chúng đều như nhau.

Nhưng, bạn biết rằng xe của ông hàng xóm không phải là xe của bạn. Of course!

Thực tế, Javascript object có khái niệm gọi là reference - địa chỉ vùng nhớ của object trên bộ nhớ.
> Khi bạn so sánh hai objects trong JS, nó được so sánh bằng reference. Đây còn gọi là tham chiếu
```js
    object == object2;
    object === object2;
```

Có phải **object** giống **object2**? Thực tế, khi bạn asign **object** bằng **object2**, tức là bạn đã assign reference của **object** tới **object2**.

Và kết quả là ta lại có example sau:

```js
    object = object2;
    object.x = 12;

    object.x; // 12
    object2.x; // 12
```

Mỗi data structure phức tạp trong JS đều follow theo nguyên tắc so sánh reference này. Nó bao gồm `lists(arrays)` và `object`. Thực tế `lists` cũng là một `object` hay nói cách khác, `với JS, Object là tất cả`.

```js
  typeof([1,2,3]) // "object"
```

Một số giá trị nguyên thủy như numbers, strings, booleans, null, undefined. Chúng đều tuân theo kiểu so sánh gọi là so sánh bằng giá trị.

## III. So sánh qua giá trị.

Với những giá trị nguyên thủy, chúng không thể nested được. Những cấu trúc không thể nest trong cấu trúc khác được gọi là shallow data structures. Và chúng check theo kiểu tham trị:

```js
    var x = 12,
    y = 12;
```

Để có thể so sánh hai object nested phức tạp, tác giả có dùng thuật toán đệ quy để so sánh từng cặp key-value của object. Bài dịch không đi tiếp vấn đề này.

## IV. React làm gì với ý tưởng này?

React component có một concept gọi là state. Mỗi khi state changes, component sẽ re-render lại.

Với state có cấu trúc phức tạp(nested object), rất là khó để có thể check được liệu rằng state có change hay không. Bạn có thể tự check các thuộc tính trong object đó - `deep equality`. Đơn giản vì với kiểu check dùng cách so sánh reference thì bạn không thể biết được state có change hay không.

Vì vậy, nếu bạn optimize React để nó chỉ update khi nào cần thiết, bạn cần hàng nghìn lần check value. Rất là tiêu tốn perfomance. Tất nhiên, `keep state minimal` sẽ là best practice cho bạn, nhưng state component lại tương ứng với số lượng phần tử mà chúng hiển thị. Tức là không tránh khỏi việc phải tính toán so sánh - và với app lớn điều đó tốn khá nhiều **performance**.

Những rắc rối mà React cần phải giải quyết chính là với JS bạn có thể mutate object. Và việc mutate đó càng gây confuse khi so sánh.

<strong>Mutations</strong>
Sửa đổi data structure là một điều bình thường trong **imperative programming languages** như JS.

> Most programmers thinks it’s the only way to work with your objects - object-oriented programming is all about changing state using messages, right?

Khi bạn thay đổi object, reference của nó vẫn không đổi. Đơn giản là nếu bạn sơn xe ô tô từ màu đỏ sang màu xanh, thì ô tô đó vẫn là của bạn, chỉ có màu là thay đổi.
```js
    var yourCar = {
        color: 'red',
        .. the same as neighboursCar
    };

    var neighboursCar = {
        color: 'red',
        ... the same as yourCar
    };

    valueEqual(yourCar, neighboursCar); // true;
    yourCar === neighboursCar; // false

    yourCar.color = 'blue';

    valueEqual(yourCar, neighboursCar); // false;
    yourCar === neighboursCar; // false
```
Như vậy, **mutations không change kết quả của check references**. Nó chỉ **thay đổi kết quả của value checks mà thôi**.

> Sự thật là React không cần quan tâm chính xác những gì đang change. Nó chỉ quan tâm tới việc liệu state có change hay không.

Và *immutability* không giúp trả lời câu hỏi: `what exactly changed` mà nó cung cấp cho bạn câu trả lời tuyệt vời cho câu hỏi: `is it changed at all or not`.

<strong>Từ bỏ mutations và hãy sử dụng immutability</strong>

Tính toàn vẹn dữ liệu là ý tưởng mà bạn có thể apply vào code của mình ở bất cứ ngôn ngữ nào. Có thể tóm tắt nó như sau:
> Whenever your object would be mutated, don’t do it. Instead, create a changed copy of it.

Và với JS có những cách giúp bạn implement theo hướng immutability:

  - Sử dụng Object.assign({}, …)
  - Sử dụng [].concat
  - Sử dụng spread operator

## V. Pros and Cons: Sẽ được nói thêm ở những bài viết sau

Be good. Sleep well. And enjoy coding.
