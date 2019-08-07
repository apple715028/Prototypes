
#  原型（Prototype）


注意：所有模擬類拷貝行為的企圖，也就是我們在前面第四章描述的內容，稱為各種種類的“mixin”，和我們要在本章中講解的[[Prototype]]鏈機製完全不同。

1. **原型是什麼？**
 一種讓別的物件繼承其中的屬性的物件。
2. **任何物件都有其原型？**
是的。唯一的例外是Object.prototype(Object的原型)沒有原型，它是所有物件的最上層的源頭。
3. **任何物件可以拿來作為原型？**
是的


---

**[[Prototype]]**
* JavaScript中的對像有一個內部屬性，在語言規範中稱為[[Prototype]]，它只是一個其他對象的引用。幾乎所有的對像在被創建時，它的這個屬性都被賦予了一個非null值。

* 注意：我們馬上就會看到，一個對象擁有一個空的[[Prototype]]鏈接是可能的，雖然這有些不尋常。

---


```js
var myObject = {
	a :  2
};

myObject . a ; // 2

```
* [[Prototype]]引用有什麼用？在第三章中，我們講解了[[Get]]操作，它會在你引用一個對像上的屬性時被調用，比如myObject.a。對於默認的[[Get]]操作來說，第一步就是檢查對象本身是否擁有一個a屬性，如果有，就使用它。


---

```js
var anotherObject = { 
	a :  2 
}; //創建一個鏈接到`anotherObject`的對象
var myObject = Object . create ( anotherObject ); myObject . a ; // 2
```
* 假設，創建了一個對象，這個對象帶有一個鏈到指定對象的[[Prototype]]鏈接，那麼，我們現在讓myObject [[Prototype]]鏈到了anotherObject。雖然很明顯myObject.a實際上不存在，但是無論如何屬性訪問成功了（在anotherObject中找到了），而且確實找到了值2。
* 如果在anotherObject上也沒有找到a，而且[[Prototype]]鏈不為空，就沿著它繼續查找。
* 如果在鏈條的末尾都沒有找到匹配的屬性，那麼[[Get]]操作的返回結果為undefined。

---

```js
//創建一個鏈接到`anotherObject`的對象
var anotherObject = { 
	a :  2 
}; 
var myObject = Object . create ( anotherObject ); 
for ( var k in myObject) {
	 console . log ( " found: " + k); 
} //找到: a 
( " a " in myObject); // true
```
* 和這種[[Prototype]]鏈查詢處理相似，如果你使用for..in循環迭代一個對象，所有在它的鏈條上可以到達的（並且是enumerable見第三章）屬性都會被枚舉。如果你使用in操作符來測試一個屬性在一個對像上的存在性，in將會檢查對象的整個鏈條（不管可枚舉性）。
* 所以，當你以各種方式進行屬性查詢時，[[Prototype]]鏈就會一個鏈接一個鏈接地被查詢。一旦找到屬性或者鏈條終結，這種查詢就會停止。

---

# （原型）繼承
 **繼承是什麼？**

我們回歸本質上來思考 **繼承的目的是什麼**，在程式開發時的目的通常是為了擴充原有的物件定義不足之處。

---

**基本上可以依照不同程式語言的物件導向特性區分為:**

1. 類別的繼承(Classical inheritance)
2. 原型為基礎的繼承(Prototype-based inheritance)

---

### 原型繼承
“繼承”意味著拷貝操作，而JavaScript不拷貝對象屬性（原生上，默認地）。相反，JS在兩個對象間建立鏈接，一個對象實質上可以將對屬性/函數的訪問委託到另一個對像上。對於描述JavaScript對象鏈接機制來說，“委託”是一個準確得多的術語。


---

```js
function  Foo ( name ) {
	 this . name  = name; 
} Foo . prototype . myName = function () {
	 return this . name ; 
}; function Bar ( name , label ) {
	 Foo . call ( this , name );
	 this . label = label; 
} //這裡，我們創建一個新的`Bar.prototype`鏈接鏈到`Foo.prototype` Bar .. myLabel (); // "obj a"
   

  


prototype  =  Object . create ( Foo . prototype );
//注意！現在`Bar.prototype.constructor`不存在了，
//如果你有依賴這個屬性的習慣的話，它可以被手動“修復”。
Bar . prototype . myLabel = function () {
	 return this . label ; 
}; 
var a = new Bar ( " a " , " obj a " );
a . myName (); // "a" a
```
(要想知道為什麼上面代碼中的this指向a，參見第二章。)
重要的部分是Bar.prototype = Object.create( Foo.prototype )。
Object.create(..)憑空創建了一個“新”對象，並將這個新對象內部的[[Prototype]]鏈接到你指定的對像上（在這裡是Foo.prototype）。

換句話說，這一行的意思是：“做一個新的鏈接到'Foo點兒prototype'的'Bar點兒prototype '對象”。

當function Bar() { .. }被聲明時，就像其他函數一樣，擁有一個鏈到默認對象的.prototype鏈接。但是那個對像沒有鏈到我們希望的Foo.prototype。所以，我們創建了一個新對象，鏈到我們希望的地方，並將原來的錯誤鏈接的對象扔掉。


---


**[[prototype]] vs proto vs prototype**
:  Object 之間可以互相成為各自的 Prototype，被繼承的 Object 將會繼承父 Object 的 Prototype 所有屬性。[[Prototype]]，是一個設定（寫入） Object 的 Prototype 的接口，是一個內部屬性(internal property)，它並不允許外部存取。


---

![](https://i.imgur.com/QnjnsTR.png =300x)

---

##  __ proto __ 

(發音 dunder prototype)
*  最先被 Firefox使用，後來在 ES6 被列為Javascript 的標準內建屬性的。
它的出現是為了解決讀寫 Object.prototype的麻煩，提供一個快捷讀寫 Object.prototype 而設的一個 API，而且它是透過連結內部屬性 [[Prototype]]完成這個功能


---


說明 b.__proto__和Foo.prototype 的關系：

```js
function Foo(name) {
  this.name = name;
}
var b = new Foo('b');
var a = new Foo('a');
b.say = function() {
  console.log('Hi from ' + this.whoAmI());
}
console.log(a.__proto__ === Foo.prototype); // true
console.log(a.__proto__ === b.__proto__); // true
```


---

![](https://i.imgur.com/jKhZeBA.png =600x)
* 當say() 加入到物件 b，只要把 say() 加到 Foo.prototype 物件，b就會從它身上繼承過來。
* 如上圖，b.__proto__ 開放接口[[Prototype]] 指向Foo.prototype 物件，把自身的 prototype 從 Foo 物件繼承過來。


---

# 

 __ proto __ 像動物體內的基因，存在於每一個物件，影響物件的行為和屬性：顏色，足數目，叫聲，是否飛行…等。prototype 像母雞身上的基因，通過 prototype-base-orient 把自身的 prototype 遺傳到下一代小雞身上的基因 __ proto __。

![](https://i.imgur.com/vk64Pi3.jpg =500x)

---

### 類別的繼承

類別的繼承方式，在JavaScript可以用Object.create方法模擬出來



---

```JS
//superclass
function Player(name) {
    this.name = name
}

// 1. 呼叫上層的建構式
function VipPlayer(name, level) {
    Player.call(this, name)
    this.level = level
}

// 2. 使用Object.create建立prototype物件。
//    VipPlayer建構式的prototype.constructor為自己。
VipPlayer.prototype = Object.create(Player.prototype)
VipPlayer.prototype.constructor = VipPlayer


var inori = new VipPlayer('inori', 5)

console.log(inori instanceof Player) //true
console.log(inori instanceof VipPlayer) //true
//instanceof 檢查inori值是否為 class 的實例物件或建構函式。
```

---

**相當於ES6中的類別定義方法，使用extends關鍵字來作類別的繼承:**
```JS
class Player{
  constructor (name){
    this.name = name
  }
}

class VipPlayer extends Player {
  constructor (name, level){
    super(name)
    this.level = level
  }
}

```


---

很多真實的應用情況下，"以合成(或擴充)代替繼承(composition over inheritance)"才是正解，在JavaScript中合成比繼承容易得多了，彈性高應用也很廣，也比較符合語言本身的特性。思考的重點不同，才能撰寫出符合應用情況的程式碼。

---

# 建構式(Constructor) 

```js
function  Foo () {
	 // ... 
} Foo . prototype . constructor === Foo; // true var a = new Foo ();
 a . constructor === Foo; // true

```
Foo.prototype對象默認地 ，得到一個公有的，稱為.constructor的不可枚舉（見第三章）屬性，而且這個屬性回頭指向這個對象關聯的函數（這裡是Foo）。另外，我們看到被“構造器”調用new Foo()創建的對象a 看起來也擁有一個稱為.constructor的屬性，也相似地指向“創建它的函數”。

注意：這實際上不是真的。a上沒有.constructor屬性，而a.constructor確實解析成了Foo函數，“constructor”並不像它看起來的那樣實際意味著“被XX創建”。我們很快就會解釋這個奇怪的地方。

---

**構造器還是調用？**

上面的代碼的段中，我們試圖認為Foo是一個“構造器”，是因為我們用new調用它，而且我們觀察到它“構建”了一個對象。

在現實中，Foo不會比你的程序中的其他任何函數“更像構造器”。函數自身不是構造器。但是，當你在普通函數調用前面放一個new關鍵字時，這就將函數調用變成了“構造器調用”。事實上，new在某種意義上劫持了普通函數並將它以另一種方式調用：構建一個對象，外加這個函數要做的其他任何事。

```js
function  NothingSpecial () {
	 console . log ( " Don't mind me! " ); 
} var a = new NothingSpecial ();
 // "Don't mind me!" 
a; // {}
```
NothingSpecial只是一個普通的函數，但當用new調用時，幾乎是一種副作用，它會構建一個對象，並被我們賦值到a。這個調用是一個構造器調用，但是NothingSpecial本身並不是一個構造器。

換句話說，在JavaScrip t中，更合適的說法是，“構造器”是在前面用new關鍵字調用的任何函數。

函數不是構造器，但是當且僅當new被使用時，函數調用是一個“構造器調用”。


---

## 對象鏈接
主要在對一個對象進行屬性/方法引用，但這樣的屬性/方法不存在時實施。在這種情況下，[[Prototype]]鏈接告訴引擎在那個被鏈接的對像上查找這個屬性/方法。接下來，如果這個對像不能滿足查詢，它的[[Prototype]]又會被查找，如此繼續。這個在對象間的一系列鏈接構成了所謂的“原形鏈”。

**創建鏈接**

```js
var foo = {
	 something :  function () {
		 console . log ( " Tell me something good... " ); 
	} 
}; 
var bar = Object . create ( foo ); 
bar . something ();
// Tell me something good ...
```
Object.create(..)創建了一個鏈接到我們指定的對象（foo）上的新對象（bar），這給了我們[[Prototype]]機制的所有力量（委託），而且沒有new函數作為類和構造器調用產生的所有沒必要的複雜性，搞亂.prototype和.constructor引用，或任何其他的多餘的東西。

注意： Object.create(null)創建一個擁有空（也就是null）[[Prototype]]鏈接的對象，如此這個對像不能委託到任何地方。
因為這樣的對像沒有原形鏈，instancof操作符（前面解釋過）沒有東西可檢查，所以它總返回false。
由於他們典型的用途是在屬性中存儲數據，這種特殊的空[[Prototype]]對象經常被稱為“字典（dictionaries）”，這主要是因為它們不可能受到在[[Prototype]]鏈上任何委託屬性/函數的影響，所以它們是純粹的扁平數據存儲。

**鏈接作為候補？**

這些對象間的鏈接主要是為了給“缺失”的屬性和方法提供某種候補。
雖然這是一個可觀察到的結果，但是我不認為這是考慮[[Prototype]]的正確方法。

```js
var anotherObject = {
	 cool :  function () {
		 console . log ( " cool! " ); 
	} 
}; 
var myObject = Object . create ( anotherObject );
myObject . cool (); // "cool!"
```
這段代碼可以工作，但如果你這樣寫是為了萬一 myObject不能處理某些開發者可能會調用的屬性/方法，而讓anotherObject作為一個候補，你的軟件大概會變得有點兒“魔性”並且更難於理解和維護。

這不是說候補在任何情況下都不是一個合適的設計模式，但它不是一個在JS 中很常見的用法，所以如果你發現自己在這麼做，那麼你可能想要退一步並重新考慮它是否真的是合適且合理的設計。

注意：在ES6中，引入了一個稱為Proxy（代理）的高級功能，它可以提供某種“方法未找到”類型的行為。Proxy超出了本書的範圍，但會在以後的“你不懂JS”系列書目中詳細講解

**這裡不要錯過一個重要的細節。**

例如，你打算為一個開發者設計軟件，如果即使在myObject上沒有cool()方法時調用myObject.cool()也能工作，會在你的API設計上引入一些“魔法”氣息，這可能會使未來維護你的軟件的開發者很吃驚。

然而你可以在你的API設計上少用些“魔法”，而仍然利用[[Prototype]]鏈接的力量。

```js
var anotherObject = {
	 cool :  function () {
		 console . log ( " cool! " ); 
	} 
};
var myObject = Object . create ( anotherObject ); 
myObject . doCool = function () {
	 this . cool (); // internal delegation! 
};
myObject . doCool (); // "cool!"
```

這裡，我們調用myObject.doCool()，它是一個實際存在於 myObject上的方法，這使我們的API設計更清晰（沒那麼“魔性”）。在它內部，我們的實現依照委託設計模式（見第六章），利用[[Prototype]]委託到anotherObject.cool()。

換句話說，如果委託是一個內部實現細節，而非在你的API結構設計中簡單地暴露出來，那麼它將傾向於減少意外/困惑。我們會在下一章中詳細解釋委託。

---

## 複習

1. 當試圖在一個對像上進行屬性訪問，而對象又沒有該屬性時，對象內部的[[Prototype]]鏈接定義了[[Get]]操作（見第三章）下一步應當到哪裡尋找它。這種對像到對象的串行鏈接定義了對象的“原形鏈”（和嵌套的作用域鏈有些相似），在解析屬性時發揮作用。

2. 所有普通的對像用內建的Object.prototype作為原形鏈的頂端（就像作用域查詢的頂端是全局作用域），如果屬性沒能在鏈條的前面任何地方找到，屬性解析就會在這裡停止。toString()，valueOf()，和其他幾種共同工具都存在於這個Object.prototype對像上，這解釋了語言中所有的對像是如何能夠訪問他們的。

3. 使兩個對象相互鏈接在一起的最常見的方法是將new關鍵字與函數調用一起使用，在它的四個步驟中（見第二章），就會建立一個新對象鏈接到另一個對象。

4. 那個用new調用的函數有一個被隨便地命名為.prototype的屬性，這個屬性所引用的對象恰好就是這個新對象鏈接到的“另一個對象”。帶有new的函數調用通常被稱為“構造器”，儘管實際上它們並沒有像傳統的面向類語言那樣初始化一個類。

5. 雖然這些JavaScript機制看起來和傳統面向類語言的“初始化類”和“類繼承”類似，而在JavaScript中的關鍵區別是，沒有拷貝發生。取而代之的是對象最終通過[[Prototype]]鍊鍊接在一起。

6. 由於各種原因，不光是前面提到的術語，“繼承”（和“原型繼承”）與所有其他的OO 用語，在考慮JavaScript 實際如何工作時都沒有道理。相反，“委託”是一個更確切的術語，因為這些關係不是拷貝而是委託鏈接。

