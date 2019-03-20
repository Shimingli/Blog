---
title: Vue.js学习No.1
date: 2019-03-08 10:00:05
tags: Vue.js
---


* **欢迎关注我的公众号**
![公众号](https://upload-images.jianshu.io/upload_images/5363507-0a0cf2e5fd8f843d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


* Vue.js 是最火的一个前端框架，React最流行，都可以开发网站和手机App的开发，Vue需要借助Weex
* 前端三大主流的框架，Vue Angular React
* Vue是一套构建用户界面的框架，只关注视图层，非常易于上手和第三方库和其他项目整合
* 框架：是一套完整的解决的方案，对项目侵入性比较大，项目如果需要更换框架，则需要重构这个项目
* 库：是一个小功能，对项目的侵入性比较小，想换就换
* MVC 是后端的分层开发的概念，MVVM是前端的概念，主要是关注视图层的分离，MVVM把前端的视图层，分为了三个部分 Model View VM ViewModel 
<!--  more  -->

####　第一个Demo
  ```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>

    <!-- 导入Vue-->
    <script src="./lib/vue.min.js"></script>
</head>
<body>
    <!-- Vue控制的元素区域 vue -->
    <div id="app">
     <p>{{ msg }}</p>      
    </div>
    <!-- vue的实力 -->
    <script>
        // new Vue 就是我们MVVM的VM
     var vm=new Vue({
        el:'#app',
        // 这里就是MVVM 就是M 保存页面的数据的
        data :{ //
             msg:'我是shiming'
        }
     })

    </script>
</body>
</html>
```

####　第二个Demo
*  使用 v-cloak 能够解决 插值表达式闪烁的问题
*  v-text是默认没有闪烁的问题的 
*  v-text会覆盖元素中原本的内容，但是插值表达式只会替换占位符，不会把整个元素清空
*  v-html 可以渲染出内容中所带的标签的内容
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
     [v-cloak]{
         display: none;
     }
    </style>
</head>
<body>
    <div id="app">
       <!-- 使用 v-cloak 能够解决 插值表达式闪烁的问题 -->
     <p v-cloak >-------------{{ msg }}------</p>
      <!-- v-text 和v-cloak作用差不多 -->
     <h4 v-text="msg">========</h4>
     <!-- v-text是默认没有闪烁的问题的 -->
     <!-- v-text会覆盖元素中原本的内容，但是插值表达式只会替换占位符，不会把整个元素清空 -->
     

     <div>{{msg2}}</div>
   
     <div v-text="msg2"></div>
      <!-- 这个就可以渲染出来哦 -->
     <div v-html="msg2"></div>
    </div>
    <script src="./lib/vue.min.js"></script>
    <script>
    var vm=new Vue({
      el:'#app',
      data :{
          msg:"123" ,
          msg2:"<h1>我是仕明 我爱学习 </h1>"
      }
    })
    </script>
    
</body>
</html>

```

#### 第三个Demo

*  如何定义一个Vue的代码结构  如何定义插值表达式 和  v-text  v-cloak 
*  v-html 
*  v-bind  缩写 : 
*  v-on 时间绑定机制 缩写 @


````
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
     [v-cloak]{
         display: none;
     }
    </style>
</head>
<body>
    <div id="app">
       <!-- 使用 v-cloak 能够解决 插值表达式闪烁的问题 -->
     <p v-cloak >-------------{{ msg }}------</p>
      <!-- v-text 和v-cloak作用差不多 -->
     <h4 v-text="msg">========</h4>
     <!-- v-text是默认没有闪烁的问题的 -->
     <!-- v-text会覆盖元素中原本的内容，但是插值表达式只会替换占位符，不会把整个元素清空 -->
     

     <div>{{msg2}}</div>
   
     <div v-text="msg2"></div>
      <!-- 这个就可以渲染出来哦 -->
     <div v-html="msg2"></div>


     <!-- v-bind  是 Vue中，提供用于绑定属性的指令 -->
     <input type="button"  value="anniu"  v-bind:title="title +'我爱你'">

      <!-- v-bind 可以被简写为 ： v-bind中可以写合法的js
表达式 -->
     <input type="button"  value="anniu"  :title="title +'我爱你2222'"  v-on:click="show">
   
     <!-- 把鼠标放在上面就会出现一个弹窗 这个比较掉哦 -->
     <input type="button"  value="阿牛"  :title="title +'我爱你2222'"  v-on:mouseover="show">
   
     <input type="button"  value="阿牛"  :title="title +'我爱你2222'"  @click="show">
    </div>
    <script src="./lib/vue.min.js"></script>
    <script>
    var vm=new Vue({
      el:'#app',
      data :{
          msg:"123" ,
          msg2:"<h1>我是仕明 我爱学习 </h1>",
          title:"这是我的title",
      },
      methods:{
         show:function(){
             alert('把鼠标放在上面就会出现一个弹窗')
         }

      }
    })
    </script>
    
</body>
</html>

<!-- 如何定义一个Vue的代码结构  如何定义插值表达式 和  v-text  v-cloak -->
<!-- v-html -->
<!-- v-bind  缩写 : -->
<!-- v-on 时间绑定机制 缩写 @-->
````


####　第四个Demo

*  .prevent  阻止默认的行为 阻止标签到处跑

*  capture 先把事件传递给div 然后在给到btn 事件传递变了一个方向 

*  /stop事件不能給div傳遞事件  

*  once 事件只触发一次 
*  stop和self的区别 

*  self 只会阻止自己身上冒泡行为的触发，并不会真正阻止 冒泡行为 但是stop就可以完全的阻止 


```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>

    <script src="./lib/vue.min.js"></script>
    <style>
        .inner {
            height:30px;
            background-color: darkblue
        }
        .outer{
            padding:40px;
            background-color: red
        }
    </style>
</head>

<body>

    <div id="app">
        <div class="inner" @Click="divClick">
            <!-- /stop事件不能給div傳遞事件  -->
            <input type="button" value="點我" @click.stop="btnClick">
        </div>

        <!-- 网站一定要写全啊 必须写 http啊 沃日 -->
        <a href="http://www.baidu.com" @click.prevent="baidu">百度一下</a>

        <!-- capture 先把事件传递给div 然后在给到btn 事件传递变了一个方向 -->
        <div class="inner" @Click.capture="divClick">
            <!-- //事件不能給div傳遞事件  -->
            <input type="button" value="點我" @click.stop="btnClick">
        </div>

        <!-- 只有点div的时候 我才执行我的点击事件 -->
        <div class="inner" @Click.self="divClick">
            <input type="button" value="點我" @click="btnClick">
        </div>

     <!-- once 事件只触发一次 -->
        <a href="http://www.baidu.com" @click.prevent.once="baidu">百度一下</a>


        <div class="outer" @click="divClick">
                <div class="inner" @click="innerClick">
                        <input type="button" value="點我dddd" @click="btnClick">
            
                    </div>

        </div>

        <div class="outer" @click="divClick">
                <div class="inner" @click="innerClick">
                    <!-- 停止往下面传递 -->
                        <input type="button" value="點我dddd" @click.stop="btnClick">
            
                    </div>
        </div>
        <!-- self 只会阻止自己身上冒泡行为的触发，并不会真正阻止 冒泡行为 但是stop就可以完全的阻止 -->
        <div class="outer" @click="divClick">
                <div class="inner" @click.self="innerClick">
                    <!-- 停止往下面传递 -->
                        <input type="button" value="點我dddd" @click="btnClick">
            
                    </div>

        </div>
    </div>



    <script>
        //    vm实例中，如果想使用data的数据 或者是想调用methods 中的方法，必须通过
        // this.数据名称  this就是 vm
        var vm = new Vue({
            //  这里注意是冒号啊 沃日哦 
            el: "#app",
            data: {
                msg: "你好，仕明同学！！",
                intervalID: null
            },
            methods: {
                baidu() {
                    console.log("badiu")
                },
                divClick() {
                    console.log("我是divClick的點擊事件")
                },
                btnClick() {
                    console.log("我是btn的點擊事件")
                },
                innerClick(){
                    console.log("我是innerClick的點擊事件")
                }
            }
        })




    </script>


</body>

</html>

<!-- .prevent  阻止默认的行为 阻止标签到处跑-->

<!-- capture 先把事件传递给div 然后在给到btn 事件传递变了一个方向 -->

<!-- /stop事件不能給div傳遞事件  -->


 <!-- once 事件只触发一次 -->  
 <!-- stop和self的区别 -->

   <!-- self 只会阻止自己身上冒泡行为的触发，并不会真正阻止 冒泡行为 但是stop就可以完全的阻止 -->

```



#####  v-model 的数据双向绑定
* 在浏览器的 console里面输入 window.vm 就可以找到我们代码中赋值，当然我们也可以 vm.msg 获取我们代码里面赋值 ,我们也可以改变页面的msg的值 vm.msg="new"; 刷新一下 页面就改变额
* v-model 可以实现表单元素和model 中数据的双向绑定，注意v-model只能运用在表单元素中 input（address email text radio） select checkbox textarea 
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>

    <script src="./lib/vue.min.js"></script>
</head>
<body>
   <div id="app">

    <h4>{{msg}}</h4>
       <!-- 一个输入框 100%的宽度  但是 v-bind只能实现 数据的单项绑定 从m自动绑定到V
    无法实现数据的双向绑定
    
    -->
    <input type="text"  v-bind:value="msg" style="width: 100%;">
        <!-- 这个是个输入框，但输入的时候呢，页面上的值也会变，这就可以实现数据的双向绑定了 -->
    <input type="text"  v-model:value="msg" style="width: 100%;">
   </div>

   <script>
    //    这个vm会加载到浏览器的内存中区
    var vm=new Vue({
        el:"#app",
        data:{
            msg:"shiming tongxue"
        }
    })
   </script>
</body>
</html>

<!-- 在浏览器的 console里面输入 window.vm 就可以找到我们代码中赋值，
当然我们也可以 vm.msg 获取我们代码里面赋值 
我们也可以改变页面的msg的值 
vm.msg="new"; 刷新一下 页面就改变额 
-->


<!-- v-model 可以实现表单元素和model 中数据的双向绑定，注意v-model只能运用在表单元素中 
     input（address email text radio） select checkbox textarea  -->

```

####  实现的计算器

* `var codeStr="parseInt(this.n1)"+this.opt+"parseInt(this.n2)"this.result=eval(codeStr)`在项目中不要经常用这个方案，这是投机取巧的方式 


```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <script src="./lib/vue.min.js"></script>
</head>

<body>

    <div id="app">
        <input type="text" v-model="n1">
        <select v-model="opt">
            <option value="+">+</option>
            <option value="-">-</option>
            <option value="*">*</option>
            <option value="/">/</option>
        </select>

        <input type="text" v-model="n2">
        <input type="button" value="=" @click="calc">
        <input type="text" v-model="result">
    </div>

    <script>
        var vm = new Vue({
            el: "#app",
            data: {
                n1: 0,
                n2: 0,
                opt: "+",
                result: 0

            },
            methods: {
                // 等号的按钮事件
                calc(){
                    // 代码量太大
                //    switch(this.opt){
                //        case "+":
                //        this.result=parseInt(this.n1)+parseInt(this.n2)
                //        break; 
                //        case "-":
                //        this.result=parseInt(this.n1)-parseInt(this.n2)
                //        break; 
                //        case "*":
                //        this.result=parseInt(this.n1)*parseInt(this.n2)
                //        break; 
                //        case "/":
                //        this.result=parseInt(this.n1)/parseInt(this.n2)
                //        break; 
                //    }  
                //在项目中不要经常用这个方案，这是投机取巧的方式 
                var codeStr="parseInt(this.n1)"+this.opt+"parseInt(this.n2)"
                this.result=eval(codeStr)
                }
            }


        })
    </script>
</body>

</html>
```


####  Vue中使用样式

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <script src="./lib/vue.min.js"></script>
    <style>
        .red {
            color: red;
        }

        .thin {
            font-weight: 200;
        }

        /* 倾斜 */
        .italic {
            font-style: italic;
        }

        /* 间距的 */
        .active {
            letter-spacing: 0.5em;
        }
    </style>
</head>

<body>

    <div id="app">
        <h1 class="red thin italic active">大大大大大大大F</h1>
        <h1 class="red ">大大大大大大大F</h1>
        <!-- v-bind 绑定属性 但是这样找不到哦 -->
        <!-- <h1 :class="red">大大大大大大大F</h1> 这样纸是找不到的哦  -->


        <!-- 直接传递数据  使用v-bind数据绑定 ，但是没啥优点 -->
        <h1 :class="['red','italic','active','thin']">大大大大大大大Fdddd</h1>

         <!-- 在数组中使用三元表达式 -->
        <h1 :class="['red','italic','active',flag?'thin':'']">我是好同学 真的是好同学</h1>
        <h1 :class="['red','italic','active',flagw?'thin':'']">我是好同学 真的是好同学</h1>

<!-- 使用类 如果标记为true 就加载，不使用的话，就不加载 ，提高代码的可读性 -->
        <h1 :class="['italic','active',{'red':flag}]">我是好同学 真的是好同学</h1>
        <h1 :class="['italic','active',{'red':flagw}]">我是好同学 真的是好同学</h1>
        
        <!-- 前面是一个类名，后面是一个标识符 使用 v-bind 绑定对象的时候 -->
        <h1 :class="[{'active':true},{'red':true}]">我是好同学 true真的是好同学</h1>
        <h1 :class="[{'active':false},{'red':true}]">我是好同学 false真的是好同学</h1>
        <h1 :class="classObj">我是一个对象保存到里面去的</h1>
   
   
   
    </div>

    <script>
        var vm = new Vue({
            el: "#app",
            data: {
                flag:true,
                flagw:false,
                classObj:{active:false}
            },
            methods: {}

        })
    </script>
</body>

</html>
```

####　Vue使用Style
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <script src="./lib/vue.min.js"></script>
</head>
<body>
    
    <div id="app">
          
          <h1 :style="{color:'red','font-weight':'200'}">我是你的爸爸</h1>  

          <h1 :style="styleObj">我是你的爸爸</h1>  
          
          <h1 :style="styleObj2">我是你的爸爸</h1>  

           <!-- 绑定两个对象，我曹牛逼哦 -->
          <h1 :style="[styleObj2,styleObj]">我是你的爸爸</h1>  
    </div>

    <script>
    var vm=new Vue({
        el:"#app",
        data:{
          styleObj:{color:'red','font-weight':'200'}  ,
          styleObj2:{'font-style':'italic'}

        }
    })
    </script>
</body>
</html>
```

####　这就就是绑定key  有个疑问？？？现在最新版本的key可以使用对象了啊
* 原来的2.4.0的时候 只能使用 number或者string    key在使用的时候，必须使用v-bind属性绑定的形式，指定key的值 在组件中循环的时候，v-for 必须指定key值
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <script src="./lib/vue.min.js"></script>
</head>
<body>
    
    <div id="app">

         <div>
           <label>
               ID:<input type="text" v-model="id">
           </label>
           <label>
                Name:<input type="text" v-model="name">
            </label>
         </div>
         <input type="button" value='添加' @click="add">

        <!-- <p v-for="item in list" >
            <input type="checkbox">
            {{item.id}}  ---{{item.name}}
        </p> -->
        <!-- 这就就是绑定key  有个疑问？？？现在最新版本的key可以使用对象了啊
        原来的2.4.0的时候 只能使用 number或者string    key在使用的时候，必须使用v-bind属性绑定的形式，指定key的值 -->
      <!-- 在组件中循环的时候，v-for 必须指定key值  -->
        <p v-for="item in list" :key="item" >
                <input type="checkbox">
                {{item.id}}  ---{{item.name}}
            </p>
    
    </div>

    <script>
    var vm=new Vue({
        el:"#app",
        data:{
            id:"",
            name:"",
           list:[
               {id:1,name:"shiming"},
               {id:2,name:"shimingai"},
               ] 
        },
        methods:{
            add(){
                // 添加 这种方式没有问题
                // this.list.push({id:this.id,name:this.name})
                // 这种添加的方式 就有问题了 哈哈 难受哦 原因就是没有key 没有指定那一项被选中了
                this.list.unshift({id:this.id,name:this.name})

            }
        }

    })
    </script>
</body>
</html>
```

#### v-show 和v-if的区别 
* v-show 频换的切换
```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>

    <script src="./lib/vue.min.js"></script>
</head>

<body>
    <div id="app">
        <!-- 不能有空格啊  "button "  -->
        <input type="button" value="点我" @click="toggle">

        <input type="button" value="点我" @click="flag=!flag">

        <!-- 就相对于安卓中 gone 控件 都会重新创建和删除 消耗性能比较严重-->
        <h3 v-if="flag">这是 v-if 控制元素</h3>
        <!-- 就相当于 invisible 这个控件还在 不会重新创建和删除，只是切换了元素 dispaly：none -->
 <!-- 如果元素涉及到频换的切换 就是用v-show -->
        <h3 v-show="flag">这是 v-show 控制元素</h3>
    </div>
    <script>
        var vm = new Vue({
            el: "#app",
            data: {
                flag: true
            },
             methods: {
                toggle() {
                    this.flag = !this.flag
                }
            }

        })
    </script>
</body>

</html>
```
