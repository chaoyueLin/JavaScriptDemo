# JavaScript
[H5](./h5.md)

## 原型
JavaScript 是动态的，并且本身不提供一个 class 实现。（在 ES2015/ES6 中引入了 class 关键字，但那只是语法糖，JavaScript 仍然是基于原型的）。

当谈到继承时，JavaScript 只有一种结构：对象。每个实例对象（ object ）都有一个私有属性（称之为 __proto__ ）指向它的构造函数的原型对象（prototype ）。该原型对象也有一个自己的原型对象( __proto__ ) ，层层向上直到一个对象的原型对象为 null。根据定义，null 没有原型，并作为这个原型链中的最后一个环节。

几乎所有 JavaScript 中的对象都是位于原型链顶端的 Object 的实例。

尽管这种原型继承通常被认为是 JavaScript 的弱点之一，但是原型继承模型本身实际上比经典模型更强大。例如，在原型模型的基础上构建经典模型相当简单。

## 基于原型链的继承
### 继承属性
JavaScript 对象是动态的属性“包”（指其自己的属性）。JavaScript 对象有一个指向一个原型对象的链。当试图访问一个对象的属性时，它不仅仅在该对象上搜寻，还会搜寻该对象的原型，以及该对象的原型的原型，依次层层向上搜索，直到找到一个名字匹配的属性或到达原型链的末尾。

	// 让我们从一个函数里创建一个对象o，它自身拥有属性a和b的：
	let f = function () {
	   this.a = 1;
	   this.b = 2;
	}
	/* 这么写也一样
	function f() {
	  this.a = 1;
	  this.b = 2;
	}
	*/
	let o = new f(); // {a: 1, b: 2}
	
	// 在f函数的原型上定义属性
	f.prototype.b = 3;
	f.prototype.c = 4;
	
	// 不要在 f 函数的原型上直接定义 f.prototype = {b:3,c:4};这样会直接打破原型链
	// o.[[Prototype]] 有属性 b 和 c
	//  (其实就是 o.__proto__ 或者 o.constructor.prototype)
	// o.[[Prototype]].[[Prototype]] 是 Object.prototype.
	// 最后o.[[Prototype]].[[Prototype]].[[Prototype]]是null
	// 这就是原型链的末尾，即 null，
	// 根据定义，null 就是没有 [[Prototype]]。
	
	// 综上，整个原型链如下: 
	
	// {a:1, b:2} ---> {b:3, c:4} ---> Object.prototype---> null
	
	console.log(o.a); // 1
	// a是o的自身属性吗？是的，该属性的值为 1
	
	console.log(o.b); // 2
	// b是o的自身属性吗？是的，该属性的值为 2
	// 原型上也有一个'b'属性，但是它不会被访问到。
	// 这种情况被称为"属性遮蔽 (property shadowing)"
	
	console.log(o.c); // 4
	// c是o的自身属性吗？不是，那看看它的原型上有没有
	// c是o.[[Prototype]]的属性吗？是的，该属性的值为 4
	
	console.log(o.d); // undefined
	// d 是 o 的自身属性吗？不是，那看看它的原型上有没有
	// d 是 o.[[Prototype]] 的属性吗？不是，那看看它的原型上有没有
	// o.[[Prototype]].[[Prototype]] 为 null，停止搜索
	// 找不到 d 属性，返回 undefined
### 继承方法
JavaScript 并没有其他基于类的语言所定义的“方法”。在 JavaScript 里，任何函数都可以添加到对象上作为对象的属性。函数的继承与其他的属性继承没有差别，包括上面的“属性遮蔽”（这种情况相当于其他语言的方法重写）。

当继承的函数被调用时，this 指向的是当前继承的对象，而不是继承的函数所在的原型对象。

	var o = {
	  a: 2,
	  m: function(){
	    return this.a + 1;
	  }
	};
	
	console.log(o.m()); // 3
	// 当调用 o.m 时，'this' 指向了 o.
	
	var p = Object.create(o);
	// p是一个继承自 o 的对象
	
	p.a = 4; // 创建 p 的自身属性 'a'
	console.log(p.m()); // 5
	// 调用 p.m 时，'this' 指向了 p
	// 又因为 p 继承了 o 的 m 函数
	// 所以，此时的 'this.a' 即 p.a，就是 p 的自身属性 'a' 

## 面向对象编程
### 命名空间
命名空间是一个容器，它允许开发人员在一个独特的，特定于应用程序的名称下捆绑所有的功能。 在JavaScript中，命名空间只是另一个包含方法，属性，对象的对象。

创造的JavaScript命名空间背后的想法很简单：一个全局对象被创建，所有的变量，方法和功能成为该对象的属性。使用命名空间也最大程度地减少应用程序的名称冲突的可能性。

	// 全局命名空间
	var MYAPP = MYAPP || {};
	// 子命名空间
	MYAPP.event = {};
	
	// 给普通方法和属性创建一个叫做MYAPP.commonMethod的容器
	MYAPP.commonMethod = {
	  regExForName: "", // 定义名字的正则验证
	  regExForPhone: "", // 定义电话的正则验证
	  validateName: function(name){
	    // 对名字name做些操作，你可以通过使用“this.regExForname”
	    // 访问regExForName变量
	  },
	 
	  validatePhoneNo: function(phoneNo){
	    // 对电话号码做操作
	  }
	}
	
	// 对象和方法一起申明
	MYAPP.event = {
	    addListener: function(el, type, fn) {
	    //  代码
	    },
	   removeListener: function(el, type, fn) {
	    // 代码
	   },
	   getEvent: function(e) {
	   // 代码
	   }
	  
	   // 还可以添加其他的属性和方法
	}
	
	//使用addListener方法的写法:
	MYAPP.event.addListener("yourel", "type", callback);

### 自定义对象

	function Person() { } 
	// 或
	var Person = function(){ }
	function Person() { }
	var person1 = new Person();
	var person2 = new Person();

#### 属性 (对象属性)
可以使用 关键字 this调用类中的属性, this是对当前对象的引用。 从外部存取(读/写)其属性的语法是: InstanceName.Property; 这与C++，Java或者许多其他语言中的语法是一样的 (在类中语法 this.Property 常用于set和get属性值)

	function Person(firstName) {
	  this.firstName = firstName;
	  alert('Person instantiated');
	}
	
	var person1 = new Person('Alice');
	var person2 = new Person('Bob');
	
	// Show the firstName properties of the objects
	alert('person1 is ' + person1.firstName); // alerts "person1 is Alice"
	alert('person2 is ' + person2.firstName); // alerts "person2 is Bob"

#### 方法（对象属性）

方法与属性很相似， 不同的是：一个是函数，另一个可以被定义为函数。 调用方法很像存取一个属性,  不同的是add () 在方法名后面很可能带着参数. 为定义一个方法, 需要将一个函数赋值给类的 prototype 属性; 这个赋值给函数的名称就是用来给对象在外部调用它使用的。

	function Person(firstName) {
	  this.firstName = firstName;
	}
	
	Person.prototype.sayHello = function() {
	  alert("Hello, I'm " + this.firstName);
	};
	
	var person1 = new Person("Alice");
	var person2 = new Person("Bob");
	
	// call the Person sayHello method.
	person1.sayHello(); // alerts "Hello, I'm Alice"
	person2.sayHello(); // alerts "Hello, I'm Bob"

#### 继承

创建一个或多个类的专门版本类方式称为继承（Javascript只支持单继承）。 创建的专门版本的类通常叫做子类，另外的类通常叫做父类。 在Javascript中，继承通过赋予子类一个父类的实例并专门化子类来实现。在现代浏览器中你可以使用 Object.create 实现继承.

	// 定义Person构造器
	function Person(firstName) {
	  this.firstName = firstName;
	}
	
	// 在Person.prototype中加入方法
	Person.prototype.walk = function(){
	  alert("I am walking!");
	};
	Person.prototype.sayHello = function(){
	  alert("Hello, I'm " + this.firstName);
	};
	
	// 定义Student构造器
	function Student(firstName, subject) {
	  // 调用父类构造器, 确保(使用Function#call)"this" 在调用过程中设置正确
	  Person.call(this, firstName);
	
	  // 初始化Student类特有属性
	  this.subject = subject;
	};
	
	// 建立一个由Person.prototype继承而来的Student.prototype对象.
	// 注意: 常见的错误是使用 "new Person()"来建立Student.prototype.
	// 这样做的错误之处有很多, 最重要的一点是我们在实例化时
	// 不能赋予Person类任何的FirstName参数
	// 调用Person的正确位置如下，我们从Student中来调用它
	Student.prototype = Object.create(Person.prototype); // See note below
	
	// 设置"constructor" 属性指向Student
	Student.prototype.constructor = Student;
	
	// 更换"sayHello" 方法
	Student.prototype.sayHello = function(){
	  console.log("Hello, I'm " + this.firstName + ". I'm studying " + this.subject + ".");
	};
	
	// 加入"sayGoodBye" 方法
	Student.prototype.sayGoodBye = function(){
	  console.log("Goodbye!");
	};
	
	// 测试实例:
	var student1 = new Student("Janet", "Applied Physics");
	student1.sayHello();   // "Hello, I'm Janet. I'm studying Applied Physics."
	student1.walk();       // "I am walking!"
	student1.sayGoodBye(); // "Goodbye!"
	
	// Check that instanceof works correctly
	console.log(student1 instanceof Person);  // true 
	console.log(student1 instanceof Student); // true