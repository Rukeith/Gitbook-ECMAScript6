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

### (1) `get(target, propKey, receiver)`
### (2) `set(target, propKey, value, receiver)`
### (3) `has(target, propKey)`
### (4) `deleteProperty(target, propKey)`

## Proxy 實例的方法
## `Proxy.revocable()`
## Reflect 概述
## Reflect 對象的方法