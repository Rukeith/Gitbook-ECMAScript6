# Symbol
## 概述
ES5 的物件屬性名都是字串，這容易造成屬性名的衝突。比如，你使用了一個他人提供的物件，但又想為這個物件添加新的方法（mixin模式），新方法的名字就有可能與現有方法產生衝突。如果有一種機制，保證每個屬性的名字都是獨一無二的就好了，這樣就從根本上防止屬性名的衝突。這就是 ES6 引入 Symbol 的原因。

ES6 引入了一種新的原始資料類型 Symbol，表示獨一無二的值。它是 JavaScript 語言的第七種資料類型，前六種是：Undefined、Null、布林值（Boolean）、字串（String）、數值（Number）、物件（Object）。

Symbol 值通過 Symbol 函數生成。這就是說，物件的屬性名現在可以有兩種類型，一種是原來就有的字串，另一種就是新增的 Symbol 類型。凡是屬性名屬於 Symbol 類型，就都是獨一無二的，可以保證不會與其他屬性名產生衝突。

	let s = Symbol();
	
	typeof s
	// "symbol"

上面代碼中，變數`s`就是一個獨一無二的值。`typeof`運算符的結果，代表變數`s`是 Symbol 資料類型，而不是字串之類的其他類型。

注意，Symbol 函數前不能使用`new`命令，否則會報錯。這是因為生成的 Symbol 是一個原始類型的值，不是物件。也就是說，由於 Symbol 值不是物件，所以不能添加屬性。基本上，它是一種類似於字串的資料類型。

Symbol 函數可以接受一個字串作為參數，表示對 Symbol 實例的描述，主要是為了在控制台顯示，或者轉為字串時，比較容易區分。

	var s1 = Symbol('foo');
	var s2 = Symbol('bar');
	
	s1 // Symbol(foo)
	s2 // Symbol(bar)
	
	s1.toString() // "Symbol(foo)"
	s2.toString() // "Symbol(bar)"

上面代碼中，`s1`和`s2`是兩個 Symbol 值。如果不加參數，它們在控制台的輸出都是`Symbol()`，不利於區分。有了參數以後，就等於為它們加上了描述，輸出的時候就能夠分清，到底是哪一個值。

注意，Symbol 函數的參數只是表示對當前 Symbol 值的描述，因此相同參數的 Symbol 函數的返回值是不相等的。

	// 没有參數的情况
	var s1 = Symbol();
	var s2 = Symbol();
	
	s1 === s2 // false
	
	// 有參數的情况
	var s1 = Symbol("foo");
	var s2 = Symbol("foo");
	
	s1 === s2 // false

上面代碼中，`s1`和`s2`都是 Symbol 函數的返回值，而且參數相同，但是它們是不相等的。

Symbol 值不能與其他類型的值進行運算，會報錯。

	var sym = Symbol('My symbol');
	
	"your symbol is " + sym
	// TypeError: can't convert symbol to string
	`your symbol is ${sym}`
	// TypeError: can't convert symbol to string

但是，Symbol 值可以顯式轉為字串。

	var sym = Symbol('My symbol');
	
	String(sym) // 'Symbol(My symbol)'
	sym.toString() // 'Symbol(My symbol)'

另外，Symbol 值也可以轉為布林值，但是不能轉為數值。

	var sym = Symbol();
	Boolean(sym) // true
	!sym  // false
	
	if (sym) {
	  // ...
	}
	
	Number(sym) // TypeError
	sym + 2 // TypeError

## 作為屬性名的 Symbol
由於每一個 Symbol 值都是不相等的，這意味著 Symbol 值可以作為標識符，用於物件的屬性名，就能保證不會出現同名的屬性。這對於一個物件由多個模塊構成的情況非常有用，能防止某一個鍵被不小心改寫或覆蓋。

	var mySymbol = Symbol();
	
	// 第一種寫法
	var a = {};
	a[mySymbol] = 'Hello!';
	
	// 第二種寫法
	var a = {
	  [mySymbol]: 'Hello!'
	};
	
	// 第三種寫法
	var a = {};
	Object.defineProperty(a, mySymbol, { value: 'Hello!' });
	
	// 以上寫法都得到同樣結果
	a[mySymbol] // "Hello!"

上面代碼通過方括號結構和`Object.defineProperty`，將物件的屬性名指定為一個 Symbol 值。

注意，Symbol 值作為物件屬性名時，不能用點運算符。

	var mySymbol = Symbol();
	var a = {};
	
	a.mySymbol = 'Hello!';
	a[mySymbol] // undefined
	a['mySymbol'] // "Hello!"

上面代碼中，因為點運算符後面總是字串，所以不會讀取`mySymbol`作為標識名所指代的那個值，導致`a`的屬性名實際上是一個字串，而不是一個 Symbol 值。

同理，在物件的內部，使用 Symbol 值定義屬性時，Symbol 值必須放​​在方括號之中。

	let s = Symbol();
	
	let obj = {
	  [s]: function (arg) { ... }
	};
	
	obj[s](123);

上面代碼中，如果`s`不放在方括號中，該屬性的鍵名就是字串`s`，而不是`s`所代表的那個 Symbol 值。

採用增強的物件寫法，上面代碼的`obj`物件可以寫得更簡潔一些。

	let obj = {
	  [s](arg) { ... }
	};

Symbol 類型還可以用於定義一組常量，保證這組常量的值都是不相等的。

	log.levels = {
	  DEBUG: Symbol('debug'),
	  INFO: Symbol('info'),
	  WARN: Symbol('warn'),
	};
	log(log.levels.DEBUG, 'debug message');
	log(log.levels.INFO, 'info message');

下面是另外一個例子。

	const COLOR_RED    = Symbol();
	const COLOR_GREEN  = Symbol();
	
	function getComplement(color) {
	  switch (color) {
	    case COLOR_RED:
	      return COLOR_GREEN;
	    case COLOR_GREEN:
	      return COLOR_RED;
	    default:
	      throw new Error('Undefined color');
	    }
	}

常量使用 Symbol 值最大的好處，就是其他任何值都不可能有相同的值了，因此可以保證上面的`switch`語句會按設計的方式工作。

還有一點需要注意，Symbol 值作為屬性名時，該屬性還是公開屬性，不是私有屬性。

## 實例：消除魔術字串
魔術字串指的是，在代碼之中多次出現、與代碼形成強耦合的某一個具體的字串或者數值。風格良好的代碼，應該盡量消除魔術字串，該由含義清晰的變數代替。

	function getArea(shape, options) {
	  var area = 0;
	
	  switch (shape) {
	    case 'Triangle': // 魔術字串
	      area = .5 * options.width * options.height;
	      break;
	    /* ... more code ... */
	  }
	
	  return area;
	}
	
	getArea('Triangle', { width: 100, height: 100 }); // 魔術字串

上面代碼中，字串 "Triangle" 就是一個魔術字串。它多次出現，與代碼形成"強耦合"，不利於將來的修改和維護。

常用的消除魔術字串的方法，就是把它寫成一個變數。

	var shapeType = {
	  triangle: 'Triangle'
	};
	
	function getArea(shape, options) {
	  var area = 0;
	  switch (shape) {
	    case shapeType.triangle:
	      area = .5 * options.width * options.height;
	      break;
	  }
	  return area;
	}
	
	getArea(shapeType.triangle, { width: 100, height: 100 });

上面代碼中，我們把 "Triangle" 寫成`shapeType`物件的`triangle`屬性，這樣就消除了強耦合。

如果仔細分析，可以發現`shapeType.triangle`等於哪個值並不重要，只要確保不會跟其他`shapeType`屬性的值衝突即可。因此，這裡就很適合改用 Symbol 值。

	const shapeType = {
	  triangle: Symbol()
	};

上面代碼中，除了將`shapeType.triangle`的值設為一個 Symbol，其他地方都不用修改。

## 屬性名的遍歷
Symbol 作為屬性名，該屬性不會出現在`for...in`、`for...of`循環中，也不會被`Object.keys()`、`Object.getOwnPropertyNames()`返回。但是，它也不是私有屬性，有一個`Object.getOwnPropertySymbols`方法，可以獲取指定物件的所有 Symbol 屬性名。

`Object.getOwnPropertySymbols`方法返回一個陣列，成員是當前物件的所有用作屬性名的 Symbol 值。

	var obj = {};
	var a = Symbol('a');
	var b = Symbol.for('b');
	
	obj[a] = 'Hello';
	obj[b] = 'World';
	
	var objectSymbols = Object.getOwnPropertySymbols(obj);
	
	objectSymbols
	// [Symbol(a), Symbol(b)]

下面是另一個例子，`Object.getOwnPropertySymbols`方法與`for...in`循環、`Object.getOwnPropertyNames`方法進行對比的例子。

	var obj = {};
	
	var foo = Symbol("foo");
	
	Object.defineProperty(obj, foo, {
	  value: "foobar",
	});
	
	for (var i in obj) {
	  console.log(i); // 無輸出
	}
	
	Object.getOwnPropertyNames(obj)
	// []
	
	Object.getOwnPropertySymbols(obj)
	// [Symbol(foo)]

上面代碼中，使用`Object.getOwnPropertyNames`方法得不到 Symbol 屬性名，需要使用`Object.getOwnPropertySymbols`方法。

另一個新的 API，`Reflect.ownKeys`方法可以返回所有類型的鍵名，包括常規鍵名和 Symbol 鍵名。

	let obj = {
	  [Symbol('my_key')]: 1,
	  enum: 2,
	  nonEnum: 3
	};
	
	Reflect.ownKeys(obj)
	// [Symbol(my_key), 'enum', 'nonEnum']

由於以 Symbol 值作為名稱的屬性，不會被常規方法遍歷得到。我們可以利用這個特性，為物件定義一些非私有的、但又希望只用於內部的方法。

	var size = Symbol('size');
	
	class Collection {
	  constructor() {
	    this[size] = 0;
	  }
	
	  add(item) {
	    this[this[size]] = item;
	    this[size]++;
	  }
	
	  static sizeOf(instance) {
	    return instance[size];
	  }
	}
	
	var x = new Collection();
	Collection.sizeOf(x) // 0
	
	x.add('foo');
	Collection.sizeOf(x) // 1
	
	Object.keys(x) // ['0']
	Object.getOwnPropertyNames(x) // ['0']
	Object.getOwnPropertySymbols(x) // [Symbol(size)]

上面代碼中，物件`x`的 size 屬性是一個 Symbol 值，所以`Object.keys(x)`、`Object.getOwnPropertyNames(x)`都無法獲取它。這就造成了一種非私有的內部方法的效果。

## `Symbol.for()`，`Symbol.keyFor()`
有時，我們希望重新使用同一個 Symbol 值，`Symbol.for`方法可以做到這一點。它接受一個字串作為參數，然後搜索有沒有以該參數作為名稱的 Symbol 值。如果有，就返回這個 Symbol 值，否則就新建並返回一個以該字串為名稱的 Symbol 值。

	var s1 = Symbol.for('foo');
	var s2 = Symbol.for('foo');
	
	s1 === s2 // true

上面代碼中，s1 和 s2 都是 Symbol 值，但是它們都是同樣參數的`Symbol.for`方法生成的，所以實際上是同一個值。

`Symbol.for()`與`Symbol()`這兩種寫法，都會生成新的 Symbol。它們的區別是，前者會被登記在**全局環境**中供搜索，後者不會。`Symbol.for()`不會每次調用就返回一個新的 Symbol 類型的值，而是會先檢查給定的 key 是否已經存在，如果不存在才會新建一個值。比如，如果你調用`Symbol.for("cat")`30次，每次都會返回同一個 Symbol 值，但是調用`Symbol("cat")`30次，會返回 30 個不同的 Symbol 值。

	Symbol.for("bar") === Symbol.for("bar")
	// true
	
	Symbol("bar") === Symbol("bar")
	// false

上面代碼中，由於`Symbol()`寫法沒有登記機制，所以每次調用都會返回一個不同的值。

`Symbol.keyFor`方法返回一個已登記的 Symbol 類型值的 key。

	var s1 = Symbol.for("foo");
	Symbol.keyFor(s1) // "foo"
	
	var s2 = Symbol("foo");
	Symbol.keyFor(s2) // undefined

上面代碼中，變數`s2`屬於未登記的 Symbol 值，所以返回`undefined`。

需要注意的是，`Symbol.for`為 Symbol 值登記的名字，是全局環境的，可以在不同的`iframe`或`service worker`中取到同一個值。

	iframe = document.createElement('iframe');
	iframe.src = String(window.location);
	document.body.appendChild(iframe);
	
	iframe.contentWindow.Symbol.for('foo') === Symbol.for('foo')
	// true

上面代碼中，iframe 窗口生成的 Symbol 值，可以在主頁面得到。

## 內置的 Symbol 值
除了定義自己使用的 Symbol 值以外，ES6 還提供了 11 個內置的 Symbol 值，指向語言內部使用的方法。

### `Symbol.hasInstance`
物件的`Symbol.hasInstance`屬性，指向一個內部方法。當其他物件使用`instanceof`運算符，判斷是否為該物件的實例時，會調用這個方法。比如，`foo instanceof Foo`在語言內部，實際調用的是`Foo[Symbol.hasInstance](foo)`。

	class MyClass {
	  [Symbol.hasInstance](foo) {
	    return foo instanceof Array;
	  }
	}
	
	[1, 2, 3] instanceof MyClass() // true

### `Symbol.isConcatSpreadable`
物件的`Symbol.isConcatSpreadable`屬性等於一個布林值，表示該物件使用`Array.prototype.concat()`時，是否可以展開。

	let arr1 = ['c', 'd'];
	['a', 'b'].concat(arr1, 'e') // ['a', 'b', 'c', 'd', 'e']
	
	let arr2 = ['c', 'd'];
	arr2[Symbol.isConcatSpreadable] = false;
	['a', 'b'].concat(arr2, 'e') // ['a', 'b', ['c','d'], 'e']

上面代碼說明，陣列的`Symbol.isConcatSpreadable`屬性預設為`true`，表示可以展開。

類似陣列的物件也可以展開，但它的`Symbol.isConcatSpreadable`屬性預設為`false`，必須手動打開。

	let obj = {length: 2, 0: 'c', 1: 'd'};
	['a', 'b'].concat(obj, 'e') // ['a', 'b', obj, 'e']
	
	obj[Symbol.isConcatSpreadable] = true;
	['a', 'b'].concat(obj, 'e') // ['a', 'b', 'c', 'd', 'e']

對於一個類來說，`Symbol.isConcatSpreadable`屬性必須寫成一個返回布林值的方法。

	class A1 extends Array {
	  [Symbol.isConcatSpreadable]() {
	    return true;
	  }
	}
	class A2 extends Array {
	  [Symbol.isConcatSpreadable]() {
	    return false;
	  }
	}
	let a1 = new A1();
	a1[0] = 3;
	a1[1] = 4;
	let a2 = new A2();
	a2[0] = 5;
	a2[1] = 6;
	[1, 2].concat(a1).concat(a2)
	// [1, 2, 3, 4, [5, 6]]

上面代碼中，類`A1`是可擴展的，類`A2`是不可擴展的，所以使用`concat`時有不一樣的結果。

### `Symbol.species`
物件的`Symbol.species`屬性，指向一個方法。該物件作為構造函數創造實例時，會調用這個方法。即如果`this.constructor[Symbol.species]`存在，就會使用這個屬性作為構造函數，來創造新的實例物件。

`Symbol.species`屬性預設的讀取器如下。

	static get [Symbol.species]() {
	  return this;
	}

### `Symbol.match`
物件的`Symbol.match`屬性，指向一個函數。當執行`str.match(myObject)`時，如果該屬性存在，會調用它，返回該方法的返回值。

	String.prototype.match(regexp)
	// 等同於
	regexp[Symbol.match](this)
	
	class MyMatcher {
	  [Symbol.match](string) {
	    return 'hello world'.indexOf(string);
	  }
	}
	
	'e'.match(new MyMatcher()) // 1

### `Symbol.replace`
物件的`Symbol.replace`屬性，指向一個方法，當該物件被`String.prototype.replace`方法調用時，會返回該方法的返回值。

	String.prototype.replace(searchValue, replaceValue)
	// 等同於
	searchValue[Symbol.replace](this, replaceValue)

### `Symbol.search`
物件的`Symbol.search`屬性，指向一個方法，當該物件被`String.prototype.search`方法調用時，會返回該方法的返回值。

	String.prototype.search(regexp)
	// 等同於
	regexp[Symbol.search](this)
	
	class MySearch {
	  constructor(value) {
	    this.value = value;
	  }
	  [Symbol.search](string) {
	    return string.indexOf(this.value);
	  }
	}
	'foobar'.search(new MySearch('foo')) // 0

### `Symbol.split`
物件的`Symbol.split`屬性，指向一個方法，當該物件被`String.prototype.split`方法調用時，會返回該方法的返回值。

	String.prototype.split(separator, limit)
	// 等同於
	separator[Symbol.split](this, limit)

### `Symbol.iterator`
物件的`Symbol.iterator`屬性，指向該物件的預設遍歷器方法，即該物件進行`for...of`循環時，會調用這個方法，返回該物件的預設遍歷器，詳細介紹參見《`Iterator`和`for.. .of`循環》一章。

	class Collection {
	  *[Symbol.iterator]() {
	    let i = 0;
	    while(this[i] !== undefined) {
	      yield this[i];
	      ++i;
	    }
	  }
	}
	
	let myCollection = new Collection();
	myCollection[0] = 1;
	myCollection[1] = 2;
	
	for(let value of myCollection) {
	  console.log(value);
	}
	// 1
	// 2

### `Symbol.toPrimitive`
物件的`Symbol.toPrimitive`屬性，指向一個方法。該物件被轉為原始類型的值時，會調用這個方法，返回該物件對應的原始類型值。

`Symbol.toPrimitive`被調用時，會接受一個字串參數，表示當前運算的模式，一共有三種模式。

* Number：該場合需要轉成數值
* String：該場合需要轉成字串
* Default：該場合可以轉成數值，也可以轉成字串

### `Symbol.toStringTag`
物件的`Symbol.toStringTag`屬性，指向一個方法。在該物件上面調用`Object.prototype.toString`方法時，如果這個屬性存在，它的返回值會出現在`toString`方法返回的字串之中，表示物件的類型。也就是說，這個屬性可以用來定制`[object Object]`或`[object Array]`中`object`後面的那個字串。

	({[Symbol.toStringTag]: 'Foo'}.toString())
	// "[object Foo]"
	
	class Collection {
	  get [Symbol.toStringTag]() {
	    return 'xxx';
	  }
	}
	var x = new Collection();
	Object.prototype.toString.call(x) // "[object xxx]"

ES6 新增內置物件的`Symbol.toStringTag`屬性值如下。

* `JSON[Symbol.toStringTag]`：'JSON'
* `Math[Symbol.toStringTag]`：'Math'
* `Module物件M[Symbol.toStringTag]`：'Module'
* `ArrayBuffer.prototype[Symbol.toStringTag]`：'ArrayBuffer'
* `DataView.prototype[Symbol.toStringTag]`：'DataView'
* `Map.prototype[Symbol.toStringTag]`：'Map'
* `Promise.prototype[Symbol.toStringTag]`：'Promise'
* `Set.prototype[Symbol.toStringTag]`：'Set'
* `%TypedArray%.prototype[Symbol.toStringTag]`：'Uint8Array'等
* `WeakMap.prototype[Symbol.toStringTag]`：'WeakMap'
* `WeakSet.prototype[Symbol.toStringTag]`：'WeakSet'
* `%MapIteratorPrototype%[Symbol.toStringTag]`：'Map Iterator'
* `%SetIteratorPrototype%[Symbol.toStringTag]`：'Set Iterator'
* `%StringIteratorPrototype%[Symbol.toStringTag]`：'String Iterator'
* `Symbol.prototype[Symbol.toStringTag]`：'Symbol'
* `Generator.prototype[Symbol.toStringTag]`：'Generator'
* `GeneratorFunction.prototype[Symbol.toStringTag]`：'GeneratorFunction'

### `Symbol.unscopables`
物件的`Symbol.unscopables`屬性，指向一個物件。該物件指定了使用`with`關鍵字時，哪些屬性會被`with`環境排除。

	Array.prototype[Symbol.unscopables]
	// {
	//   copyWithin: true,
	//   entries: true,
	//   fill: true,
	//   find: true,
	//   findIndex: true,
	//   keys: true
	// }
	
	Object.keys(Array.prototype[Symbol.unscopables])
	// ['copyWithin', 'entries', 'fill', 'find', 'findIndex', 'keys']

上面代碼說明，陣列有 6 個屬性，會被`with`命令排除。

	// 没有 unscopables 時
	class MyClass {
	  foo() { return 1; }
	}
	
	var foo = function () { return 2; };
	
	with (MyClass.prototype) {
	  foo(); // 1
	}
	
	// 有 unscopables 時
	class MyClass {
	  foo() { return 1; }
	  get [Symbol.unscopables]() {
	    return { foo: true };
	  }
	}
	
	var foo = function () { return 2; };
	
	with (MyClass.prototype) {
	  foo(); // 2
	}