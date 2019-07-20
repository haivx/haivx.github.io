---
title: 'Polyfills: Everything you ever wanted to know, or maybe a bit less'
date: 2018-09-18 15:40:43
tags:
  - technical
  - Javascript
---

Công nghệ web ngày nay càng ngày càng trở nên mạnh mẽ. Bạn có thể sử dụng **promises**, **fetch**, **arrow function**, và **const** lẫn **let** - tất cả feature này đều rất mới - và nó chạy được trên nhiều trình duyệt. Không cần polyfills, không cần transpiling, nó chạy tốt.

Điều đó rất tuyệt vời, nhưng nó cũng không phải lúc nào cũng thích hợp...
<!-- more -->
Bởi vì, một số user sử dụng những trình duyệt thế hệ cũ như IE, UC browser, hay Safari. Và những magical things trên lại không hoạt động :D.

Đối mặt với hiện thực rằng, bạn không thể lúc nào cũng sử dụng modern code - tính năng JS mới nhất và mong đợi rằng tất cả người dùng đều sử dụng được, tất cả người dùng đều update phiên bản trình duyệt mới nhất, hot nhất, bạn có chính xác 2 lựa chọn:
- Chỉ sử dụng những tính năng JS có thể chạy được trên tất cả các loại trình duyệt
- Sử dụng những tính năng mới nhất để code, code đẹp, hiệu quả. Và làm cách gì đó để nó chạy được trên các browser cũ.

Nếu bạn chọn cách 1, tôi tin rằng bạn có lý do của riêng mình. Bài viết này sẽ nói về cách giúp modern code chạy được trên non-modern browsers.

### <strong>Transpiling vs polyfilling</strong>

Trong một thời gian dài, tôi không phân biệt được sự khác nhau giữa transpiling và polyfilling. Hãy để tôi giải thích:

<strong>Transpiling</strong>(Chuyển đổi)
Một transpiler lấy `syntax` mà những browser đời cũ không hiểu được(như classes, const, arrow function), sau khi `xào nấu`, nó trả về `syntax` mà những browser này hiểu được.

Xem ví dụ:
```js
  class BetterArray {
    constructor(arr) {
      this.arr = arr;
    }
    has(item) {
      return this.arr.includes(item);
    }
  }

  const myArray = new BetterArray([1, 2, 3]);

  console.log(myArray.has(2)); // true
  console.log(myArray.has(4)); // false
```
Sau khi nó sử dụng transpiler như Babel, kết quả trả về sẽ như sau:
```js
  "use strict";

  var _createClass = function () { function defineProperties(target, props) { for (var i = 0; i < props.length; i++) { var descriptor = props[i]; descriptor.enumerable = descriptor.enumerable || false; descriptor.configurable = true; if ("value" in descriptor) descriptor.writable = true; Object.defineProperty(target, descriptor.key, descriptor); } } return function (Constructor, protoProps, staticProps) { if (protoProps) defineProperties(Constructor.prototype, protoProps); if (staticProps) defineProperties(Constructor, staticProps); return Constructor; }; }();

  function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }

  var BetterArray = function () {
    function BetterArray(arr) {
      _classCallCheck(this, BetterArray);

      this.arr = arr;
    }

    _createClass(BetterArray, [{
      key: "has",
      value: function has(item) {
        return this.arr.includes(item);
      }
    }]);

    return BetterArray;
  }();

  var myArray = new BetterArray([1, 2, 3]);

  console.log(myArray.has(2)); // true
  console.log(myArray.has(4)); // false
```
> Có thể check trực tiếp tại [đây](https://babeljs.io/repl/#?babili=false&evaluate=true&lineWrap=false&presets=es2015%2Creact%2Cstage-2%2Cstage-3&code=class%20BetterArray%20%7B%0A%20%20constructor%28arr%29%20%7B%0A%20%20%20%20this.arr%20%3D%20arr%3B%0A%20%20%7D%0A%20%20has%28item%29%20%7B%0A%20%20%20%20return%20this.arr.includes%28item%29%3B%0A%20%20%7D%0A%7D%0Aconst%20myArray%20%3D%20new%20BetterArray%28%5B1%2C%202%2C%203%5D%29%3B%0Aconsole.log%28myArray.has%282%29%29%3B%20%2F%2F%20true%0Aconsole.log%28myArray.has%284%29%29%3B%20%2F%2F%20false%0A)

Syntax, như bạn thấy, nó sẽ được chuyển sang dạng mà browser có thể hiểu được.

Bạn có thể nghĩ rằng "Hey, code này sẽ chạy được trên IE9 vì nó được transpile rồi". Rất tiếc điều này là sai.

Như bạn thấy, tôi sử dụng method `includes`. Và bạn nhìn kĩ trên đoạn code được transpile, nó vẫn ở đó thôi. Bạn nên bỏ suy nghĩ rằng `arr.includes(item)` sẽ được transpile thành `arr.indexOf(item) > -1` hay gì gì đó đi.

Thực tế, `syntax` đã được sắp xếp lại, nhưng ta vẫn sử dụng những feature code ES2016, và nó sẽ không chạy được trên IE9.

Vì vậy, transpiling cho chúng ta một số cách, nhưng ta cần cách để giúp cho browsers đời cũ hiểu được "includes` nghĩa là gì.
> More Info: Transpile có thể hiểu được là chuyển đổi. CoffeeScript, TypeScript, Elm thường được gọi là ngôn ngữ transpilation (chuyển đổi) vì level của TypeScript và JS là tương đương nhau, viết code JS trong TypeScript thì TS vẫn hiểu được. Còn compilation thường nói về việc chuyển sang ngôn ngữ bậc thấp hơn,

<strong>Polyfill</strong>
Polyfill là đoạn mã giúp định nghĩa một object/method mới trên browser mà bản thân browser đó chưa support object/method đó. Bạn có thể có polyfill cho nhiều features khác nhau. Một trong đó có thể là `Array.prototype.includes`, tiếp theo có thể là `Map`, `Promise`.

<strong>Transpilers and polyfills can do everything, that’s wonderful!</strong>
Điều này là hoàn toàn sai.
Transpilers and polyfills không thể  Transpilers and polyfills làm được mọi thứ.
Không phải lúc nào cũng clear được cái gì cần transpile, cái gì cần polyfill. Ảnh dưới là sự so sánh của 2 cách này:

Một số rule cho điều này:
- Nếu đó là syntax mới, bạn có thể cần transpile
- Nếu nó là object/method mới, có thể là polyfill
- Nếu có gì đó mà browser làm ngoài scope của dòng code của bạn, có thể là SOL.

<strong>Where can I find polyfills?</strong>
Bạn có thể tham khảo `babel-polyfill` và `core-js`(là những gì babel-polyfill đang làm). Nếu feature bạn muốn polyfill ở trong package đó, hãy sử dụng chúng.

<strong>Polyfills and performance</strong>
Storytime: Hiện tại tôi muốn format một number thành giá tiền, chẳng hạn chuyển 123.4567 sang $1,234.57.

Đối mặt với những task như vậy, một số bạn dùng regex, một số dùng vài package npm để dùng. Một số dùng những features có sẵn của trình duyệt: [JavaScript’s fairly advanced built-in formatting features](https://hackernoon.com/polyfills-everything-you-ever-wanted-to-know-or-maybe-a-bit-less-7c8de164e423).
Với cách giải quyết này:
```js
  const price = 1234.567;

  const formattedPrice = price.toLocaleString('en', {
    style: 'currency',
    currency: 'USD'
  });
```
Job done! Tất nhiên có một vài nhược điểm. Nếu sử dụng safari 9, bạn cần sử dụng polyfill.

Đặc biệt, bạn sẽ cần polyfill `Internationalization API` dùng để define parameters cho hàm tolocalString().

`Intl` polyfill chỉ có 214 KB, khi nén chỉ còn 25KB.

Để polyfill tất cả lên ES2017, cần một con số lớn hơn. Ta không thể put all polyfill vào app được. Do đó mục tiêu là: **LOAD LESS CODE**

  1. Polyfill fewer things by targeting only newer browsers: Chỉ dùng để polyfill những browser mới
  2. Only send polyfills to browsers that need them: Chỉ gửi polyfill tới browser thực sự cần
  3. Only polyfill things used in your code: Cái nào cần thì mới polyfill

<strong>Polyfill fewer things by only targeting newer browsers</strong>

Babel có một [tool](http://babeljs.io/docs/en/babel-preset-env/) có thể giảm số lượng feature bằng việc chỉ polyfill dựa vào browser mà mình config target tới.

Ví dụ, nếu browser cũ nhất bạn support là IE11, bạn không cần dùng polyfill `Array.prototype.forEach()` vì nó được sử dụng từ IE9. Đó không phải là ý kiến hay. Bạn không cần phải polyfill everything. Thay vào đó
```js
  "babel": {
    "presets": [
      [
        "env",
        {
          "targets": {
            "browsers": [
              "last 2 versions",
              "IE >= 11"
            ]
          },
          "useBuiltIns": true
        }
      ]
    ]
  },
```
  > Well … your app will shrink from 127 KB to 121 KB. A measly six kilobytes.

Bạn bắt đầu suy nghĩ: Thay vì giảm dung lượng polyfill được bundle và mất đi khách hàng, tại sao ta không...

<strong>Only send polyfills to browsers that need them</strong>

Trước khi tiến hành, tôi muốn hỏi bạn. Bạn có thoải mái khi dùng một script từ bên thứ 3 trong page của bạn - chấp nhận 1 thời gian nhỏ bị block khi loading trên page không. Nếu có hãy dùng `polyfill.io`

Và bạn sẽ thêm một vài đoạn mã để chắc chắn khi nào cần polyfill, nó đại loại như là:

```
  - Your script: "Hey, browser, mày hiểu thế nào là `fetch()` không?"
  - IE11: [lẩm bẩm, ngao ngán]
  - Your script: "Đừng lo, tao sẽ load một polyfill và dạy cho mày vài đường cơ bản về món ấy nhé"
```

Ngày xưa thì cần dùng plain JS, bây giờ mình có thể dùng `require.ensure` syntax của webpack:
```js
  if ('fetch' in window) {
    carryOnStartingTheApp();
  } else {
    require.ensure([], () => {
      require('whatwg-fetch');

      carryOnStartingTheApp();
    });
  }
```

Mặc dù tôi tốn rất nhiều effort để giúp website của tôi hoạt động tốt trên trình duyệt cũ nhưng tôi sẽ không tốn time đi tiết kiệm một ít KB cho những trình duyệt lỗi thời nữa. Thay vào đó, tôi sẽ tập trung vào performance cho trình duyệt từ IE9 trở đi.

<strong>Only polyfill things used in your code</strong>

Còn nhớ những gì tôi vừa đề cập về cách thứ 3 để giảm thiểu feature cần polyfill không. Tôi remove 11KB - tiết kiệm 11KB khi chỉ send tới browser cũ. Đó là một chủ đề dài.

Nếu bạn cũng làm tương tự và chẳng đem lại lợi ích gì cho phần lớn người dùng. Vì sự thật, hiện nay người dùng đã và đang sử dụng các modern browsers, các browser này đều support modern JS một cách tuyệt vời. Thế nên, với tiêu chí 1: `Polyfill fewer things by targeting only newer browsers`. Bạn đã nắm chắc concept của bài viết muốn truyền tải về polyfill. Cheer!

Happy coding!
