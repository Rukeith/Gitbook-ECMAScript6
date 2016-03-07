# `Set`和`Map`資料結構

## `Set`

### 基本用法

ES6 提供了新的數據結構`Set`。它類似於數組，但是成員的值都是**唯一**的，沒有重複的值。

`Set`本身是一個構造函數，用來生成 Set 數據結構。

	var s = new Set();
	
	[2,3,5,4,5,2,2].map(x => s.add(x))
	
	for (let i of s) {console.log(i)}
	// 2 3 5 4

上面代碼通過`add`方法向`Set`結構加入成員，結果表明`Set`結構不會添加重複的值。

`Set`函數可以接受一個數組（或類似數組的對象）作為參數，用來初始化。

	var set = new Set([1, 2, 3, 4, 4])
	[...set]
	// [1, 2, 3, 4]
	
	var items = new Set([1, 2, 3, 4, 5, 5, 5, 5]);
	items.size // 5
	
	function divs () {
	  return [...document.querySelectorAll('div')]
	}
	
	var set = new Set(divs())
	set.size // 56
	
	// 類似於
	divs().forEach(div => set.add(div))
	set.size // 56

向`Set`加入值的時候，**不會發生類型轉換**，所以 5 和 "5" 是兩個不同的值。`Set`內部判斷兩個值是否不同，使用的算法叫做 "Same-value equality"，它類似於精確相等運算符（`===`），主要的區別是`NaN`等於自身，而精確相等運算符認為`NaN`不等於自身。

	let set = new Set();
	let a = NaN;
	let b = NaN;
	set.add(a);
	set.add(b);
	set // Set {NaN}

上面代碼向`Set`實例添加了兩個`NaN`，但是只能加入一個。這表明，在`Set`內部，兩個`NaN`是相等。

另外，兩個對象總是不相等的。

	let set = new Set();
	
	set.add({})
	set.size // 1
	
	set.add({})
	set.size // 2

上面代碼表示，由於兩個空對像不相等，所以它們被視為兩個值。

### `Set`實例的屬性和方法
`Set`結構的實例有以下屬性。

* `Set.prototype.constructor`：構造函數，默認就是`Set`函數。
* `Set.prototype.size`：返回`Set`實例的成員總數。

`Set`實例的方法分為兩大類：操作方法（用於操作數據）和遍歷方法（用於遍歷成員）。下面先介紹四個操作方法。

* `add(value)`：添加某個值，返回Set結構本身。
* `delete(value)`：刪除某個值，返回一個布爾值，表示刪除是否成功。
* `has(value)`：返回一個布爾值，表示該值是否為`Set`的成員。
* `clear()`：清除所有成員，沒有返回值。

上面這些屬性和方法的實例如下。

	s.add(1).add(2).add(2);
	// 注意 2 被加入了两次
	
	s.size // 2
	
	s.has(1) // true
	s.has(2) // true
	s.has(3) // false
	
	s.delete(2);
	s.has(2) // false

下面是一個對比，看看在判斷是否包括一個鍵上面，`Object`結構和`Set`結構的寫法不同。

	// 物件的寫法
	var properties = {
	  "width": 1,
	  "height": 1
	};
	
	if (properties[someName]) {
	  // do something
	}
	
	// Set 的寫法
	var properties = new Set();
	
	properties.add("width");
	properties.add("height");
	
	if (properties.has(someName)) {
	  // do something
	}

`Array.from`方法可以將`Set`結構轉為數組。

	var items = new Set([1, 2, 3, 4, 5]);
	var array = Array.from(items);

這就提供了一種去除數組的重複元素的方法。

	function dedupe(array) {
	  return Array.from(new Set(array));
	}
	
	dedupe([1,1,2,3]) // [1, 2, 3]

### 遍歷操作

Set 結構的實例有四個遍歷方法，可以用於遍歷成員。

* `keys()`：返回一個鍵名的遍歷器
* `values​​()`：返回一個鍵值的遍歷器
* `entries()`：返回一個鍵值對的遍歷器
* `forEach()`：使用回調函數遍歷每個成員

`key`方法、`value`方法、`entries`方法返回的都是遍歷器對象。由於`Set`結構沒有鍵名，只有鍵值（或者說鍵名和鍵值是同一個值），所以`key`方法和`value`方法的行為完全一致。

	let set = new Set(['red', 'green', 'blue']);
	
	for ( let item of set.keys() ){
	  console.log(item);
	}
	// red
	// green
	// blue
	
	for ( let item of set.values() ){
	  console.log(item);
	}
	// red
	// green
	// blue
	
	for ( let item of set.entries() ){
	  console.log(item);
	}
	// ["red", "red"]
	// ["green", "green"]
	// ["blue", "blue"]

上面代碼中，`entries`方法返回的遍歷器，同時包括鍵名和鍵值，所以每次輸出一個數組，它的兩個成員完全相等。

`Set`結構的實例默認可遍歷，它的默認遍歷器生成函數就是它的`values`​​方法。

	Set.prototype[Symbol.iterator] === Set.prototype.values
	// true

這意味著，可以省略`values`​​方法，直接用`for...of`循環遍歷`Set`。

	let set = new Set(['red', 'green', 'blue']);
	
	for (let x of set) {
	  console.log(x);
	}
	// red
	// green
	// blue

由於擴展運算符（`...`）內部使用`for...of`循環，所以也可以用於`Set`結構。

	let set = new Set(['red', 'green', 'blue']);
	let arr = [...set];
	// ['red', 'green', 'blue']

這就提供了另一種快速的去除數組重複元素的方法。

	let arr = [3, 5, 2, 2, 5, 5];
	let unique = [...new Set(arr)];
	// [3, 5, 2]

而且，數組的`map`和`filter`方法也可以用於`Set`了。

	let set = new Set([1, 2, 3]);
	set = new Set([...set].map(x => x * 2));
	// 返回 Set 結構：{2, 4, 6}
	
	let set = new Set([1, 2, 3, 4, 5]);
	set = new Set([...set].filter(x => (x % 2) == 0));
	// 返回 Set 結構：{2, 4}

因此使用 Set，可以很容易地實現並集（Union）、交集（Intersect）和差集（Difference）。

	let a = new Set([1, 2, 3]);
	let b = new Set([4, 3, 2]);
	
	// 聯集
	let union = new Set([...a, ...b]);
	// [1, 2, 3, 4]
	
	// 交集
	let intersect = new Set([...a].filter(x => b.has(x)));
	// [2, 3]
	
	// 差集
	let difference = new Set([...a].filter(x => !b.has(x)));
	// [1]

`Set`結構的實例的`forEach`方法，用於對每個成員執行某種操作，沒有返回值。

	let set = new Set([1, 2, 3]);
	set.forEach((value, key) => console.log(value * 2) )
	// 2
	// 4
	// 6

上面代碼說明，`forEach`方法的參數就是一個處理函數。該函數的參數依次為鍵值、鍵名 ​​、集合本身（上例省略了該參數）。另外，`forEach`方法還可以有第二個參數，表示綁定的`this`對象。

如果想在遍歷操作中，同步改變原來的`Set`結構，目前沒有直接的方法，但有兩種變通方法。一種是利用原`Set`結構映射出一個新的結構，然後賦值給原來的`Set`結構；另一種是利用`Array.from`方法。

	// 方法一
	let set = new Set([1, 2, 3]);
	set = new Set([...set].map(val => val * 2));
	// set的值是2, 4, 6
	
	// 方法二
	let set = new Set([1, 2, 3]);
	set = new Set(Array.from(set, val => val * 2));
	// set的值是2, 4, 6

上面代碼提供了兩種方法，直接在遍歷操作中改變原來的`Set`結構。

## `WeakSet`
`WeakSet`結構與`Set`類似，也是不重複的值的集合。但是，它與`Set`有兩個區別。

* 首先，`WeakSet`的成員只能是對象，而不能是其他類型的值。
* 其次，`WeakSet`中的對像都是弱引用，即垃圾回收機制不考慮`WeakSet`對該對象的引用，也就是說，如果其他對像都不再引用該對象，那麼垃圾回收機制會自動回收該對象所佔用的內存，不考慮該對像還存在於`WeakSet`之中。這個特點意味著，無法引用`WeakSet`的成員，因此`WeakSet`是不可遍歷的。

	var ws = new WeakSet();
	ws.add(1);
	// TypeError: Invalid value used in weak set
	ws.add(Symbol());
	// TypeError: invalid value used in weak set

上面代碼試圖向`WeakSet`添加一個數值和`Symbol`值，結果報錯。  
`WeakSet`是一個構造函數，可以使用`new`命令，創建`WeakSet`數據結構。

	var ws = new WeakSet();

作為構造函數，`WeakSet`可以接受一個數組或類似數組的對像作為參數。（實際上，任何具有`iterable`接口的對象，都可以作為`WeakSet`的參數。）該數組的所有成員，都會自動成為`WeakSet`實例對象的成員。

	var a = [[1, 2], [3, 4]];
	var ws = new WeakSet(a);

上面代碼中，`a`是一個數組，它有兩個成員，也都是數組。將`a`作為`WeakSet`構造函數的參數，`a`的成員會自動成為`WeakSet`的成員。

`WeakSet`結構有以下三個方法。

* WeakSet.prototype.add(value)：向`WeakSet`實例添加一個新成員。
* WeakSet.prototype.delete(value)：清除`WeakSet`實例的指定成員。
* WeakSet.prototype.has(value)：返回一個布林值，表示某個值是否在`WeakSet`實例之中。

下面是一個例子：

	var ws = new WeakSet();
	var obj = {};
	var foo = {};
	
	ws.add(window);
	ws.add(obj);
	
	ws.has(window); // true
	ws.has(foo);    // false
	 
	ws.delete(window);
	ws.has(window);    // false

`WeakSet`沒有`size`屬性，沒有辦法遍歷它的成員。

	ws.size // undefined
	ws.forEach // undefined
	 
	ws.forEach(function (item) {console.log('WeakSet has ' + item); })
	// TypeError: undefined is not a function

上面代碼試圖獲取`size`和`forEach`屬性，結果都不能成功。

`WeakSet`不能遍歷，是因為成員都是弱引用，隨時可能消失，遍歷機制無法保證成員的存在，很可能剛剛遍歷結束，成員就取不到了。  
*`WeakSet`的一個用處，是儲存`DOM`節點，而不用擔心這些節點從文檔移除時，會引發內存洩漏。*

下面是`WeakSet`的另一個例子：

	const foos = new WeakSet();
	class Foo {
	  constructor() {
	    foos.add(this);
	  } 
	  method() {
	    if (!foos.has(this)) {
	      throw new TypeError('Foo.prototype.method只能在Foo的實例上調用！');
	    }
	  }
	}

上面代碼保證了`Foo`的實例方法，只能在`Foo`的實例上調用。這裡使用`WeakSet`的好處是，`foos`對實例的引用，不會被計入內存回收機制，所以刪除實例的時候，不用考慮`foos`，也不會出現內存洩漏。

## `Map`
### `Map`結構的目的和基本用法
JavaScript 的對象（Object），本質上是鍵值對的集合（Hash 結構），但是只能用字符串當作鍵。這給它的使用帶來了很大的限制。

	var data = {}; 
	var element = document.getElementById("myDiv");
	
	data[element] = metadata;
	data["[Object HTMLDivElement]"] // metadata

上面代碼原意是將一個 DOM 節點作為對象`data`的鍵，但是由於對像只接受字符串作為鍵名，所以 element 被自動轉為字符串`[Object HTMLDivElement]`。

為了解決這個問題，ES6 提供了`Map`數據結構。它類似於對象，也是鍵值對的集合，但是 "鍵" 的範圍不限於字符串，各種類型的值（包括對象）都可以當作鍵。也就是說，`Object`結構提供了 "字符串 — 值" 的對應，`Map`結構提供了 "值 — 值" 的對應，是一種更完善的 Hash 結構實現。如果你需要 "鍵值對" 的數據結構，`Map`比`Object`更合適。

	var m = new Map();
	var o = {p: "Hello World"};
	
	m.set(o, "content");
	m.get(o); // "content"
	 
	m.has(o); 	// true
	m.delete(o); // true
	m.has(o); 	// false

上面代碼使用`set`方法，將對象`o`當作`m`的一個鍵，然後又使用`get`方法讀取這個鍵，接著使用`delete`方法刪除了這個鍵。

作為構造函數，`Map`也可以接受一個數組作為參數。該數組的成員是一個個表示鍵值對的數組。

	var map = new Map([["name", "張三"], ["title", "Author"]]);
	
	map.size; // 2
	map.has("name");  // true
	map.get("name");  // "張三"
	map.has("title"); // true
	map.get("title"); // "Author"

上面代碼在新建`Map`實例時，就指定了兩個鍵`name`和`title`。

`Map`構造函數接受數組作為參數，實際上執行的是下面的算法。

	var items = [ 
	  ["name", "張三"],
	  ["title", "Author"]
	]; 
	var map = new Map();
	items.forEach(([key, value]) => map.set(key, value));

如果對同一個鍵多次賦值，後面的值將覆蓋前面的值。

	let map = new Map();

	map.set(1, 'aaa');
	map.set(1, 'bbb');
	
	map.get(1); // "bbb"

上面代碼對鍵`1`連續賦值兩次，後一次的值覆蓋前一次的值。

如果讀取一個未知的鍵，則返回`undefined`。

	new Map().get('asfddfsasadf');
	// undefined

注意，只有對同一個對象的引用，`Map`結構才將其視為同一個鍵。這一點要非常小心。

	var map = new Map();
	
	map.set(['a'], 555);
	map.get(['a']); // undefined

上面代碼的set和get方法，表面是針對同一個鍵，但實際上這是兩個值，內存地址是不一樣的，因此get方法無法讀取該鍵，返回undefined。

同理，同樣的值的兩個實例，在Map結構中被視為兩個鍵。

var map =  new  Map ( ) ;

var k1 =  [ 'a' ] ; 
var k2 ​​=  [ 'a' ] ;

map
. set ( k1 ,  111 ) 
. set ( k2 ,  222 ) ;

map . get ( k1 ) // 111
 map . get ( k2 ) // 222
上面代碼中，變量k1和k2的值是一樣的，但是它們在Map結構中被視為兩個鍵。

由上可知，Map的鍵實際上是跟內存地址綁定的，只要內存地址不一樣，就視為兩個鍵。這就解決了同名屬性碰撞（clash）的問題，我們擴展別人的庫的時候，如果使用對像作為鍵名，就不用擔心自己的屬性與原作者的屬性同名。

如果Map的鍵是一個簡單類型的值（數字、字符串、布爾值），則只要兩個值嚴格相等，Map將其視為一個鍵，包括0和-0。另外，雖然NaN不嚴格相等於自身，但Map將其視為同一個鍵。

let map =  new  Map ( ) ;

map . set ( NaN ,  123 ) ; 
map . get ( NaN ) // 123
 
map . set ( - 0 ,  123 ) ; 
map . get ( + 0 ) // 123

## `WeakMap`