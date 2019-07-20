---
title: Why This Keyword In Javascript
date: 2018-08-18 11:42:19
tags:
  - technical
---

Nhân tiện vừa có bro hỏi về vụ this và mình thấy bản thân cũng còn khá mơ hồ nên tìm hiểu thêm. Mình dịch luôn bài này. Hi vọng nó giúp ích cho một vài bạn.
<!-- more -->
Bên cạnh việc Javascript là một ngôn ngữ thú vị và mạnh mẽ thì nó còn khá là tricky và yêu cầu mức độ hiểu biết khá sâu về những nguyên lý cơ bản để có thể giảm thiểu được những lỗi thông thường khi làm việc thường xuyên với ngôn ngữ này.


Trong bài post này, chúng tôi sẽ giới thiệu tới bạn con trỏ `this`, behaviour của `this` và hard choices(không biết dịch sát như nào :D) đằng sau nó.

## What is this? (clgt)

Developer thường sai lầm khi nghĩ rằng this refer tới scope của function thực thi trong nó. Điều đó không đúng vì mỗi khi function được gọi, nó sẽ tạo ra một context mới cho đến khi thực thi xong.

Mỗi ngữ cảnh khi chạy function này thường reference đến object và value của object sẽ chuyển đổi (translate) sang value của con trỏ this.

Để hiểu đúng this ở đây có nghĩa là gì, chúng ta cần đi qua 2 concept cơ bản:

 - Context
 - Call-site

 ### Context

 Trong tiếng Anh, một khi một danh từ được sử dụng trong câu, thường thì tiếp theo khi gọi lại danh từ sẽ dùng đại từ nhân xưng thay vì dùng chính danh từ đấy(đại từ kiểu như I/you/we/they/…).
 > First Statement: Brad is a good boy. Second Statement: He is a very brilliant boy!

 Ai là **brilliant boy**? Tất nhiên là Brad rồi.

 Điều đó khá giống với JS, dòng lệnh comand đầu sẽ tạo ra context, trong khi this functions giống như he trong câu/command thứ 2 và sử dụng để reference đến chủ ngữ được sử dụng trong câu đầu.

 > this refer đến context mà function thực thi

 Điều trên có nghĩa là trong mỗi quá trình thực thi function, trọng tâm ở đây chính là việc context mới sẽ được xác định lại bất cứ khi nào function được gọi. Và từ khi context hold reference tới object, value của con trỏ this cũng được update theo.

 Không thể giả định rằng value của `this` là được fix và luôn refers tới scope của function. Giá trị của con trỏ `this` trên thực tế luôn dynamic và được xác định bởi cách function được gọi và nơi function được gọi.

 Phần khác biệt khi một chương trình Javscript chạy trong một context và giữ nguyên context đó cho đến khi chuyển qua context khác. Context của function đang chạy reference tới object đang gọi nó, do đó, con trỏ `this` trong một function sẽ refer tới object thực thi function đó.

 Context thực sự rất quan trọng để có thể hiểu được con trỏ `this`, chúng ta cần hiếu được how/why con text được thay đổi:

 ## Context change như thế nào?

 Nơi function được gọi trong javascript gọi là **call-site**. Điều này cũng quan trọng trong việc xác định giá trị của **this** vì nếu function không được gọi bởi bất cứ object nào xác định được(visible) thì context hiện tại của **call-site** sẽ chịu trách nhiệm cho việc tạo ra context mới cho function đó.

Một điều đơn giản, giá trị của **this** đại diện cho context lúc khởi tạo một function trong call-site và giá trị này sẽ được lưu trong call-site khác khi gọi function đấy mà không cung cấp context.

>Giá trị của con trỏ this phụ thuộc vào object chứa nó mà nó được gọi và không phải là function hay scope của function đó. (The value of the this keyword is dependent on the object upon which it is invoked and not the function itself or the function’s scope.)

**call-site** của function là điều quan trọng để xác định được con trỏ this đang refer đến cái gì.

## Bind this:

Có 4 binding quan trọng để xác định giá trị của con trỏ **this** trong một chương trình javascript. Chúng ta sẽ đi vào từng phần cụ thể:

1. <strong>Default binding</strong>

Chẳng hạn khi chúng ta viết một chương trình và define một function nơi ta sử dụng **this** keyword, sau đó ta gọi function ở global scope(ngoài function). Object nào bạn nghĩ sẽ reference tới con trỏ **this****?

Xem ví dụ sau:
```js
  function randomFunction () {
      this.attribute = 'This is a new attribute to be created on the invoking object';
      console.log(this);
  }

  randomFunction();
```

Cách này hay được gọi là default binding, **this** ở đây reference tới global object **window**.

Ở function trên ta tạo ngẫu nhiên một function có kèm 1 thuộc tính mới là “attribute”. Bởi vì **call-site** của function được gọi có context reference tới global object(cái này là default) nên chúng ta có thể thấy được khi console.log ra output là global object.

Đây chính là default binding **this** của Javascript. Khi function được gọi không chứa object, **this** default sẽ được bind vào context hiện tại.

> Default, this sẽ bind tới global object khi được gọi bởi một global function.

Một điều đáng chú ý là khi ta dùng chế độ “strict mode”, this sẽ hold giá trị `undefined` của global function và trong anonymous function sẽ không bind được tới object nào.

2. <strong>Binding tường minh - Explicit Binding</strong>

Bạn có nhớ chúng ta đã nói như thế nào về việc context có thể thay đổi khi gọi một function/object? Có nhiều cách rõ ràng/cụ thể để có thể trả về cùng một kết quả, dưới đây là 1 số tóm tắt:
  - Bind: Method này sẽ tạo ra một function và khi gọi nó sẽ set con trỏ this tới value được cung cấp(value được pass qua tham số truyền vào)
  - Call: Method này sẽ gọi một function và cho phép ta có thể chỉ định context của **this** sẽ được bind tới tham số truyền vào. Method này cho phép các tham số truyền vào tự chọn.
  - Apply: Method này tương tự với **Call** với sự khác biệt ở đây là cho phép chúng ta pass tham số truyền vào dưới dạng array.

  Hãy xem ví dụ sau:
```js
    function randomFunction() {
        console.log(this);
    }

    let newObj = {
        description : "This is a new Object"
    }

    console.log(randomFunction.bind(newObj)());
    console.log(randomFunction.call(newObj));
    console.log(randomFunction.apply(newObj));
```
Ở đoạn code trên, ta đã bind this một cách cụ thể trong randomFunction tới biến newObj và ta có thể thấy các method call, bind, apply đều có kết quả trả về như nhau.

3. <strong>Binding ngầm - Implicit Binding:</strong>

Khi một object define một method(gọi là **this**) như một thuộc tính và gọi method đó bất cứ đâu trong chương trình/file, con trỏ **this** trong method sẽ ngầm định được binding tới object đó. Hãy xem ví dụ sau:

```js
    let newObj = {
        description : "This is a new Object",
        randomFunction(){
            console.log(this);
        }
    }
    newObj.randomFunction();
```
4. <strong>New binding</strong>

Nếu ta có một object constructor và tạo ra một object mới với nó, object mới sẽ có value this reference tới constructor từ nơi nó được tạo. Việc binding này sẽ diễn ra khi một object mới được tạo sử dụng constructor. Hãy xem ví dụ sau:

```js
    function newObj() {
        this.description = "This is an object constructor"

        this.randomFunction = () => {
            console.log(this);
        }  
    }

    let anotherFunction = new newObj()
    anotherFunction.randomFunction()
```

Theo như trên, ta define một function constructor và đưa vào thuộc tính **description** cũng như **randomFunction** với method tương ứng. Ta tạo một instance với biến **anotherFunction** và gọi method **randomFunction**, output sẽ là ?

Object log ra sẽ có thuộc tính description đã được define trong object constructor để chứng minh this reference tới constructor function.

## This & That

> Bất cứ nơi đâu mà con trỏ this được define bởi function, nó thường không được assign một value cụ thể cho đến khi object gọi đến function đó

Bạn có thể nghĩ đoạn thông tin trên là những gì bạn cần khi làm việc với **this** nhưng có một vài kịch bản nơi mà **this** có những behavior khác.

Hãy xem đoạn code dưới khi ta define một closure mà reference tới **this** trong Javascript:

```js
    let newObj = {
        description : "This is a new Object",

        randomFunction(){
            let a = 1

            return function() {
            console.log(this);
            }
        }
    }

    newObj.randomFunction()();
```

Bạn có thể đoán một cách hợp lý rằng closure sẽ return ra giá trị của this là object hiện đang gọi **randomFunction** nhưng hãy out put lại là object window.

Hãy xem, bằng cách nào mà this bây giờ lại refer tới global object? Không phải Outer function đã được gọi bởi newObject object sao?

Không phải việc thay đổi context và updated this reference hay sao?

Tất cả những câu hỏi trên đều đúng, tuy nhiên có một vài điểm đáng lưu ý:
> Closure không thể access giá trị this của outer function bằng cách call chính con trỏ this nằm trong nó

Điều này diễn ra bởi giá trị this của function chỉ có thể access bởi function bên trong nơi mà nó đang chạy và không phải inner function. Vì thế this của anonymous closure sẽ được bind tới global object nơi strict mode không được sử dụng.

Làm thế nào để ta có thể xác định được điều trên ?

Đây là trick. Hãy xem đoạn code nhỏ dưới:
```js
    var newObj = {
        description : "This is a new Object",
        randomFunction(){
            var that = this;

            return function() {
            console.log(that);
            }
        }
    }

    newObj.randomFunction()();
```

Ở đây, trong randomFunction ta khai báo một biến mới gọi là that và assign giá trị của this cho nó. Bằng cách này, ta có thể reference giá trị của this của outer function trong closure từ khi scope của outer function có thể access bởi inner function.

Great! Chúng ta đã reference this value của outer function trong closure bằng cách tạo thêm một biến that.
Một điều nên nhớ là ta có thể gọi biến đó là gì cũng được tuy nhiên nên chọn là that vì nhiều JS developer thường thấy thoải mái khi gọi nó là that.

Bây giờ bạn đã hiểu về this và that.

Có một cách mới nếu không muốn phải khai báo thêm biến mới, đó là dùng arrow function. Ta có:

```js
    var newObj = {
        description : "This is a new Object",
        randomFunction(){
            return () =>  console.log(this)
        }
    }

    newObj.randomFunction()();
```

Hai cách trên đều có chung kết quả:
```js
    // Sự khác nhau giữa Function thông thường và Arrow Function
    // Function thông thường
    let regularObj = function() {
        this.name = 'An Vu';
        return {
            name: 'John',
            getName: function() {
                // this sẽ bị ghi đè vì this lúc này thuộc về Obj được trả về
                return this.name;
            }
        }
    }
    // Arrow Function
    let arrowObj = function() {
        this.name = 'An Vu';
        return {
            name: 'John',
            getName: () => {
                // this này không bị ghi đè bởi Obj mà vẫn giữ được this của hàm cha
                return this.name;
            }
        };
    }
    console.log('arrowObj: ' + arrowObj().getName());
```

```
    regularObj: John
    arrowObj: An Vu
    Ở ví dụ này các bạn cũng đã thấy từ khóa this của chúng ta có giá trị khác biệt, this ở function thường sẽ tham chiếu đến chính Object chứa nó, còn Arrow Function sẽ tham chiếu đến cha gần nhất của nó nên vì thế nó hiểu từ khóa this của function cha bao hàm nó.
    Và khi các bạn có dùng thêm use strict để ràng buộc chặt chẽ hơn về phạm vi hoạt động của biến thì this trong Arrow Function có giá trị sẽ là undefined thay vì trả về cho các bạn đối tượng window. Nên các bạn hãy chú ý điều này.
```

## Tổng kết

Chúng ta đã làm sáng tỏ this và cách sử dụng nó. Ta cũng đã nói về bài toán closure nơi mà logic của this khá là khó nắm bắt và giải pháp, tuy nhiên thực tế nhiều tình huống this vẫn tỏ ra khá là tricky. Cách tốt nhất để tìm ra là xác định rõ việc binding là do function nào thực thi và lần ra context của nó một cách cẩn thận. Chúc bạn may mắn :))

Be good. Sleep well. And enjoy coding.
