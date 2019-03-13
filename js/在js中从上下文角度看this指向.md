## 在js中从上下文角度看this指向

话不多说先放码
```js
var fobj = {
	name: "G",
	showcc: function() {
		console.log(this.name)
	},
	obj: {
		id: 39,
		c: this,				
		showcc: () => {
			console.log(this.name)
			console.log(this)
		},
		showiderr: () => {
			console.log(this.id)
		},
		showid: function() {
			console.log(this.c)				
			console.log(this.id)
		}
	},
	
	fn: function() {
		return () => console.log(this.name)
	}
}
fobj.showcc()
fobj.obj.showcc()
fobj.obj.showiderr()
fobj.obj.showid()			
fobj.fn()()			
var f = fobj.obj.showid
f()			
```
请先写出各个函数调用之后的console输出

嗯。。。最后我会贴出执行结果和相应解释注释

先了解this是什么，引用点东西
> 1.this是关键字，不是变量，不是属性名，js语法不允许给this赋值。  
2.关键字this没有作用域限制，嵌套的函数不会从调用它的函数中继承this。  
3.如果嵌套函数作为方法调用，其this指向调用它的对象。  
4.如果嵌套函数作为函数调用，其this值是window（非严格模式），或undefined（严格模式下）。  

突然不想单讲举例了，就索性直接在贴代码吧  
this是调用时才能被确定的，我写在fobj内的注释都是和之后调用对应的，再改变一些情况后this也会发生变化  
这种变化我在这里举出例子了，就是最后两句调用
```javascript
// 理解作用域链 执行上下文 js执行过程 this的指向
// this的指向与上下文的关系
var fobj = {
	name: "G",
	showcc: function() {
		console.log(this.name)
	},
	// 上下文 分全局执行上下文和函数执行上下文
	// this:
	// 全局执行上下文的时候this是window
	// 函数执行上下文的时候 this情况分两种
	// 函数独立运行的时候 this还是window
	// 其他情况看被谁调用，谁调用它this就是谁
	// 具体网络上有很多分类，其实有一定共同点，看多了就能分辨
	obj: {
		id: 39,
		// 这个this是window
		// 这里是全局执行上下文
		c: this,				
		showcc: () => {
			// 箭头函数应该与父级上下文相同 其父级是obj
			// 所以游览器中这里的this指向的是window； 在nodejs环境中输出{}，空对象
			console.log(this.name)
			console.log(this)
		},
		showiderr: () => {
			// undefined 这里的this同上所述
			console.log(this.id)
		},
		showid: function() {
			console.log(this.c)				//会看到输出window or nodejs环境下的{}
			// 在fobj.obj.showid()调用下 它的this是fobj.obj 
			// 但是按照下面最后调用的情况下this应该是window or node下的{} 
			// 为什么会这样，请理解obj上面的注释内容
			console.log(this.id)
		}
	},
	
	fn: function() {
		// 这里如果不用箭头函数那就是闭包了
		// 箭头函数与父级同上下文 
		// 所以this指向与fn（）中this指向相同，为fobj
		return () => console.log(this.name)
	}
}

fobj.showcc()				// output: G
fobj.obj.showcc()			// output: undefined or 空的(window有name这个属性但是是空的)  and   window or {}
fobj.obj.showiderr()		// output: undefined  
fobj.obj.showid()			// 这里的showid是被fobj.obj调用的  output: window or {}  and  39
fobj.fn()()					// output: G
var f = fobj.obj.showid		
f()			// 这里的f是独立运行的  output: undefined  and  undefined
			// 原因: {} or window 没有 c和id这两个属性
```

chrome游览器下运行结果如下: 
![chrome下运行](.\images\1903\chrometest.png)

nodejs环境下运行结果如下:
![nodejs下运行](.\images\1903\nodetest.png)


> 之后记录下我的心路历程。主要在看vue官方文档的时候，看见箭头函数不能再vue的method中使用（其实现在知道了，其实是上下文同父级）。 之后就查到this这么多神奇的变化
> 之后一路查到作用域链、上下文以及js如何运行。其实原本我想放弃了，但是，看到我代码里有对象的对象的属性用了this，输出竟然是window的时候我懵了。现在明白了，这些对象
> 不过都在全局执行上下文中，自然this指向window