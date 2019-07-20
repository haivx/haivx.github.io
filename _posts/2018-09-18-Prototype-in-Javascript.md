---
title: Prototype in Javascript
date: 2018-09-18 15:56:15
tags:
  - technical
  - Javascript
---
<strong>Prototype</strong> là một concept cơ bản mà mỗi JS dev nên biết, và mục đích của bài post này giúp giải thích về khái niệm này. Bài viết được trans từ link [Blog javascriptissexy](http://javascriptissexy.com/javascript-prototype-in-plain-detailed-language/).

Để hiểu được prototype trong JS bạn cần hiểu được khái niệm Object trong JS. Bên cạnh đó, cần hiểu rằng một `property` đơn giản là một biến được defined trong 1 function...

<!-- more -->
Có hai concept liên quan đến `prototype` trong JS:

 1. **Đầu tiên**, mỗi JS function đều có một `prototype property` (default thuộc tính này có giá trị rỗng), và bạn sẽ attach thêm các thuộc tính và methods vào `prototype property` khi bạn sử dụng phương thức thừa kế.  `prototype property` này là không đếm được, tức là nó không thể access được bằng vòng lặp for/in.

  Nhưng Firefox và nhiều version của Safari cũng như Chrome có một thuộc tính giả gọi là `__proto__ ` cho phép bạn có thể access vào đây được.

  Bạn hầu như không cần sử dụng `_proto_` bao giờ, tuy nhiên bạn nên biết rằng nó tồn tại và đó là một cách đơn giản để access một `prototype property` của một object trên một số browser.

  `prototype property` đơn giản được sử dụng trong việc thừa kế - inheritance. Nếu bạn add method và thuộc tính vào một `prototype property` của function sẽ tạo ra các methods và thuộc tính trong instance của function đó.

 ```js
  function PrintStuff (myDocuments) {
    this.documents = myDocuments;
  }

  // We add the print () method to PrintStuff prototype property so that other instances (objects) can inherit it:
  PrintStuff.prototype.print = function () {
    console.log(this.documents);
  }

  // Create a new object with the PrintStuff () constructor, thus allowing this new object to inherit PrintStuff's properties and methods.
  var newObj = new PrintStuff ("I am a new Object and I can print.");

  // newObj inherited all the properties and methods, including the print method, from the PrintStuff function. Now newObj can call print directly, even though we never created a print () method on it.
  newObj.print (); //I am a new Object and I can print.
 ```
2. **Concept thứ 2** đó là `prototype attribute`. Hãy nghĩ đơn giản `prototype attribute` như là đặc trưng của object, những đặc trưng này miêu tả cho chúng ta hiểu được về object `parent` của nó - object mà nó thừa kế. `prototype attribute` thường được refer tới `prototype object`, và nó tự động set khi bạn tạo object mới.

Điều này có thể giải thích như sau: Mỗi object thừa kế các property từ object khác, và những object khác đó là `parent`. Trên example trên, prototype của `newObj` là PrintStuff.prototype.

 Hãy nhớ rằng: Tất cả các object có các attributes giống như `property của object` đó có các attributes. Và các `object attributes` đó là `prototype`, `class` và `extensible` attributes.

Cũng nên chú ý rằng, `_proto_` property chứa một `prototype object` của một object(là object mẹ mà chúng thừa kế method và properties)

 > Mình không dịch `attribute` và `property` vì hai khái niệm này khi dịch sang tiếng Việt thường gây confuse.

 ## Constructor

 Trước khi tiếp tục, hãy xem xét một chút về **constructor**. Một **constructor** là một function được sử dụng để khởi tạo một object mới, và bạn sử dụng keyword `new` để gọi tới constructor.
 ```js
    function Account () {
    }
    // This is the use of the Account constructor to create the userAccount object.
    var userAccount = new Account ();
 ```
Hơn nữa, tất cả các object thừa kế từ object khác cũng thừa kế  chính **constructor property**. Và constructor này đơn giản là một property được trỏ tới constructor của object cha.

```js
    //The constructor in this example is Object ()
    var myObj = new Object ();
    // And if you later want to find the myObj constructor:
    console.log(myObj.constructor); // Object()

    // Another example: Account () is the constructor
    var userAccount = new Account ();
    // Find the userAccount object's constructor
    console.log(userAccount.constructor); // Account()
```

## Prototype Attribute của object sẽ khởi tạo bởi Object mới hoặc Object Literal

Tất cả object được tạo với object literals và với Object constructor thừa kế từ `Object.prototype`. Do đó, Object.prototype là một `prototype attribute` (hay prototype object) của tất cả các object tạo với new Object() hoặc {}. <strong>Object.prototype</strong> không thừa kế bất cứ method hay property từ object khác.

```js
  // The userAccount object inherits from Object and as such its prototype attribute is Object.prototype.
  var userAccount = new Object ();

  // This demonstrates the use of an object literal to create the userAccount object; the userAccount object inherits from Object; therefore, its prototype attribute is Object.prototype just as the userAccount object does above.
  var userAccount = {name: “Mike”}
```
## Prototype attribute của object tạo bởi một Constructor function

Object tạo với `new` keyword và bất cứ constructor khác ngoài Object() constructor, sẽ nhận được prototype từ constructor function.

```js
    function Account () {

    }
    var userAccount = new Account () // userAccount initialized with the Account () constructor and as such its prototype attribute (or prototype object) is Account.prototype.
```
Tương tự vậy, bất cứ array nào ví dụ var myArray = new Array(), get prototype của nó từ Array.prototype và nó thừa kế các properties của Array.prototype.

## Tại sao Prototype lại quan trọng và khi nào sử dụng nó?

### 1. **Prototype Property: Prototype-based Inheritance**

Prototype quan trọng trong Javascript vì JS không có cách thừa kế cổ điển thông qua Class(như các ngôn ngữ hướng đối tượng khác), và do đó tất cả các thừa kế trong JS sẽ được sử dụng thông qua các **prototype property**.

Cơ chế thừa kế của JS là dựa vào prototype. Ví dụ, bạn tạo một Fruit function(Đây là một object, với JS tất cả function là object) và add các property, metjhod vào Fruit prototype property, và tất cả instance của Fruit function sẽ thừa kế từ tất cả property, method của Fruit.
```js
    function Plant () {
        this.country = "Mexico";
        this.isOrganic = true;
    }

    // Add the showNameAndColor method to the Plant prototype property
    Plant.prototype.showNameAndColor =  function () {
        console.log("I am a " + this.name + " and my color is " + this.color);
    }

    // Add the amIOrganic method to the Plant prototype property
    Plant.prototype.amIOrganic = function () {
        if (this.isOrganic)
        console.log("I am organic, Baby!");
    }

    function Fruit (fruitName, fruitColor) {
        this.name = fruitName;
        this.color = fruitColor;
    }

    // Set the Fruit's prototype to Plant's constructor, thus inheriting all of Plant.prototype methods and properties.
    Fruit.prototype = new Plant ();

    // Creates a new object, aBanana, with the Fruit constructor
    var aBanana = new Fruit ("Banana", "Yellow");

    // Here, aBanana uses the name property from the aBanana object prototype, which is Fruit.prototype:
    console.log(aBanana.name); // Banana

    // Uses the showNameAndColor method from the Fruit object prototype, which is Plant.prototype. The aBanana object inherits all the properties and methods from both the Plant and Fruit functions.
    console.log(aBanana.showNameAndColor()); // I am a Banana and my color is yellow.
```
Nhớ rằng method **showNameAndColor** được thừa kế từ **aBanana** object mặc dù nó được define thông qua việc chain prototype **Plant.prototype object**.

Thật ra, bất cứ object nào sử dụng Fruit() constructor sẽ thừa kế tất cả **Fruit.prototype** properties và methods và tất cả properties và methods từ prototype của Fruit, là Plant.prototype. `Đây là nguyên tắc cơ bản mà tính thừa kế được implement trong Javascript` và `vai trò không thể tách rời từ việc chain các prototype` trong process.

### 2. **Object.prototype properties thừa kế bởi tất cả Objects**

Tất cả các object trong JS đều thừa kế các properties và method từ **Object.prototype**. Những properties và method được thừa kế này bao gồm `constructor, hasOwnProperty(), isPrototypeOf(), propertyIsEnumerable(), toLocaleString(), toString(), and valueOf() `.

Tất cả built-in constructor(Array (), Number (), String (), etc...) được tạo từ Object constructor và prototype của chúng là Object.prototype.

### 3.**Vậy với ES6, có sự khác biệt gì ở đây không khi ta đã có phương thức class**
Đằng sau `class` keyword, cơ chế nó vẫn là cách tiếp cận như cũ. Bạn có thể tìm hiểu thêm tại [bài này.](https://medium.com/@parsyval/javascript-prototype-vs-class-a7015d5473b)


### Be good. Sleep well. And enjoy coding.
