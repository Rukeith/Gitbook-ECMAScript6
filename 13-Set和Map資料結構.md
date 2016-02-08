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

## `Map`

## `WeakMap`