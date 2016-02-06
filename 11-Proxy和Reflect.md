# Proxy 和 Reflect
## Proxy 概述
Proxy 用於修改某些操作的默認行為，等同於在語言層面做出修改，所以屬於一種"元編程"（meta programming），即對編程語言進行編程。

Proxy 可以理解成，在目標對象之前架設一層"攔截"，外界對該對象的訪問，都必須先通過這層攔截，因此提供了一種機制，可以對外界的訪問進行過濾和改寫。Proxy 這個詞的原意是代理，用在這裡表示由它來"代理"某些操作，可以譯為"代理器"。

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

上面代碼對一個空物件架設了一層攔截，重定義了屬性的讀取（`get`）和設置（`set`）行為。這裡暫時先不解釋具體的語法，只看運行結果。對設置了攔截行為的對象`obj`，去讀寫它的屬性，就會得到下面的結果。

	obj.count = 1
	//  setting count!
	++obj.count
	//  getting count!
	//  setting count!
	//  2

上面代碼說明，Proxy 實際上重載（overload）了點運算符，即用自己的定義覆蓋了語言的原始定義。

ES6 原生提供 Proxy 構造函數，用來生成 Proxy 實例。

	var proxy = new Proxy(target, handler);

Proxy 物件的所有用法，都是上面這種形式，不同的只是`handler`參數的寫法。其中，`new Proxy()`表示生成一個 Proxy 實例，target 參數表示所要攔截的目標對象，`handler`參數也是一個對象，用來定制攔截行為。

下面是另一個攔截讀取屬性行為的例子。

	var proxy = new Proxy({}, {
	  get: function(target, property) {
	    return 35;
	  }
	});
	
	proxy.time // 35
	proxy.name // 35
	proxy.title // 35

上面代碼中，作為構造函數，Proxy 接受兩個參數。第一個參數是所要代理的目標對象（上例是一個空對象），即如果沒有 Proxy 的介入，操作原來要訪問的就是這個對象；第二個參數是一個配置對象，對於每一個被代理的操作，需要提供一個對應的處理函數，該函數將攔截對應的操作。比如，上面代碼中，配置對像有一個`get`方法，用來攔截對目標對象屬性的訪問請求。`get`方法的兩個參數分別是目標對象和所要訪問的屬性。可以看到，由於攔截函數總是返回 35，所以訪問任何屬性都得到 35。

注意，要使得 Proxy 起作用，必須針對 Proxy 實例（上例是 proxy 對象）進行操作，而不是針對目標對象（上例是空對象）進行操作。

如果`handler`沒有設置任何攔截，那就等同於直接通向原對象。

	var target = {};
	var handler = {};
	var proxy = new Proxy(target, handler);
	proxy.a = 'b';
	target.a // "b"

上面代碼中，`handler`是一個空對象，沒有任何攔截效果，訪問`handeler`就等同於訪問`target`。

一個技巧是將 Proxy 對象，設置到`object.proxy`屬性，從而可以在`object`對像上調用。

	var object = { proxy: new Proxy(target, handler) }

Proxy 實例也可以作為其他對象的原型對象。

	var proxy = new Proxy({}, {
	  get: function(target, property) {
	    return 35;
	  }
	});
	
	let obj = Object.create(proxy);
	obj.time // 35

上面代碼中，`proxy`對像是`obj`對象的原型，`obj`對象本身並沒有`time`屬性，所以根據原型鏈，會在`proxy`對像上讀取該屬性，導致被攔截。

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

對於可以設置、但沒有設置攔截的操作，則直接落在目標對像上，按照原先的方式產生結果。

### (1) `get(target, propKey, receiver)`
### (2) `set(target, propKey, value, receiver)`
### (3) `has(target, propKey)`
### (4) `deleteProperty(target, propKey)`

## Proxy 實例的方法
## `Proxy.revocable()`
## Reflect 概述
## Reflect 對象的方法