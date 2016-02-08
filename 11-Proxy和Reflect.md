# Proxy 和 Reflect
## Proxy 概述
Proxy 用於修改某些操作的預設行為，等同於在語言層面做出修改，所以屬於一種"元編程"（meta programming），即對編程語言進行編程。

Proxy 可以理解成，在目標物件之前架設一層"攔截"，外界對該物件的訪問，都必須先通過這層攔截，因此提供了一種機制，可以對外界的訪問進行過濾和改寫。Proxy 這個詞的原意是代理，用在這裡表示由它來"代理"某些操作，可以譯為"代理器"。

	var obj = new Proxy({}, {
	  get: function (target, key, receiver) {
	    console.log(`getting ${key}!`);
	    return Reflect.get(target, key, receiver);
	  },
	  set: function (target, key, value, receiver) {
	    console.log(`setting ${key}!`);
	    return Reflect.set(target, key, value, receiver);
	  }
	});

上面代碼對一個空物件架設了一層攔截，重定義了屬性的讀取（`get`）和設置（`set`）行為。這裡暫時先不解釋具體的語法，只看運行結果。對設置了攔截行為的物件`obj`，去讀寫它的屬性，就會得到下面的結果。

	obj.count = 1
	//  setting count!
	++obj.count
	//  getting count!
	//  setting count!
	//  2

上面代碼說明，Proxy 實際上重載（overload）了點運算符，即用自己的定義覆蓋了語言的原始定義。

ES6 原生提供 Proxy 構造函數，用來生成 Proxy 實例。

	var proxy = new Proxy(target, handler);

Proxy 物件的所有用法，都是上面這種形式，不同的只是`handler`參數的寫法。其中，`new Proxy()`表示生成一個 Proxy 實例，target 參數表示所要攔截的目標物件，`handler`參數也是一個物件，用來定制攔截行為。

下面是另一個攔截讀取屬性行為的例子。

	var proxy = new Proxy({}, {
	  get: function(target, property) {
	    return 35;
	  }
	});

	proxy.time // 35
	proxy.name // 35
	proxy.title // 35

上面代碼中，作為構造函數，Proxy 接受兩個參數。第一個參數是所要代理的目標物件（上例是一個空物件），即如果沒有 Proxy 的介入，操作原來要訪問的就是這個物件；第二個參數是一個配置物件，對於每一個被代理的操作，需要提供一個對應的處理函數，該函數將攔截對應的操作。比如，上面代碼中，配置物件有一個`get`方法，用來攔截對目標物件屬性的訪問請求。`get`方法的兩個參數分別是目標物件和所要訪問的屬性。可以看到，由於攔截函數總是返回 35，所以訪問任何屬性都得到 35。

注意，要使得 Proxy 起作用，必須針對 Proxy 實例（上例是 proxy 物件）進行操作，而不是針對目標物件（上例是空物件）進行操作。

如果`handler`沒有設置任何攔截，那就等同於直接通向原物件。

	var target = {};
	var handler = {};
	var proxy = new Proxy(target, handler);
	proxy.a = 'b';
	target.a // "b"

上面代碼中，`handler`是一個空物件，沒有任何攔截效果，訪問`handeler`就等同於訪問`target`。

一個技巧是將 Proxy 物件，設置到`object.proxy`屬性，從而可以在`object`物件上調用。

	var object = { proxy: new Proxy(target, handler) }

Proxy 實例也可以作為其他物件的原型物件。

	var proxy = new Proxy({}, {
	  get: function(target, property) {
	    return 35;
	  }
	});

	let obj = Object.create(proxy);
	obj.time // 35

上面代碼中，`proxy`物件是`obj`物件的原型，`obj`物件本身並沒有`time`屬性，所以根據原型鏈，會在`proxy`物件上讀取該屬性，導致被攔截。

同一個攔截器函數，可以設置攔截多個操作。

	var handler = {
	  get: function(target, name) {
	    if (name === 'prototype') return Object.prototype;
	    return 'Hello, '+ name;
	  },
	  apply: function(target, thisBinding, args) { return args[0]; },
	  construct: function(target, args) { return args[1]; }
	};

	var fproxy = new Proxy(function(x,y) {
	  return x+y;
	},  handler);

	fproxy(1,2); // 1
	new fproxy(1,2); // 2
	fproxy.prototype; // Object.prototype
	fproxy.foo; // 'Hello, foo'

下面是 Proxy 支持的攔截操作一覽。

對於可以設置、但沒有設置攔截的操作，則直接落在目標物件上，按照原先的方式產生結果。

### (1) `get(target, propKey, receiver)`
攔截物件屬性的讀取，比如`proxy.foo`和`proxy['foo']`，返回類型不限。最後一個參數`receiver`可選，當`target`物件設置了`propKey`屬性的`get`函數時，`receiver`物件會綁定`get`函數的`this`物件。

### (2) `set(target, propKey, value, receiver)`
攔截物件屬性的設置，比如`proxy.foo = v`或`proxy['foo'] = v`，返回一個布林值。

### (3) `has(target, propKey)`
攔截`propKey in proxy`的操作，返回一個布林值。

### (4) `deleteProperty(target, propKey)`
攔截`delete proxy[propKey]`的操作，返回一個布林值。

### (5) `enumerate(target)`
攔截`for (var x in proxy)`，返回一個遍歷器。

### (6) `ownKeys(target)`
攔截`Object.getOwnPropertyNames(proxy)`、`Object.getOwnPropertySymbols(proxy)`、`Object.keys(proxy)`，返回一個陣列。該方法返回物件所有自身的屬性，而`Object.keys()`僅返回物件可遍歷的屬性。

### (7) `getOwnPropertyDescriptor(target, propKey)`
攔截`Object.getOwnPropertyDescriptor(proxy, propKey)`，返回屬性的描述物件。

### (8) `defineProperty(target, propKey, propDesc​​)`
攔截`Object.defineProperty(proxy, propKey, propDesc）`、`Object.defineProperties(proxy, propDescs)`，返回一個布林值。

### (9) `preventExtensions(target)`
攔截`Object.preventExtensions(proxy)`，返回一個布林值。

### (10) `getPrototypeOf(target)`
攔截`Object.getPrototypeOf(proxy)`，返回一個物件。

### (11) `isExtensible(target)`
攔截`Object.isExtensible(proxy)`，返回一個布林值。

### (12) `setPrototypeOf(target, proto)`
攔截`Object.setPrototypeOf(proxy, proto)`，返回一個布林值。
如果目標物件是函數，那麼還有兩種額外操作可以攔截。

### (13) `apply(target, object, args)`
攔截 Proxy 實例作為函數調用的操作，比如`proxy(...args)`、`proxy.call(object, ...args)`、`proxy.apply(...)`。

### (14) `construct(target, args, proxy)`
攔截 Proxy 實例作為構造函數調用的操作，比如`new proxy(...args)`。


## Proxy 實例的方法
下面是上面這些攔截方法的詳細介紹。

### `get()`
`get`方法用於攔截某個屬性的讀取操作。上文已經有一個例子，下面是另一個攔截讀取操作的例子。

	var person = {
	  name: "張三"
	};

	var proxy = new Proxy(person, {
	  get: function(target, property) {
	    if (property in target) {
	      return target[property];
	    } else {
	      throw new ReferenceError("Property \"" + property + "\" does not exist.");
	    }
	  }
	});

	proxy.name // "張三"
	proxy.age  // 抛出一個錯誤

上面代碼表示，如果訪問目標物件不存在的屬性，會拋出一個錯誤。如果沒有這個攔截函數，訪問不存在的屬性，只會返回`undefined`。

`get`方法可以繼承。

	let proto = new Proxy({}, {
	  get(target, propertyKey, receiver) {
	    console.log('GET ' + propertyKey);
	    return target[propertyKey];
	  }
	});

	let obj = Object.create(proto);
	obj.xxx // "GET xxx"

上面代碼中，攔截操作定義在 Prototype 物件上面，所以如果讀取`obj`物件繼承的屬性時，攔截會生效。

下面的例子使用`get`攔截，實現陣列讀取負數的索引。

	function createArray(...elements) {
	  let handler = {
	    get(target, propKey, receiver) {
	      let index = Number(propKey);
	      if (index < 0) {
	        propKey = String(target.length + index);
	      }
	      return Reflect.get(target, propKey, receiver);
	    }
	  };

	  let target = [];
	  target.push(...elements);
	  return new Proxy(target, handler);
	}

	let arr = createArray('a', 'b', 'c');
	arr[-1] // c

上面代碼中，陣列的位置參數是`-1`，就會輸出陣列的倒數最後一個成員。

利用 proxy，可以將讀取屬性的操作（`get`），轉變為執行某個函數，從而實現屬性的鍊式操作。

	var pipe = (function () {
	  var pipe;
	  return function (value) {
	    pipe = [];
	    return new Proxy({}, {
	      get: function (pipeObject, fnName) {
	        if (fnName == "get") {
	          return pipe.reduce(function (val, fn) {
	            return fn(val);
	          }, value);
	        }
	        pipe.push(window[fnName]);
	        return pipeObject;
	      }
	    });
	  }
	}());

	var double = n => n * 2;
	var pow = n => n * n;
	var reverseInt = n => n.toString().split('').reverse().join('') | 0;

	pipe(3).double.pow.reverseInt.get
	// 63

上面代碼設置Proxy以後，達到了將函數名鍊式使用的效果。

### `set()`
`set`方法用來攔截某個屬性的賦值操作。

假定 Person 物件有一個`age`屬性，該屬性應該是一個不大於 200 的整數，那麼可以使用 Proxy 物件保證`age`的屬性值符合要求。

	let validator = {
	  set: function(obj, prop, value) {
	    if (prop === 'age') {
	      if (!Number.isInteger(value)) {
	        throw new TypeError('The age is not an integer');
	      }
	      if (value > 200) {
	        throw new RangeError('The age seems invalid');
	      }
	    }

	    // 對於 age 以外的屬性，直接保存
	    obj[prop] = value;
	  }
	};

	let person = new Proxy({}, validator);

	person.age = 100;

	person.age // 100
	person.age = 'young' // 報錯
	person.age = 300 // 報錯

上面代碼中，由於設置了存值函數`set`，任何不符合要求的`age`屬性賦值，都會拋出一個錯誤。利用`set`方法，還可以資料綁定，即每當物件發生變化時，會自動更新 DOM。

有時，我們會在物件上面設置內部屬性，屬性名的第一個字符使用下劃線開頭，表示這些屬性不應該被外部使用。結合`get`和`set`方法，就可以做到防止這些內部屬性被外部讀寫。

	var handler = {
	  get (target, key) {
	    invariant(key, 'get');
	    return target[key];
	  },
	  set (target, key, value) {
	    invariant(key, 'set');
	    return true;
	  }
	}
	function invariant (key, action) {
	  if (key[0] === '_') {
	    throw new Error(`Invalid attempt to ${action} private "${key}" property`);
	  }
	}
	var target = {};
	var proxy = new Proxy(target, handler);
	proxy._prop
	// Error: Invalid attempt to get private "_prop" property
	proxy._prop = 'c'
	// Error: Invalid attempt to set private "_prop" property

上面代碼中，只要讀寫的屬性名的第一個字符是下劃線，一律拋錯，從而達到禁止讀寫內部屬性的目的。

### `apply()`
`apply`方法攔截函數的調用、`call`和`apply`操作。

	var handler = {
	  apply (target, ctx, args) {
	    return Reflect.apply(...arguments);
	  }
	}

`apply`方法可以接受三個參數，分別是目標物件、目標物件的上下文物件（this）和目標物件的參數陣列。

下面是一個例子。

	var target = function () { return 'I am the target'; };
	var handler = {
	  apply: function () {
	    return 'I am the proxy';
	  }
	};

	var p = new Proxy(target, handler);

	p() === 'I am the proxy';
	// true

上面代碼中，變數`p`是 Proxy 的實例，當它作為函數調用時（`p()`），就會被`apply`方法攔截，返回一個字串。

下面是另外一個例子。

	var twice = {
	  apply (target, ctx, args) {
	    return Reflect.apply(...arguments) * 2;
	  }
	};
	function sum (left, right) {
	  return left + right;
	};
	var proxy = new Proxy(sum, twice);
	proxy(1, 2) // 6
	proxy.call(null, 5, 6) // 22
	proxy.apply(null, [7, 8]) // 30

上面代碼中，每當執行`proxy`函數，就會被`apply`方法攔截。

另外，直接調用`Reflect.apply`方法，也會被攔截。

	Reflect.apply(proxy, null, [9, 10]) // 38

### `has()`
`has`方法可以隱藏某些屬性，不被`in`操作符發現。

	var handler = {
	  has (target, key) {
	    if (key[0] === '_') {
	      return false;
	    }
	    return key in target;
	  }
	};
	var target = { _prop: 'foo', prop: 'foo' };
	var proxy = new Proxy(target, handler);
	'_prop' in proxy // false

上面代碼中，如果原物件的屬性名的第一個字符是下劃線，`proxy.has`就會返回`false`，從而不會被`in`運算符發現。

如果原物件不可配置或者禁止擴展，這時`has`攔截會報錯。

	var obj = { a: 10 };
	Object.preventExtensions(obj);
	var p = new Proxy(obj, {
	  has: function(target, prop) {
	    return false;
	  }
	});

	"a" in p; // TypeError is thrown

上面代碼中，`obj`物件禁止擴展，結果使用`has`攔截就會報錯。

### `construct()`
`construct`方法用於攔截`new`命令。

	var handler = {
	  construct (target, args) {
	    return new target(...args);
	  }
	}

下面是一個例子。

	var p = new Proxy(function() {}, {
	  construct: function(target, args) {
	    console.log('called: ' + args.join(', '));
	    return { value: args[0] * 10 };
	  }
	});

	new p(1).value
	// "called: 1"
	// 10

如果`construct`方法返回的不是物件，就會拋出錯誤。

	var p = new Proxy(function() {}, {
	  construct: function(target, argumentsList) {
	    return 1;
	  }
	});

	new p() // 報錯

### `deleteProperty()`
`deleteProperty`方法用於攔截`delete`操作，如果這個方法拋出錯誤或者返回`false`，當前屬性就無法被`delete`命令刪除。

	var handler = {
	  deleteProperty (target, key) {
	    invariant(key, 'delete');
	    return true;
	  }
	}
	function invariant (key, action) {
	  if (key[0] === '_') {
	    throw new Error(`Invalid attempt to ${action} private "${key}" property`);
	  }
	}

	var target = { _prop: 'foo' }
	var proxy = new Proxy(target, handler)
	delete proxy._prop
	// Error: Invalid attempt to delete private "_prop" property

上面代碼中，`deleteProperty`方法攔截了`delete`操作符，刪除第一個字符為下劃線的屬性會報錯。

### `defineProperty()`
`defineProperty`方法攔截了`Object.defineProperty`操作。

	var handler = {
	  defineProperty (target, key, descriptor) {
	    return false
	  }
	}
	var target = {}
	var proxy = new Proxy(target, handler)
	proxy.foo = 'bar'
	// TypeError: proxy defineProperty handler returned false for property '"foo"'

上面代碼中，`defineProperty`方法返回`false`，導致添加新屬性會拋出錯誤。

### `enumerate()`
`enumerate`方法用來攔截`for...in`循環。注意與 Proxy 物件的`has`方法區分，後者用來攔截`in`操作符，對`for...in`循環無效。

	var handler = {
	  enumerate (target) {
	    return Object.keys(target).filter(key => key[0] !== '_')[Symbol.iterator]();
	  }
	}
	var target = { prop: 'foo', _bar: 'baz', _prop: 'foo' }
	var proxy = new Proxy(target, handler)
	for (let key in proxy) {
	  console.log(key);
	  // "prop"
	}

上面代碼中，`enumerate`方法取出原物件的所有屬性名，將其中第一個字符等於下劃線的都過濾掉，然後返回這些符合條件的屬性名的一個遍歷器物件，供`for...in`循環消費。

下面是另一個例子。

	var p = new Proxy({}, {
	  enumerate(target) {
	    console.log("called");
	    return ["a", "b", "c"][Symbol.iterator]();
	  }
	});

	for (var x in p) {
	  console.log(x);
	}
	// "called"
	// "a"
	// "b"
	// "c"

如果`enumerate`方法返回的不是一個物件，就會報錯。

	var p = new Proxy({}, {
	  enumerate(target) {
	    return 1;
	  }
	});

	for (var x in p) {} // 報錯

### `getOwnPropertyDescriptor()`
`getOwnPropertyDescriptor`方法攔截`Object.getOwnPropertyDescriptor`，返回一個屬性描述物件或者`undefined`。

	var handler = {
	  getOwnPropertyDescriptor (target, key) {
	    if (key[0] === '_') {
	      return
	    }
	    return Object.getOwnPropertyDescriptor(target, key)
	  }
	}
	var target = { _foo: 'bar', baz: 'tar' };
	var proxy = new Proxy(target, handler);
	Object.getOwnPropertyDescriptor(proxy, 'wat')
	// undefined
	Object.getOwnPropertyDescriptor(proxy, '_foo')
	// undefined
	Object.getOwnPropertyDescriptor(proxy, 'baz')
	// { value: 'tar', writable: true, enumerable: true, configurable: true }

上面代碼中，`handler.getOwnPropertyDescriptor`方法對於第一個字符為下劃線的屬性名會返回`undefined`。

### `getPrototypeOf()`
`getPrototypeOf`方法主要用來攔截`Object.getPrototypeOf()`運算符，以及其他一些操作。

* `Object.prototype.__proto__`
* `Object.prototype.isPrototypeOf()`
* `Object.getPrototypeOf()`
* `Reflect.getPrototypeOf()`
* `instanceof`運算符
下面是一個例子。

	var proto = {};
	var p = new Proxy({}, {
	  getPrototypeOf(target) {
	    return proto;
	  }
	});
	Object.getPrototypeOf(p) === proto // true

上面代碼中，`getPrototypeOf`方法攔截`Object.getPrototypeOf()`，返回`proto`物件。

### `isExtensible()`
`isExtensible`方法攔截`Object.isExtensible`操作。

	var p = new Proxy({}, {
	  isExtensible: function(target) {
	    console.log("called");
	    return true;
	  }
	});

	Object.isExtensible(p)
	// "called"
	// true

上面代碼設置了`isExtensible`方法，在調用`Object.isExtensible`時會輸出`called`。

這個方法有一個強限制，如果不能滿足下面的條件，就會拋出錯誤。

	Object.isExtensible(proxy) === Object.isExtensible(target)

下面是一個例子。

	var p = new Proxy({}, {
	  isExtensible: function(target) {
	    return false;
	  }
	});

	Object.isExtensible(p); // 報錯

### `ownKeys()`
`ownKeys`方法用來攔截`Object.keys()`操作。

	let target = {};

	let handler = {
	  ownKeys(target) {
	    return ['hello', 'world'];
	  }
	};

	let proxy = new Proxy(target, handler);

	Object.keys(proxy)
	// [ 'hello', 'world' ]

上面代碼攔截了對於`target`物件的`Object.keys()`操作，返回預先設定的陣列。

下面的例子是攔截第一個字符為下劃線的屬性名。

	var target = {
	  _bar: 'foo',
	  _prop: 'bar',
	  prop: 'baz'
	};

	var handler = {
	  ownKeys (target) {
	    return Reflect.ownKeys(target).filter(key => key[0] !== '_');
	  }
	};

	var proxy = new Proxy(target, handler);
	for (let key of Object.keys(proxy)) {
	  console.log(key)
	}
	// "baz"

### `preventExtensions()`
`preventExtensions`方法攔截`Object.preventExtensions()`。該方法必須返回一個布林值。

這個方法有一個限制，只有當`Object.isExtensible(proxy)`為`false`（即不可擴展）時，`proxy.preventExtensions`才能返回`true`，否則會報錯。

	var p = new Proxy({}, {
	  preventExtensions: function(target) {
	    return true;
	  }
	});

	Object.preventExtensions(p); // 報錯

上面代碼中，`proxy.preventExtensions`方法返回`true`，但這時`Object.isExtensible(proxy)`會返回`true`，因此報錯。

為了防止出現這個問題，通常要在`proxy.preventExtensions`方法裡面，調用一次`Object.preventExtensions`。

	var p = new Proxy({}, {
	  preventExtensions: function(target) {
	    console.log("called");
	    Object.preventExtensions(target);
	    return true;
	  }
	});

	Object.preventExtensions(p)
	// "called"
	// true

### `setPrototypeOf()`
`setPrototypeOf`方法主要用來攔截`Object.setPrototypeOf`方法。

下面是一個例子。

	var handler = {
	  setPrototypeOf (target, proto) {
	    throw new Error('Changing the prototype is forbidden');
	  }
	}
	var proto = {};
	var target = function () {};
	var proxy = new Proxy(target, handler);
	proxy.setPrototypeOf(proxy, proto);
	// Error: Changing the prototype is forbidden

上面代碼中，只要修改`target`的原型物件，就會報錯。

## `Proxy.revocable()`
`Proxy.revocable`方法返回一個可取消的 Proxy 實例。

	let target = {};
	let handler = {};

	let {proxy, revoke} = Proxy.revocable(target, handler);

	proxy.foo = 123;
	proxy.foo // 123

	revoke();
	proxy.foo // TypeError: Revoked

`Proxy.revocable`方法返回一個物件，該物件的`proxy`屬性是 Proxy 實例，`revoke`屬性是一個函數，可以取消 Proxy 實例。上面代碼中，當執行`revoke`函數之後，再訪問 Proxy 實例，就會拋出一個錯誤。

## Reflect 概述
`Reflect`物件與`Proxy`物件一樣，也是 ES6 為了操作物件而提供的新 API。`Reflect`物件的設計目的有這樣幾個。

（1）將`Object`物件的一些明顯屬於語言內部的方法（比如`Object.defineProperty`），放到`Reflect`物件上。現階段，某些方法同時在`Object`和`Reflect`物件上部署，未來的新方法將只部署在`Reflect`物件上。

（2）修改某些`Object`方法的返回結果，讓其變得更合理。比如，`Object.defineProperty(obj, name, desc)`在無法定義屬性時，會拋出一個錯誤，而`Reflect.defineProperty(obj, name, desc)`則會返回`false`。

	// 老寫法
	try {
	  Object.defineProperty(target, property, attributes);
	  // success
	} catch (e) {
	  // failure
	}

	// 新寫法
	if (Reflect.defineProperty(target, property, attributes)) {
	  // success
	} else {
	  // failure
	}

（3）讓`Object`操作都變成函數行為。某些`Object`操作是命令式，比如`name in obj`和`delete obj[name]`，而`Reflect.has(obj, name)`和`Reflect.deleteProperty(obj, name)`讓它們變成了函數行為。

	// 老寫法
	'assign' in Object // true

	// 新寫法
	Reflect.has(Object, 'assign') // true

（4）`Reflect`物件的方法與`Proxy`物件的方法一一對應，只要是`Proxy`物件的方法，就能在`Reflect`物件上找到對應的方法。這就讓`Proxy`物件可以方便地調用對應的`Reflect`方法，完成預設行為，作為修改行為的基礎。也就是說，不管`Proxy`怎麼修改預設行為，你總可以在`Reflect`上獲取預設行為。

	Proxy(target, {
	  set: function(target, name, value, receiver) {
	    var success = Reflect.set(target,name, value, receiver);
	    if (success) {
	      log('property ' + name + ' on ' + target + ' set to ' + value);
	    }
	    return success;
	  }
	});

上面代碼中，`Proxy`方法攔截`target`物件的屬性賦值行為。它採用`Reflect.set`方法將值賦值給物件的屬性，然後再部署額外的功能。

下面是另一個例子。

	var loggedObj = new Proxy(obj, {
	  get(target, name) {
	    console.log('get', target, name);
	    return Reflect.get(target, name);
	  },
	  deleteProperty(target, name) {
	    console.log('delete' + name);
	    return Reflect.deleteProperty(target, name);
	  },
	  has(target, name) {
	    console.log('has' + name);
	    return Reflect.has(target, name);
	  }
	});

上面代碼中，每一個`Proxy`物件的攔截操作（`get`、`delete`、`has`），內部都調用對應的`Reflect`方法，保證原生行為能夠正常執行。添加的工作，就是將每一個操作輸出一行日誌。

有了`Reflect`物件以後，很多操作會更易讀。

	// 老寫法
	Function.prototype.apply.call(Math.floor, undefined, [1.75]) // 1

	// 新寫法
	Reflect.apply(Math.floor, undefined, [1.75]) // 1

## Reflect 物件的方法
`Reflect`物件的方法清單如下，共14個。

* `Reflect.apply(target,thisArg,args)`
* `Reflect.construct(target,args)`
* `Reflect.get(target,name,receiver)`
* `Reflect.set(target,name,value,receiver)`
* `Reflect.defineProperty(target,name,desc)`
* `Reflect.deleteProperty(target,name)`
* `Reflect.has(target,name)`
* `Reflect.ownKeys(target)`
* `Reflect.enumerate(target)`
* `Reflect.isExtensible(target)`
* `Reflect.preventExtensions(target)`
* `Reflect.getOwnPropertyDescriptor(target, name)`
* `Reflect.getPrototypeOf(target)`
* `Reflect.setPrototypeOf(target, prototype)`

上面這些方法的作用，大部分與`Object`物件的同名方法的作用都是相同的，而且它與`Proxy`物件的方法是一一對應的。下面是對其中幾個方法的解釋。

### (1) `Reflect.get(target, name, receiver)`
查找並返回`target`物件的`name`屬性，如果沒有該屬性，則返回`undefined`。

如果`name`屬性部署了讀取函數，則讀取函數的`this`綁定`receiver`。

	var obj = {
	  get foo() { return this.bar(); },
	  bar: function() { ... }
	}

	// 下面語句會讓 this.bar()
	// 變成調用 wrapper.bar()
	Reflect.get(obj, "foo", wrapper);

### (2) `Reflect.set(target, name, value, receiver)`
設置`target`物件的`name`屬性等於`value`。如果`name`屬性設置了賦值函數，則賦值函數的`this`綁定`receiver`。

### (3) `Reflect.has(obj, name)`
等同於`name in obj`。

### (4) `Reflect.deleteProperty(obj, name)`
等同於`delete obj[name]`。

### (5) `Reflect.construct(target, args)`
等同於`new target(...args)`，這提供了一種不使用`new`，來調用構造函數的方法。

### (6) `Reflect.getPrototypeOf(obj)`
讀取物件的`__proto__`屬性，對應`Object.getPrototypeOf(obj)`。

### (7) `Reflect.setPrototypeOf(obj, newProto)`
設置物件的`__proto__`屬性，對應`Object.setPrototypeOf(obj, newProto)`。

### (8) `Reflect.apply(fun,thisArg,args)`
等同於`Function.prototype.apply.call(fun,thisArg,args)`。一般來說，如果要綁定一個函數的`this`物件，可以這樣寫`fn.apply(obj, args)`，但是如果函數定義了自己的`apply`方法，就只能寫成`Function.prototype.apply.call(fn, obj, args)`，採用`Reflect`物件可以簡化這種操作。

另外，需要注意的是，`Reflect.set()、Reflect.defineProperty()`、`Reflect.freeze()`、`Reflect.seal()`和`Reflect.preventExtensions()`返回一個布林值，表示操作是否成功。它們對應的`Object`方法，失敗時都會拋出錯誤。

	// 失敗時拋出錯誤
	Object.defineProperty(obj, name, desc);
	// 失敗時回傳 false
	Reflect.defineProperty(obj, name, desc);

上面代碼中，`Reflect.defineProperty`方法的作用與`Object.defineProperty`是一樣的，都是為物件定義一個屬性。但是，`Reflect.defineProperty`方法失敗時，不會拋出錯誤，只會返回`false`。
