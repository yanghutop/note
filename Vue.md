# 一、Vue基础



## 模板语法

Vue模板语法有2大类：

​                    1.插值语法：

​                            功能：用于解析标签体内容。

​                            写法：{{xxx}}，xxx是js表达式，且可以直接读取到data中的所有属性。

​                    2.指令语法：

​                            功能：用于解析标签（包括：标签属性、标签体内容、绑定事件.....）。

​                            举例：v-bind:href="xxx" 或  简写为 :href="xxx"，xxx同样要写js表达式，

​										且可以直接读取到data中的所有属性。

![1655861659919](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1655861659919.png)



## 数据绑定

Vue中有2种数据绑定的方式：

​                    1.单向绑定(v-bind)：数据只能从data流向页面。

​                    2.双向绑定(v-model)：数据不仅能从data流向页面，还可以从页面流向data。

​                        备注：

​                                1.双向绑定一般都应用在表单类元素上（如：input、select等）

​                                2.v-model:value 可以简写为 v-model，因为v-model默认收集的就是value值。

![1655861830855](C:\Users\86198\AppData\Roaming\Typora\typora-user-images\1655861830855.png)



## data与el的2种写法

​                    1.el有2种写法

​                                    (1).new Vue时候配置el属性。

​                                    (2).先创建Vue实例，随后再通过vm.$mount('#root')指定el的值。

​                    2.data有2种写法

​                                    (1).对象式

​                                    (2).函数式

​                                    如何选择：目前哪种写法都可以，以后学习到组件时，data必须使用函数式，否则会报错。

​                    3.一个重要的原则：

​                                    由Vue管理的函数，一定不要写箭头函数，一旦写了箭头函数，this就不再是Vue实例了。

```javascript
//el的两种写法
	    const v = new Vue({
			//el:'#root', //第一种写法
			data:{
				name:'尚硅谷'
			}
		})
		console.log(v)

		v.$mount('#root') //第二种写法

//data的两种写法
		new Vue({
			el:'#root',
			//data的第一种写法：对象式
			 data:{
				name:'尚硅谷'
			} 

			//data的第二种写法：函数式
			data(){
				console.log('@@@',this) //此处的this是Vue实例对象
				return{
					name:'尚硅谷'
				}
			}
		})
```



## 事件处理

### 事件的基本使用

​                            1.使用v-on:xxx 或 @xxx 绑定事件，其中xxx是事件名；

​                            2.事件的回调需要配置在methods对象中，最终会在vm上；

​                            3.methods中配置的函数，不要用箭头函数！否则this就不是vm了；

​                            4.methods中配置的函数，都是被Vue所管理的函数，this的指向是vm 或 组件实例对象；

​                            5.@click="demo" 和 @click="demo($event)" 效果一致，但后者可以传参；

```javascript
<button @click="showInfo1">点我提示信息1（不传参）</button>
<button @click="showInfo2($event,66)">点我提示信息2（传参）</button>

methods:{
				showInfo1(event){
					// console.log(event.target.innerText)
					// console.log(this) //此处的this是vm
					alert('同学你好！')
				},
				showInfo2(event,number){
					console.log(event,number)
					// console.log(event.target.innerText)
					// console.log(this) //此处的this是vm
					alert('同学你好！！')
				}
			}
```



### Vue中的事件修饰符

​                        1.prevent：阻止默认事件（常用）；

​                        2.stop：阻止事件冒泡（常用）；

​                        3.once：事件只触发一次（常用）；

​                        4.capture：使用事件的捕获模式；

​                        5.self：只有event.target是当前操作的元素时才触发事件；

​                        6.passive：事件的默认行为立即执行，无需等待事件回调执行完毕；

```javascript
<div id="root">
			<h2>欢迎来到{{name}}学习</h2>
			<!-- 阻止默认事件（常用） -->
			<a href="http://www.atguigu.com" @click.prevent="showInfo">点我提示信息</a>

			<!-- 阻止事件冒泡（常用） -->
			<div class="demo1" @click="showInfo">
				<button @click.stop="showInfo">点我提示信息</button>
				<!-- 修饰符可以连续写 -->
				<!-- <a href="http://www.atguigu.com" @click.prevent.stop="showInfo">点我提示信息</a> -->
			</div>

			<!-- 事件只触发一次（常用） -->
			<button @click.once="showInfo">点我提示信息</button>

			<!-- 使用事件的捕获模式 -->
			<div class="box1" @click.capture="showMsg(1)">
				div1
				<div class="box2" @click="showMsg(2)">
					div2
				</div>
			</div>

			<!-- 只有event.target是当前操作的元素时才触发事件； -->
			<div class="demo1" @click.self="showInfo">
				<button @click="showInfo">点我提示信息</button>
			</div>

			<!-- 事件的默认行为立即执行，无需等待事件回调执行完毕； -->
			<ul @wheel.passive="demo" class="list">
				<li>1</li>
				<li>2</li>
				<li>3</li>
				<li>4</li>
			</ul>

</div>

methods:{
				showInfo(e){
					alert('同学你好！')
					// console.log(e.target)
				},
				showMsg(msg){
					console.log(msg)
				},
				demo(){
					for (let i = 0; i < 100000; i++) {
						console.log('#')
					}
					console.log('累坏了')
				}
			}
```



### 键盘事件

1.Vue中常用的按键别名：

​                            回车 => enter

​                            删除 => delete (捕获“删除”和“退格”键)

​                            退出 => esc

​                            空格 => space

​                            换行 => tab (特殊，必须配合keydown去使用)

​                            上 => up

​                            下 => down

​                            左 => left

​                            右 => right

​                2.Vue未提供别名的按键，可以使用按键原始的key值去绑定，但注意要转为kebab-case（短横线命名）

​                3.系统修饰键（用法特殊）：ctrl、alt、shift、meta

​                            (1).配合**keyup**使用：按下修饰键的同时，再按下其他键，随后释放其他键，事件才被触发。

​                            (2).配合**keydown**使用：正常触发事件。

​                4.也可以使用keyCode去指定具体的按键（不推荐）

​                5.Vue.config.keyCodes.自定义键名 = 键码，可以去定制按键别名

```javascript
<input type="text" placeholder="按下回车提示输入" @keydown.huiche="showInfo">
    
    showInfo(e){
    // console.log(e.key,e.keyCode)
    console.log(e.target.value)
}
```



## 计算属性





## 监视属性





## 样式绑定







## 条件渲染

​                            1.v-if

​                                        写法：

​                                                (1).v-if="表达式" 

​                                                (2).v-else-if="表达式"

​                                                (3).v-else="表达式"

​                                        适用于：切换频率较低的场景。

​                                        特点：不展示的DOM元素直接被移除。

​                                        注意：v-if可以和:v-else-if、v-else一起使用，但要求结构不能被“打断”。

​                            2.v-show

​                                        写法：v-show="表达式"

​                                        适用于：切换频率较高的场景。

​                                        特点：不展示的DOM元素未被移除，仅仅是使用样式隐藏掉

​                            3.备注：使用v-if的时，元素可能无法获取到，而使用v-show一定可以获取到。

```javascript
<div id="root">
			<h2>当前的n值是:{{n}}</h2>
			<button @click="n++">点我n+1</button>
			<!-- 使用v-show做条件渲染 -->
			<!-- <h2 v-show="false">欢迎来到{{name}}</h2> -->
			<!-- <h2 v-show="1 === 1">欢迎来到{{name}}</h2> -->

			<!-- 使用v-if做条件渲染 -->
			<!-- <h2 v-if="false">欢迎来到{{name}}</h2> -->
			<!-- <h2 v-if="1 === 1">欢迎来到{{name}}</h2> -->

			<!-- v-else和v-else-if -->
			<!-- <div v-if="n === 1">Angular</div>
			<div v-else-if="n === 2">React</div>
			<div v-else-if="n === 3">Vue</div>
			<div v-else>哈哈</div> -->

			<!-- v-if与template的配合使用 -->
			<template v-if="n === 1">
				<h2>你好</h2>
				<h2>尚硅谷</h2>
				<h2>北京</h2>
			</template>
</div>
```



## 列表渲染

v-for指令:

​                        1.用于展示列表数据

​                        2.语法：v-for="(item, index) in xxx" :key="yyy"

​                        3.可遍历：数组、对象、字符串（用的很少）、指定次数（用的很少）

```
<div id="root">
			<!-- 遍历数组 -->
			<h2>人员列表（遍历数组）</h2>
			<ul>
				<li v-for="(p,index) of persons" :key="index">
					{{p.name}}-{{p.age}}
				</li>
			</ul>

			<!-- 遍历对象 -->
			<h2>汽车信息（遍历对象）</h2>
			<ul>
				<li v-for="(value,k) of car" :key="k">
					{{k}}-{{value}}
				</li>
			</ul>

			<!-- 遍历字符串 -->
			<h2>测试遍历字符串（用得少）</h2>
			<ul>
				<li v-for="(char,index) of str" :key="index">
					{{char}}-{{index}}
				</li>
			</ul>
			
			<!-- 遍历指定次数 -->
			<h2>测试遍历指定次数（用得少）</h2>
			<ul>
				<li v-for="(number,index) of 5" :key="index">
					{{index}}-{{number}}
				</li>
			</ul>
</div>

new Vue({
				el:'#root',
				data:{
					persons:[
						{id:'001',name:'张三',age:18},
						{id:'002',name:'李四',age:19},
						{id:'003',name:'王五',age:20}
					],
					car:{
						name:'奥迪A8',
						price:'70万',
						color:'黑色'
					},
					str:'hello'
				}
			})
```

### key的原理

4. 开发中如何选择key?:

​                               1.最好使用每条数据的唯一标识作为key, 比如id、手机号、身份证号、学号等唯一值。

​                                2.如果不存在对数据的逆序添加、逆序删除等破坏顺序操作，仅用于渲染列表用于展示，

​                                 使用index作为key是没有问题的。

```
<ul>
				<li v-for="(p,index) of persons" :key="p.id">
					{{p.name}}-{{p.age}}
					<input type="text">
				</li>
</ul>
```



### 列表过滤

### 列表排序



## 收集表单数据

​                    若：<input type="text"/>，则v-model收集的是value值，用户输入的就是value值。

​                    若：<input type="radio"/>，则v-model收集的是value值，且要给标签配置value值。

​                    若：<input type="checkbox"/>

​                            1.没有配置input的value属性，那么收集的就是checked（勾选 or 未勾选，是布尔值）

​                            2.配置input的value属性:

​                                    (1)v-model的初始值是非数组，那么收集的就是checked（勾选 or 未勾选，是布尔值）

​                                    (2)v-model的初始值是数组，那么收集的的就是value组成的数组

​                    备注：v-model的三个修饰符：

​                                    lazy：失去焦点再收集数据

​                                    number：输入字符串转为有效的数字

​                                    trim：输入首尾空格过滤

```html
<div id="root">
			<form @submit.prevent="demo">
				账号：<input type="text" v-model.trim="userInfo.account"> <br/><br/>
				密码：<input type="password" v-model="userInfo.password"> <br/><br/>
				年龄：<input type="number" v-model.number="userInfo.age"> <br/><br/>
				性别：
				男<input type="radio" name="sex" v-model="userInfo.sex" value="male">
				女<input type="radio" name="sex" v-model="userInfo.sex" value="female"> <br/><br/>
				爱好：
				学习<input type="checkbox" v-model="userInfo.hobby" value="study">
				打游戏<input type="checkbox" v-model="userInfo.hobby" value="game">
				吃饭<input type="checkbox" v-model="userInfo.hobby" value="eat">
				<br/><br/>
				所属校区
				<select v-model="userInfo.city">
					<option value="">请选择校区</option>
					<option value="beijing">北京</option>
					<option value="shanghai">上海</option>
					<option value="shenzhen">深圳</option>
					<option value="wuhan">武汉</option>
				</select>
				<br/><br/>
				其他信息：
				<textarea v-model.lazy="userInfo.other"></textarea> <br/><br/>
				<input type="checkbox" v-model="userInfo.agree">阅读并接受<a href="http://www.atguigu.com">《用户协议》</a>
				<button>提交</button>
			</form>
		</div>

new Vue({
			el:'#root',
			data:{
				userInfo:{
					account:'',
					password:'',
					age:18,
					sex:'female',
					hobby:[],
					city:'beijing',
					other:'',
					agree:''
				}
			},
			methods: {
				demo(){
					console.log(JSON.stringify(this.userInfo))
				}
			}
		})
```





## 过滤器

​                定义：对要显示的数据进行特定格式化后再显示（适用于一些简单逻辑的处理）。

​                语法：

​                        1.注册过滤器：Vue.filter(name,callback) 或 new Vue{filters:{}}

​                        2.使用过滤器：{{ xxx | 过滤器名}}  或  v-bind:属性 = "xxx | 过滤器名"

​                备注：

​                        1.过滤器也可以接收额外参数、多个过滤器也可以串联

​                        2.并没有改变原本的数据, 是产生新的对应的数据



### 内置指令

​						v-bind    : 单向绑定解析表达式, 可简写为 :xxx

​                        v-model : 双向数据绑定

​                        v-for   : 遍历数组/对象/字符串

​                        v-on    : 绑定事件监听, 可简写为@

​                        v-if        : 条件渲染（动态控制节点是否存存在）

​                        v-else  : 条件渲染（动态控制节点是否存存在）

​                        v-show  : 条件渲染 (动态控制节点是否展示)

​               		 v-text指令：

​                        	1.作用：向其所在的节点中渲染文本内容。

​                        	2.与插值语法的区别：v-text会替换掉节点中的内容，{{xx}}则不会。

 v-html指令：

​                        1.作用：向指定节点中渲染包含html结构的内容。

​                        2.与插值语法的区别：

​                                    (1).v-html会替换掉节点中所有的内容，{{xx}}则不会。

​                                    (2).v-html可以识别html结构。

​                        3.严重注意：v-html有安全性问题！！！！

​                                    (1).在网站上动态渲染任意HTML是非常危险的，容易导致XSS攻击。

​                                    (2).一定要在可信的内容上使用v-html，永不要用在用户提交的内容上！



v-cloak指令（没有值）：

​                        1.本质是一个特殊属性，Vue实例创建完毕并接管容器后，会删掉v-cloak属性。

​                        2.使用css配合v-cloak可以解决网速慢时页面展示出{{xxx}}的问题。

v-once指令：

​                        1.v-once所在节点在初次动态渲染后，就视为静态内容了。

​                        2.以后数据的改变不会引起v-once所在结构的更新，可以用于优化性能。

v-pre指令：

​                    1.跳过其所在节点的编译过程。

​                    2.可利用它跳过：没有使用指令语法、没有使用插值语法的节点，会加快编译。















# 二、Vue组件化编程





# 三、使用脚手架



## ref属性

1. 被用来给元素或子组件注册引用信息（id的替代者）
2. 应用在html标签上获取的是真实DOM元素，应用在组件标签上是组件实例对象（vc）
3. 使用方式：
   1. 打标识：```<h1 ref="xxx">.....</h1>``` 或 ```<School ref="xxx"></School>```
   2. 获取：```this.$refs.xxx```

```
<div>
		<h1 v-text="msg" ref="title"></h1>
		<button ref="btn" @click="showDOM">点我输出上方的DOM元素</button>
		<School ref="sch"/>
</div>
methods: {
			showDOM(){
				console.log(this.$refs.title) //真实DOM元素
				console.log(this.$refs.btn) //真实DOM元素
				console.log(this.$refs.sch) //School组件的实例对象（vc）
			}
		},
```



## **props**

1. **作用：**用于父组件给子组件传递数据     **父组件 ===> 子组件**

2. **读取方式一: 只指定名称** 

   props: ['name', 'age', 'setName'] 

3. **读取方式二: 指定名称和类型** 

   props: {

   ​	name: String, 

   ​	age: Number, 

   ​	setNmae: Function 

   } 

4. **读取方式三: 指定名称/类型/必要性/默认值** 

   props: {

   ​	name: {type: String, required: true, default:xxx}, 

   } 

   备注：props是只读的，Vue底层会监测你对props的修改，如果进行了修改，就会发出警告，若业务需求确实需要修改，那么请复制props的内容到data中一份，然后去修改data中的数据。

```
	<Student name="李四" sex="女" :age="18"/> //app组件中
	
	//Student组件中
		//简单声明接收
		// props:['name','age','sex'] 

		//接收的同时对数据进行类型限制
		/* props:{
			name:String,
			age:Number,
			sex:String
		} */

		//接收的同时对数据：进行类型限制+默认值的指定+必要性的限制
		props:{
			name:{
				type:String, //name的类型是字符串
				required:true, //name是必要的
			},
			age:{
				type:Number,
				default:99 //默认值
			},
			sex:{
				type:String,
				required:true
			}
		}
```



## mixin混入



## 插件

1. 功能：用于增强Vue

2. 本质：包含install方法的一个对象，install的第一个参数是Vue，第二个以后的参数是插件使用者传递的数据。

3. 定义插件：

   ```js
   对象.install = function (Vue, options) {
       // 1. 添加全局过滤器
       Vue.filter(....)
   
       // 2. 添加全局指令
       Vue.directive(....)
   
       // 3. 配置全局混入(合)
       Vue.mixin(....)
   
       // 4. 添加实例方法
       Vue.prototype.$myMethod = function () {...}
       Vue.prototype.$myProperty = xxxx
   }
   ```

4. 使用插件：```Vue.use()```





## webStorage(浏览器本地存储)

1. 存储内容大小一般支持5MB左右（不同浏览器可能还不一样）

2. 浏览器端通过 Window.sessionStorage 和 Window.localStorage 属性来实现本地存储机制。

3. 相关API：

   1. ```xxxxxStorage.setItem('key', 'value');```
      				该方法接受一个键和值作为参数，会把键值对添加到存储中，如果键名存在，则更新其对应的值。

   2. ```xxxxxStorage.getItem('person');```

      ​		该方法接受一个键名作为参数，返回键名对应的值。

   3. ```xxxxxStorage.removeItem('key');```

      ​		该方法接受一个键名作为参数，并把该键名从存储中删除。

   4. ``` xxxxxStorage.clear()```

      ​		该方法会清空存储中的所有数据。

4. 备注：

   1. SessionStorage存储的内容会随着浏览器窗口关闭而消失。
   2. LocalStorage存储的内容，需要手动清除才会消失。
   3. ```xxxxxStorage.getItem(xxx)```如果xxx对应的value获取不到，那么getItem的返回值是null。
   4. ```JSON.parse(null)```的结果依然是null。



## 组件自定义事件

1. 一种组件间通信的方式，适用于：<strong style="color:red">子组件 ===> 父组件</strong>

2. 使用场景：A是父组件，B是子组件，B想给A传数据，那么就要在A中给B绑定自定义事件（<span style="color:red">事件的回调在A中</span>）。

3. 绑定自定义事件：

   1. 第一种方式，在父组件中：```<Demo @atguigu="test"/>```  或 ```<Demo v-on:atguigu="test"/>```

   2. 第二种方式，在父组件中：

      ```js
      <Demo ref="demo"/>
      ......
      mounted(){
         this.$refs.xxx.$on('atguigu',this.test)
      }
      ```

   3. 若想让自定义事件只能触发一次，可以使用```once```修饰符，或```$once```方法。

4. 触发自定义事件：```this.$emit('atguigu',数据)```		

5. 解绑自定义事件```this.$off('atguigu')```

6. 组件上也可以绑定原生DOM事件，需要使用```native```修饰符。

7. 注意：通过```this.$refs.xxx.$on('atguigu',回调)```绑定自定义事件时，回调<span style="color:red">要么配置在methods中</span>，<span style="color:red">要么用箭头函数</span>，否则this指向会出问题！





## 全局事件总线

1. 一种组件间通信的方式，适用于<span style="color:red">任意组件间通信</span>。

2. 安装全局事件总线：

   ```js
   new Vue({
   	......
   	beforeCreate() {
   		Vue.prototype.$bus = this //安装全局事件总线，$bus就是当前应用的vm
   	},
       ......
   }) 
   ```

3. 使用事件总线：

   1. 接收数据：A组件想接收数据，则在A组件中给$bus绑定自定义事件，事件的<span style="color:red">回调留在A组件自身。</span>

      ```js
      methods(){
        demo(data){......}
      }
      ......
      mounted() {
        this.$bus.$on('xxxx',this.demo)
      }
      ```

   2. 提供数据：```this.$bus.$emit('xxxx',数据)```

4. 最好在beforeDestroy钩子中，用$off去解绑<span style="color:red">当前组件所用到的</span>事件。



## 消息订阅与发布

1. 一种组件间通信的方式，适用于<span style="color:red">任意组件间通信</span>。

2. 使用步骤：

   1. 安装pubsub：```npm i pubsub-js```

   2. 引入: ```import pubsub from 'pubsub-js'```

   3. 接收数据：A组件想接收数据，则在A组件中订阅消息，订阅的<span style="color:red">回调留在A组件自身。</span>

      ```js
      methods(){
        demo(data){......}
      }
      ......
      mounted() {
        this.pid = pubsub.subscribe('xx消息名x',this.demo) //订阅消息
      }
      ```

   4. 提供数据：```pubsub.publish('xxxx消息名xx',数据)```

   5. 最好在beforeDestroy钩子中，用```PubSub.unsubscribe(pid)```去<span style="color:red">取消订阅。</span>



## nextTick（一般在编辑时用）

1. 语法：```this.$nextTick(回调函数)```
2. 作用：在下一次 DOM 更新结束后执行其指定的回调。
3. 什么时候用：当改变数据后，要基于更新后的新DOM进行某些操作时，要在nextTick所指定的回调函数中执行。





## 配置代理













# 四、Vue中的ajax



# 五、vuex状态管理



# 六、vue-router路由