# Symbol
## 概述
ES5 的對象屬性名都是字符串，這容易造成屬性名的衝突。比如，你使用了一個他人提供的對象，但又想為這個對象添加新的方法（mixin模式），新方法的名字就有可能與現有方法產生衝突。如果有一種機制，保證每個屬性的名字都是獨一無二的就好了，這樣就從根本上防止屬性名的衝突。這就是 ES6 引入 Symbol 的原因。

ES6 引入了一種新的原始數據類型 Symbol，表示獨一無二的值。它是 JavaScript 語言的第七種數據類型，前六種是：Undefined、Null、布林值（Boolean）、字符串（String）、數值（Number）、對象（Object）。

Symbol 值通過 Symbol 函數生成。這就是說，對象的屬性名現在可以有兩種類型，一種是原來就有的字符串，另一種就是新增的 Symbol 類型。凡是屬性名屬於 Symbol 類型，就都是獨一無二的，可以保證不會與其他屬性名產生衝突。

	let s = Symbol();
	
	typeof s
	// "symbol"

上面代碼中，變量`s`就是一個獨一無二的值。`typeof`運算符的結果，表明變量`s`是 Symbol 數據類型，而不是字符串之類的其他類型。

注意，Symbol 函數前不能使用`new`命令，否則會報錯。這是因為生成的 Symbol 是一個原始類型的值，不是對象。也就是說，由於 Symbol 值不是對象，所以不能添加屬性。基本上，它是一種類似於字符串的數據類型。

Symbol 函數可以接受一個字符串作為參數，表示對 Symbol 實例的描述，主要是為了在控制台顯示，或者轉為字符串時，比較容易區分。

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

但是，Symbol 值可以顯式轉為字符串。

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


## 實例：消除魔術字符串


## 屬性名的遍歷


## `Symbol.for()`，`Symbol.keyFor()`


## 內置的 Symbol 值


