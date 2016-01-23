# 變數的解構賦值
## 陣列的解構賦值
### Basic
ES6 允許按照一定模式，從陣列和物件中提取值，對變數進行賦值，這被稱作 Destructuring。

ES6允許寫成下面這樣。

	var [a, b, c] = [1, 2, 3];

本質上，這種寫法屬於“模式匹配”，只要等號兩邊的模式相同，左邊的變數就會被賦予對應的值。下面是一些使用嵌套陣列進行解構的例子。

	let [foo, [[bar], baz]] = [1, [[2], 3]];
	foo // 1
	bar // 2
	baz // 3

	let [ , , third] = ["foo", "bar", "baz"];
	third // "baz"

	let [x, , y] = [1, 2, 3];
	x // 1
	y // 3

	let [head, ...tail] = [1, 2, 3, 4];
	head // 1
	tail // [2, 3, 4]

	let [x, y, ...z] = ['a'];
	x // "a"
	y // undefined
	z // []

如果解構不成功，變數的值就等於`undefined`。

	var [foo] = [];
	var [bar, foo] = [1];

以上兩種情況都不成功，`foo`的值會等於`undefined`。

另一種情況是不完全的解構，亦即只匹配一部份的資料。

	let [x, y] = [1, 2, 3];
	x // 1
	y // 2

	let [a, [b], d] = [1, [2, 3], 4];
	a // 1
	b // 2
	d // 4

如果等號右邊不是可以遍歷的結構。

	// 報錯
	let [foo] = 1;
	let [foo] = false;
	let [foo] = NaN;
	let [foo] = undefined;
	let [foo] = null;
	let [foo] = {};

不僅適用於`var`，也適用於`let`、`const`和`Set`結構

	var [v1, v2, ..., vN ] = array;
	let [v1, v2, ..., vN ] = array;
	const [v1, v2, ..., vN ] = array;
	let [x, y, z] = new Set(["a", "b", "c"])
	x // "a"

事實上，只要資料結構具有 Iterator 接口，都可以採用陣列形式的解構賦值。

	function* fibs () {
		var a = 0;
		var b = 1;
		while (true) {
			yield a;
			[a, b] = [b, a + b];
		}
	}

	var [first, second, third, fourth, fifth, sixth] = fibs();
	sixth // 5

上面程式碼，`fibs`是一個 Generator 函數，原生具有 Iterator 接口。解構賦值會依次從這個接口獲取值。

### 預設值
解構賦值允許指定預設值。

	var [foo = true] = [];
	foo // true

	[x, y = 'b'] = ['a'] // x='a', y='b'
	[x, y = 'b'] = ['a', undefined] // x='a', y='b'

注意，ES6 內部使用`===`，判斷一個位置是否有值。所以如果不等於`undefined`，則預設值不會生效。

	var [x = 1] = [undefined];
	x // 1

	var [x = 1] = [null];
	x // null

預設值可以引用解構賦值的其他變數，但該變數必須已經宣告

	let [x = 1, y = x] = [];     // x=1; y=1
	let [x = 1, y = x] = [2];    // x=2; y=2
	let [x = 1, y = x] = [1, 2]; // x=1; y=2
	let [x = y, y = 1] = [];     // ReferenceError, y 還沒有宣告

## 物件的解構賦值
解構也可以用於物件

	var { foo, bar } = { foo: "aaa", bar: "bbb" };
	foo // "aaa"
	bar // "bbb"

陣列的元素是按次序排序的，變數的取值由位置決定。物件的屬性沒有次序，變數需與屬性同名，才能取到正確的值。

	var { bar, foo } = { foo: "aaa", bar: "bbb" };
	foo // "aaa"
	bar // "bbb"

	var { baz } = { foo: "aaa", bar: "bbb" };
	baz // undefined

如果變數名與屬性名不一致，必須寫成下面形式：

	var { foo: baz } = { foo: "aaa", bar: "bbb" };
	baz // "aaa"

	let obj = { first: 'hello', last: 'world' };
	let { first: f, last: l } = obj;
	f // 'hello'
	l // 'world'

物件的解構賦值的內部機制，是先找到同名屬性，然後再賦值給對應的變數。真正被賦值的是後者，而不是前者。

	var { foo: baz } = { foo: "aaa", bar: "bbb" };
	baz // "aaa"
	foo // error: foo is not defined

和陣列一樣，解構也可以用於謙套結構的物件

	var obj = {
	  p: [
	    "Hello",
	    { y: "World" }
	  ]
	};

	var { p: [x, { y }] } = obj;
	x // "Hello"
	y // "World"

這時的`p`是模式，不是變數，因此不會賦值

	var node = {
	  loc: {
	    start: {
	      line: 1,
	      column: 5
	    }
	  }
	};

	var { loc: { start: { line }} } = node;
	line // 1
	loc  // error: loc is undefined
	start // error: start is undefined

下面是嵌套賦值的例子：

	let obj = {};
	let arr = [];

	({ foo: obj.prop, bar: arr[0] } = { foo: 123, bar: true });

	obj // {prop:123}
	arr // [true]

物件的解構也可以指定預設值：

	var {x = 3} = {};
	x // 3

	var {x, y = 5} = {x: 1};
	x // 1
	y // 5

	var { message: msg = "Something went wrong" } = {};
	msg // "Something went wrong"

如果要將一個已經宣告的變數用於解構賦值，要非常小心

	// 錯誤的寫法

	var x;
	{x} = {x: 1};
	// SyntaxError: syntax error

上面的程式法會錯誤，因為會將`{x}`判斷為一個作用域。只有不將大括號寫在開頭，就可以避免。

	// 正確
	({x} = {x: 1});

## 字串的解構賦值
字串也可以使用解構賦值，這是因為會被轉換成一個類似陣列的物件。

	const [a, b, c, d, e] = 'hello';
	a // "h"
	b // "e"
	c // "l"
	d // "l"
	e // "o"

類似陣列的物件都有一個`length`屬性，因此還可以對這個屬性解構賦值。

	let {length : len} = 'hello';
	len // 5

## 數值和布林值的解構賦值
如果等號右邊是數值或是布林值，會先轉為物件。

	let {toString: s} = 123;
	s === Number.prototype.toString // true

	let {toString: s} = true;
	s === Boolean.prototype.toString // true

解構賦值的規則是，只要等號右邊的值不是物件，就先將其轉為物件。由於`undefined`和`null`無法轉為物件，所以對它們進行解構賦值，會錯誤。

	let { prop: x } = undefined; // TypeError
	let { prop: y } = null; // TypeError

## 函數參數的解構賦值
函數的參數也可以使用解構賦值

	function add([x, y]){
	  return x + y;
	}

	add([1, 2]) // 3

上面程式碼中，`add`的參數實際上不是一個陣列，而是通過解構得到的變數`x`和`y`。
函數參數的解構也可以使用預設值。

	function move({x = 0, y = 0} = {}) {
	  return [x, y];
	}

	move({x: 3, y: 8}); // [3, 8]
	move({x: 3}); // [3, 0]
	move({}); // [0, 0]
	move(); // [0, 0]

`undefined`會觸發函數參數的預設值。

	[1, undefined, 3].map((x = 'yes') => x)
	// [ 1, 'yes', 3 ]

## 圓括號問題
解構賦值雖然方便，但是解析起來並不容易。對編譯器來說，到底是模式還是表達式，無法從一開始知道。ES6 的規則是，只要有可能導致解構的歧異，就不得使用圓括號。因此建議只要有可能，就不要在模式中放置圓括號。

### 不能使用的情況
以下三種情況不能使用圓括號：

1. 變數宣告語句

		// 全部錯誤
		var [(a)] = [1];
		var { x: (c) } = {};
		var { o: ({ p: p }) } = { o: { p: 2 } };

2. 函數參數。也屬於變數宣告，因此不能有圓括號

		// 錯誤
		function f([(z)]) { return z; }

3. 不能將整個模式，或嵌套模式中的一層，放在圓括號之中

		// 全部錯誤
		({ p: a }) = { p: 42 };
		([a]) = [5];

### 可以使用的情況
只有一種情況：賦值語句的非模式部分

	[(b)] = [3]; // 正確，模式是取陣列第一個成員，與圓括號無關
	({ p: (d) } = {}); // 正確，模式是 p 而不是 d
	[(parseInt.prop)] = [3]; // 正確，與第一個一樣

## 用途
變數的解構賦值用途很多。

1. 交換變數的值


		[x, y] = [y, x];

2. 從函數返回多個值  
函數只能返回一個值，如果要返回多個，只能將它們放在陣列或物件裡。

		// 返回陣列

		function example() {
		  return [1, 2, 3];
		}
		var [a, b, c] = example();

		// 返回物件

		function example() {
		  return {
		    foo: 1,
		    bar: 2
		  };
		}
		var { foo, bar } = example();

3. 函數參數的定義
可以方便地將一組參數與變數名對應起來

		// 參數是一組有次序的值
		function f([x, y, z]) { ... }
		f([1, 2, 3])

		// 參數是一組無次序的值
		function f({x, y, z}) { ... }
		f({z: 3, y: 2, x: 1})

4. 函數參數的預設值
指定參數的預設值，避免在函數內部再寫`var foo = config.foo || "default foo"`的語句。

		jQuery.ajax = function (url, {
		  async = true,
		  beforeSend = function () {},
		  cache = true,
		  complete = function () {},
		  crossDomain = false,
		  global = true,
		  // ... more config
		}) {
		  // ... do stuff
		};

5. 提取 JSON 資料
對於從 JSON 物件取值，尤其有用。

		var jsonData = {
		  id: 42,
		  status: "OK",
		  data: [867, 5309]
		}

		let { id, status, data: number } = jsonData;

		console.log(id, status, number)
		// 42, OK, [867, 5309]

6. 遍歷 Map 結構
任何部署了 Iterator 接口的物件，都可以用`for...of`迴圈遍歷。Map 結構原生支持 Iterator 接口，配合變數的解構賦值，獲取 key 名或 key value 就很方便。

		var map = new Map();
		map.set('first', 'hello');
		map.set('second', 'world');

		for (let [key, value] of map) {
		  console.log(key + " is " + value);
		}
		// first is hello
		// second is world

		// 獲取 key
		for (let [key] of map) {
		  // ...
		}

		// 獲取 key value
		for (let [,value] of map) {
		  // ...
		}

7. 輸入模組的指定方法
載入模組時，常需要指定輸入哪些方法，解構賦值可以使語句非常清晰。

		const { SourceMapConsumer, SourceNode } = require("source-map");

-

# 字串的強化
ES6加強了對Unicode的支持，並且擴展了字串物件。

## 字串的 Unicode 表示法
JavaScript 允許採用`\uxxxx`形式表示一個字符，其中"xxxx"表示字符的碼點。

	"\u0061" // "a"

但是，這種表示法只限於`\u0000`->`\uFFFF`。超出範圍的字符，需要用兩個雙字節的形式表達。

	"\uD842\uDFB7"
	// "𠮷"

	"\u20BB7"
	// " 7"

上面代碼表示，如果直接在“\u”後面跟上超過`0xFFFF`的數值（比如`\u20BB7`），JavaScript會理解成“\u20BB+7”。由於`\u20BB`是一個不可打印字符，所以只會顯示一個空格，後面跟著一個7。

ES6對這一點做出了改進，只要將碼點放入大括號，就能正確解讀該字符。

	"\u{20BB7}"
	// "𠮷"

	"\u{41}\u{42}\u{43}"
	// "ABC"

	let hello =  123 ;
	hell\u { 6F } // 123

	'\u{1F680}'  ===  '\uD83D\uDE80'
	// true

上面代碼中，最後一個例子代表，大括號表示法與四字節的 UTF-16 編碼是等價的。

有了這種表示法之後，JavaScript共有6種方法可以表示一個字符。

	'\z'  ===  'z'  // true
	'\172'  ===  'z' // true
	'\x7A'  ===  'z' // true
	'\u007A'  ===  'z' / / true
	'\u{7A}'  ===  'z' // true

## `codePointAt()`
JavaScript 內部，字符是以 UTF-16，每個字符固定式兩個字節。對於需要4個字節儲存的字符（Unicode 碼大於 0xFFFF 的字節），JavaScript 會認為他們是兩個字符。

	var s = "𠮷";

	s.length // 2
	s.charAt(0) // ''
	s.charAt(1) // ''
	s.charCodeAt(0) // 55362
	s.charCodeAt(1) // 57271

上面代碼中"𠮷"的碼點是 0x20BB7，UTF-16 編碼為 0xD842 0xDFB7（十進制為55362 57271），需要4個字節儲存。對於這種4個字節的字符，JavaScript不能正確處理，字串長度會誤判為2，而且charAt方法無法讀取整個字符，charCodeAt方法只能分別返回前兩個字節和後兩個字節的值。

ES6 提供了`codePointAt`方法，能夠正確處理4個字節儲存的字符，返回一個字符的碼點。

	var s = '𠮷a';

	s.codePointAt(0) // 134071
	s.codePointAt(1) // 57271

	s.charCodeAt(2) // 97

`codePointAt`方法的參數，是字符在字串中的位置（從0開始）。上面代碼中，JavaScript將"𠮷a"視為三個字符，`codePointAt`方法在第一個字符上，正確地識別了"吉"，返回了它的十進制碼點 134071（即十六進制的`20BB7`）。在第二個字符（即"𠮷"的後兩個字節）和第三個字符“a”上，`codePointAt`方法的結果與`charCodeAt`方法相同。



## `String.fromCodePoint()`
## 字串的遍歷器接口
ES6 替字串添加了遍歷器接口，使得字串可以被`for...of`循環遍歷。

	for (let codePoint of 'foo') {
		console.log(codePoint)
	}
	// "f"
	// "o"
	// "o"

這個遍歷器最大的優點是可以識別大於`0xFFFF`的碼點，傳統的`for`循環無法識別這樣的碼點。

	var text = String.fromCodePoint(0x20BB7);

	for (let i = 0; i < text.length; i++) {
	  console.log(text[i]);
	}
	// " "
	// " "

	for (let i of text) {
	  console.log(i);
	}
	// "𠮷"

## `at()`
ES5對字串物件提供charAt方法，返回字串給定位置的字符。該方法不能識別碼點大於`0xFFFF`的字符。

	"abc".charAt(0) // "a"
	"吉".charAt(0) // "\uD842"

上面代碼中，`charAt`方法返回的是 UTF-16 編碼的第一個字節，實際上是無法顯示的。

ES7 提供了字串實例的`at`方法，可以識別 Unicode 編號大於`0xFFFF`的字符，返回正確的字符。Chrome 瀏覽器已經支持該方法。

	"abc".at(0) // "a"
	"吉".at(0) // "吉"

## `normalize()`
## `includes()`, `startsWith()`, `endsWith()`
## `repeat()`
## `padStart()`，`padEnd()`
## 模板字串
## 實例：模板編譯
## 標籤模板
## `String.raw()`
