---
title: Vue.js学习No.3
date: 2019-03-27 09:32:29
tags: Vue
---
* **欢迎关注我的公众号**
![公众号](https://upload-images.jianshu.io/upload_images/5363507-0a0cf2e5fd8f843d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* **学习的内容如下**
<!--  more  --> 

* **开始**
#### 使用`vue-resource`
* 导入`bootstrap`也可以这样导入,这个是最新的版本
```
 <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css"
        integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous">


```

*   点击这两个删除的区别,一个会重新刷新网页
                   ```     <a href="" @click.prevent="del">删除</a>
                        <a href="">删除</a>```


* 全局配置地址  
*  如果配置了请求的数据接口根域名，在每次单独发起`http`的时候，也因该是相对路径开头，不能带斜线，否则不会启用根路径拼接
*  `{emulateJSON:true} `以普通表单格式，将数据提交给服务器 `application/x-www-form-urlencoded`
               
```
      Vue.http.options.root = "http://shiming"
        //全局启用这个配置
        Vue.http.options.emulateJSON = true;
```
* Demo
```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <script src="./lib/vue.min.js"></script>

    <!-- vue-resource 依赖于 Vue 有个先后的顺序  this.$http.get -->
    <script src="https://cdn.jsdelivr.net/npm/vue-resource@1.5.1"></script>
    <!-- 这个包才能一样 -->
    <link rel="stylesheet" href="./lib/bootstrap.min.css">

    <script src="https://cdn.jsdelivr.net/npm/vue-resource@1.5.1"></script>

    <!-- 这个是个最新的包 -->
    <!-- <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css"
        integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous"> -->
</head>

<body>

    <div id="app">






        <div class="panel panel-primary">
            <div class="panel-heading">
                <h3 class="panel-title">添加</h3>
            </div>
            <div class="panel-body form-inline">


                <label>
                    Name:
                    <input type="text" v-model="name" class="form-control">

                </label>

                <input type="button" value="添加" @click="add" class="btn btn-primary">


            </div>
        </div>


        <table class="table table-bordered table-hover table-striped">
            <thead>
                <tr>
                    <th>Id</th>
                    <th>Name</th>
                    <th>Ctime</th>
                    <th>Operation</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="item in list" :key="item.id">
                    <td>{{item.id}}</td>
                    <td>{{item.name}}</td>
                    <td>{{item.ctime}}</td>
                    <td>
                        <!-- 点击这两个删除的区别 -->
                        <a href="" @click.prevent="del">删除</a>
                        <a href="">删除</a>
                    </td>
                </tr>
            </tbody>
        </table>


    </div>

    <script>
        // 全局配置地址  
        // 如果配置了请求的数据接口根域名，在每次单独发起http的时候，也因该是相对路径开头，不能带斜线，否则不会启用根路径拼接


        Vue.http.options.root = "http://shiming"
        //全局启用这个配置
        Vue.http.options.emulateJSON = true;



        var vm = new Vue({
            el: "#app",
            data: {

                name: "",
                list: [
                    { id: 1, name: "宝马", ctime: new Date() },
                    { id: 2, name: "奔驰", ctime: new Date() }
                ]
            },
            methods: {
                // post
                // {emulateJSON:true} 以普通表单格式，将数据提交给服务器 application/x-www-form-urlencoded
                add() {
                    this.$http.post("", { name: this.name }, { emulateJSON: true }).then(result => {
                        if (result.body.status === 0) {
                            //    成功了在获取数据
                            this.getAllList()
                            // 清空输入框
                            this.name = ""

                        } else {

                        }
                    })
                },
                // get
                getAllList() {
                    console.log("方法请求了 getAllList")
                    // 如果配置了请求的数据接口根域名，在每次单独发起http的时候，也因该是相对路径开头，不能带斜线，否则不会启用根路径拼接
                    this.$http.get("api/getprodlist", { headers: { 'Access-Control-Allow-Origin': '*' } }).then(result => {

                        if (result.status === 0) {

                        } else {
                            alert("请求失败");
                            console.log(result)
                        }
                    }, response => {
                        // error callback
                        alert("请求失败");
                        console.log("原因：：：" + response)
                    }
                    )
                },

                del() {

                }
            },
            created() {
                console.log("方法请求了 created")
                this.getAllList()
            }
        }
        )

    </script>

</body>

</html>
```
#### Vue动画

v-enter：定义进入过渡的开始状态。在元素被插入之前生效，在元素被插入之后的下一帧移除。

v-enter-active：定义进入过渡生效时的状态。在整个进入过渡的阶段中应用，在元素被插入之前生效，在过渡/动画完成之后移除。这个类可以被用来定义进入过渡的过程时间，延迟和曲线函数。

v-enter-to: 2.1.8版及以上 定义进入过渡的结束状态。在元素被插入之后下一帧生效 (与此同时 v-enter 被移除)，在过渡/动画完成之后移除。

v-leave: 定义离开过渡的开始状态。在离开过渡被触发时立刻生效，下一帧被移除。

v-leave-active：定义离开过渡生效时的状态。在整个离开过渡的阶段中应用，在离开过渡被触发时立刻生效，在过渡/动画完成之后移除。这个类可以被用来定义离开过渡的过程时间，延迟和曲线函数。

v-leave-to: 2.1.8版及以上 定义离开过渡的结束状态。在离开过渡被触发之后下一帧生效 (与此同时 v-leave 被删除)，在过渡/动画完成之后移除。

![transition.png](https://upload-images.jianshu.io/upload_images/5363507-dd337e0af85dd024.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/animate.css/3.7.0/animate.min.css">
```

*  Vue 提供的 实现动画的话，就必须带上 V- 如果不带的话在transition就不能显示   所以我可以在外面显示

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <script src="./lib/vue.min.js"></script>
    <!-- 自定義樣式 -->
    <style>
        /* 可以设置不同的进入和离开动画 */
        /* 设置持续时间和动画函数 */
        .slide-fade-enter-active {
            transition: all 3s ease;
        }

        .slide-fade-leave-active {
            transition: all 8s cubic-bezier(1.0, 0.5, 0.8, 1.0);
        }

        /* 進入的時間 */
        .slide-fade-enter,
        .slide-fade-leave-to

        /* .slide-fade-leave-active for below version 2.1.8 */
            {
            transform: translateX(900px);
            opacity: 0;
        }


        /* 可以设置不同的进入和离开动画 */
        /* 设置持续时间和动画函数 */
        .my-enter-active {
            transition: all 3s ease;
        }

        .my-leave-active {
            transition: all 8s cubic-bezier(1.0, 0.5, 0.8, 1.0);
        }

        /* 進入的時間 */
        .my-enter,
        .my-leave-to

        /* .slide-fade-leave-active for below version 2.1.8 */
            {
            transform: translateX(100px);
            opacity: 0;
        }
    </style>
</head>

<body>

    <div id="example-1">
        <button @click="show = !show">
            Toggle render
        </button>

        <transition name="slide-fade">
            <p v-if="show">仕明同学</p>
        </transition>
        <!-- Vue 提供的 实现动画的话，就必须带上 V- 如果不带的话在transition就不能显示
        所以我可以在外面显示
        -->
        <transition name="my">

            <p v-if="show">仕明同学Copy</p>

        </transition>
    </div>


    <script>
        new Vue({
            el: '#example-1',
            data: {
                show: true
            }
        })
    </script>

</body>

</html>
```
* 使用开源的动画
* **[animate.css](https://github.com/daneden/animate.css)**
* [animate.css动画的效果](https://daneden.github.io/animate.css/)

*  需要加`animated `在前面 同时前面不能有空格
```
<transition enter-active-class="animated bounceIn" leave-active-class="animated bounceOut"  :duration="100">

            <h1 v-if="flag">看我的动画</h1>
        </transition>
```
* `duration `设置动画的执行的时间，但是我写出来好像没有效果
* `duration="毫秒值"` 来设置动画的市场
* `:duration="{ enter: 100, leave:500 }"` 设置入场和离场的市场
```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <script src="./lib/vue.min.js"></script>
    <!-- 使用第三方动画实现的库 -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/animate.css/3.7.0/animate.min.css">
</head>

<body>

    <div id="app">
        <input type="button" value="按钮" @click="flag=!flag">

        <!-- 需要加animated 在前面 同时前面不能有空格 -->
        <transition enter-active-class="animated bounceIn" leave-active-class="animated bounceOut"  :duration="100">

            <h1 v-if="flag">看我的动画</h1>
        </transition>
        <!-- :duration 我咋设置的不管作用啊  -->
        <!-- duration="毫秒值" 来设置动画的市场  :duration =  -->
        <transition enter-active-class="animated zoomOutUp" leave-active-class="animated zoomOutDown"  :duration="10000">

            <h1 v-if="flag">看我的动画有时间 </h1>
        </transition>
        <transition enter-active-class="animated zoomOutUp" leave-active-class="animated zoomOutDown" >

                <h1 v-if="flag">看我的动画没时间</h1>
            </transition>
        <!--  :duration="{ enter: 100, leave:500 }" 设置入场和离场的市场 -->
        <transition enter-active-class="animated zoomOutLeft" leave-active-class="animated zoomOutRight"
            :duration="{ enter: 100, leave:500 }">

            <h1 v-if="flag">看我的动画duration</h1>
        </transition>




    </div>


    <script>
        var vm = new Vue({
            el: "#app",
            data: {
                flag: true
            },
            methods: {

            }

        })
    </script>
</body>

</html>
```



### [JavaScript 钩子函数实现半场动画](https://cn.vuejs.org/v2/guide/transitions.html#JavaScript-%E9%92%A9%E5%AD%90 "JavaScript 钩子")
* 加入购物车的动画
```
<transition
  v-on:before-enter="beforeEnter"
  v-on:enter="enter"
  v-on:after-enter="afterEnter"
  v-on:enter-cancelled="enterCancelled"

  v-on:before-leave="beforeLeave"
  v-on:leave="leave"
  v-on:after-leave="afterLeave"
  v-on:leave-cancelled="leaveCancelled"
>
  <!-- ... -->
</transition>
```

* Demo
```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>

    <script src="./lib/vue.min.js"></script>
    <!-- 使用第三方动画实现的库 -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/animate.css/3.7.0/animate.min.css">

    <style>
        .ball {
            width: 15px;
            height: 15px;
            border-radius: 50%;
            background-color: blueviolet
        }
    </style>
</head>

<body>

    <div id="app">

        <input type="button" value="点我" @click="show=!show">
<!-- before-enter 动画么有开始，可以设置元素的样式，设置元素的位置 -->
<!-- enter 表示动画开始之后的样式 -->
        <transition @before-enter="before" @enter="enter" @after-enter="ater">
            <div class="ball" v-show="show"></div>
        </transition>



    </div>
    <script>
        var vm = new Vue({
            el: "#app",
            data: {
                show: true

            },
            methods: {
                // 第一个el参数是一个元素的js对象，
                // el认为就是document.getElementById 获取到的原生对象
                before(el) {
                    console.log("before");
                    el.style.transform = "translate(0,0)"

                },
                // 细细体会以下这个done的函数
                enter(el,done) {
                    // 这个必须写，要不然出来动画
                    // 可以认为这个方法强制刷新，位置在偏移
                    // 以下方法都是可以的
                    // el.offsetWidth
                    // el.offsetHeigh
                    // el.offsetLeft
                    el.offsetTop
                    console.log("enter");
                    el.style.transform = "translate(150px,450px)"
                    el.style.transition="all 5s ease"
                //    动画结束的时候消失小球作用的
                    done()
                    // done就是ater函数的引用
                },
                ater(el) {
                    console.log("ater");
                    this.show=!this.show;
                },
            }

        })
    </script>
</body>

</html>
```
#### 列表动画

* 鼠标覆盖的颜色:可以设置`style`
```  /* 鼠标覆盖的颜色 */
        li:hover {
            background-color: aqua;
            /* 背景也可以过度，就是鼠标覆盖在上面有个小小的动画*/
            transition: all 3s ease;
        }
```
* `v-move` 和 `.v-leave-active` 配合使用，可以实现列表元素的删除动画,设置元素位移时候的动画 但是还要设置` v-leave-active `设置类 `absolute`
```
  /* 可以设置元素位移时候的动画 但是还要设置 v-leave-active 设置类 absolute*/
        /* .v-move 和 .v-leave-active 配合使用，可以实现列表元素的删除动画 */
        .v-move {
            transition: all 1s ease;
        }
```
* 在实现列表过度的时候，如果需要过度的元素是通过`v-for`渲染出来的，不能使用` transition` 包裹,需要使用` transition-group`

* 给`transitiong-group ` 加上 `appear` 实现页面刚展示出来的入场效果


* 通过tag属性设置，指定transiton-group渲染为指定的元素 如果不指定的，默认渲染为span
```
 <!-- 通过tag属性设置，指定transiton-group渲染为指定的元素 如果不指定的，默认渲染为span -->
            <transition-group appear >
                <li v-for="(item,i) in list" :key="item.id" @click="del(i)">
                    {{item.id}} ----- {{item.name}}
                </li>
            </transition-group>
```
![image.png](https://upload-images.jianshu.io/upload_images/5363507-5de2d19abcb32173.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 这是指定了` tag="ul"`
```
 <!-- 通过tag属性设置，指定transiton-group渲染为指定的元素 如果不指定的，默认渲染为span -->
            <transition-group appear tag="ul">
                <li v-for="(item,i) in list" :key="item.id" @click="del(i)">
                    {{item.id}} ----- {{item.name}}
                </li>
            </transition-group>
```
![image.png](https://upload-images.jianshu.io/upload_images/5363507-f62941db775fd367.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* Demo
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
        li {
            border: 2px dashed #ff0;
            margin: 5px;
            line-height: 35px;
            padding: 10px;
            font-size: 50px;
            width: 100%;
        }

        /* 鼠标覆盖的颜色 */
        li:hover {
            background-color: aqua;
            /* 背景也可以过度，就是鼠标覆盖在上面有个小小的动画*/
            transition: all 3s ease;
        }

        .v-enter,
        .v-enter-to {
            opacity: 0;
            transform: translateY(80px)
        }

        .v-enter-active,
        .v-enter-active {
            transition: all 2s ease;
        }

        /* 可以设置元素位移时候的动画 但是还要设置 v-leave-active 设置类 absolute*/
        /* .v-move 和 .v-leave-active 配合使用，可以实现列表元素的删除动画 */
        .v-move {
            transition: all 1s ease;
        }

        .v-leave-active {
            position: absolute;
        }
    </style>
</head>

<body>

    <div id="app">

        <div>
            <label>
                Id:
                <input type="text" v-model="id">
            </label>

            <label>
                Name:
                <input type="text" v-model="name">
            </label>

            <input type="button" value="加" @click="add">
        </div>
        <ul>
            <!-- 在实现列表过度的时候，如果需要过度的元素是通过v-for渲染出来的，不能使用 transition 包裹
        需要使用 transition-group
        -->
            <!-- 当然也必须的设置key的属性 -->
            <!-- <li v-for="item in list" :key="item.id">
               {{item.id}} ----- {{item.name}}
            </li> -->
            <!-- 给transitiong-group  加上 appear 实现页面刚展示出来的入场效果 -->
           
           <!-- 通过tag属性设置，指定transiton-group渲染为指定的元素 如果不指定的，默认渲染为span -->
            <transition-group appear tag="ul">
                <li v-for="(item,i) in list" :key="item.id" @click="del(i)">
                    {{item.id}} ----- {{item.name}}
                </li>
            </transition-group>

        </ul>
    </div>

    <script>
        var vm = new Vue({
            el: "#app",
            data: {
                id: "",
                name: "",
                list: [
                    { id: 1, name: "世明" },
                    { id: 2, name: "pangfan" },
                ]
            },
            methods: {
                add() {
                    this.list.push({
                        id: this.id,
                        name: this.name
                    })
                    this.name = ""
                    this.id = ""
                },
                del(i) {
                    this.list.splice(i, 1)
                }
            }
        })
    </script>
</body>

</html>
```
#### 组件
* 组件的创建一

  * 1、使用 Vue.extend 组件
   ```
  var com = Vue.extend({
            template: "<h3>我是一个com组件<h3>"
        })
  ```
  *  2、使用Vue.component
  ```Vue.component("com",com)   ```
  *  3、使用组件，以html标签引入进来 -->
      ```  <com></com>  ```
  *  4、驼峰的命名规则，使用my-com来引入，其他的可能有点问题 
   ```        <my-com></my-com>  ```

* 组件的创建二
  * 使用Vue.component
   ``` 
          Vue.component("com",Vue.extend({
            template: "<h3>我是一个myCom组件<h3>"
        })) 
   ``` 
* 组件的创建三
   ``` 
  Vue.component("com",{
            template: "<h3>我是一个com组件<h3><span>123ddd</span>"
        })   
    ``` 

* 定义私有组件 :最重要的引用的方法是 "#app"
```
<body>
    <div id="app">
        
        <login></login>
         <test></test>
    </div> 

    <template id="temp">

        <h1>我是全局组件，但是我在私有里面引用了的</h1>
    </template>
    
    <script>
        var vm = new Vue({
            el: "#app",
            //单例
            data() {
                return {

                }
            },
            methods: {

            },
            filters: {

            },
            directives: {},
            //定义私有组件
            components: {
              login:{
                  template:"<h1> 私有组件</h1>"
              },
              test:{
                  //记得加上# 引用
                    template:"#temp"
              }
            },
            //颜色不一样 
            beforeCreate() {

            },
        })
    </script>
</body>
```
* 组件中的`data`和`methods` :组件中的`data`是一个方法，方法内部返回一个对象 
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
        <tem></tem>
    </div>


    <script>

        Vue.component("tem", {
            template: "<h1>你要好好学习---{{msg}}---</h1> ",
            //组件中的data是一个方法，方法内部返回一个对象 
            data: function () {
                return {
                    msg: "组件的data的msg属性"
                }

            }
        })
        var vm = new Vue({
            el: "#app",
            data() {
                return {

                }
            },
            methods: {

            },
        })
    </script>
</body>

</html>
```
* 为什么组件中的`data`返回一个对象，下面的例子可以很好地说明，说通俗一点，当页面上引用了很多组件的话，组件是同一个对象，我们去改变一个对象，其他的就全部改变了，没有意义了
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
        <tem></tem>
        <!-- 当组件应用更多的时候，就会一起改变，肯定是不行 -->
        <counter></counter>
        <hr>
        <counter></counter>
        <hr>
        <counter></counter>
        <hr>
        <counter></counter>
    </div>


    <!-- 在外面定义组件可以有代码提醒 -->
    <template id="temp1">
        <div>
            <input type="button" value="+1" @click="add">
            <h3>{{count}}</h3>
        </div>

    </template>


    <script>

        var obj = { count: 0 }
        //计数器组件，点击按钮，数目加上1 
        Vue.component("counter", {
            // 定义组件 一定要命名争取啊 我错了两次了
            template: "#temp1",
            data: function () {
                // 所以这里不能够这样返回
                // return obj
                //保证每次单独的返回一个对象
                return { count: 0 }
            },
            methods: {
                add() {
                    this.count += 1
                }
            }
        })

        Vue.component("tem", {
            template: "<h1>你要好好学习---{{msg}}---</h1> ",
            //组件中的data是一个方法，方法内部返回一个对象 
            data: function () {
                return {
                    msg: "组件的data的msg属性"
                }

            }
        })
        var vm = new Vue({
            el: "#app",
            data() {
                return {

                }
            },
            methods: {

            },
        })
    </script>
</body>

</html>

```
* **组件的切换**
* 1、v-if和v-else，就是一个判断，但是只能切换两个组件
* 2、Vue提供 component ，来展示对应的名称的组件， componentId 是一个占位符 :is=属性，可以用来展示组件的名称 `<component :is="name"></component>`可以切换多个组件。
* Vue的标签
*  `component`组件 ` template   transition（过渡）   transition-group（列表过渡）`

* 多个组件的过渡简单很多 - 我们不需要使用 `key `特性。相反，我们只需要使用动态组件：过度的模式`mode="out-in" `先`out`在`in`
```
   <transition mode="out-in">
            <component :is="name"></component>
        </transition>
```
* Demo
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
        .v-enter,
        .v-leave-to {
            opacity: 0;
            transform: translateX(150px)
        }

        .v-enter-active,
        .v-leave-active {
            transition: all 0.5s ease;
        }
    </style>
</head>

<body>

    <div id="app" id="app">
        <h1>组件的切换</h1>

        <!-- 只能切换两个 尴尬 -->
        <a href="" @click.prevent="flag=true">登陆</a>
        <a href="" @click.prevent="flag=false">注册</a>

        <demo1 v-if="flag"></demo1>
        <demo2 v-else="flag"></demo2>
        <h1>Vue提供 component ，来展示对应的名称的组件</h1>
        <component :is="'demo1'"></component>
        <h1>组件的切换2</h1>

        <a href="" @click.prevent="name='demo1'">登陆2</a>
        <a href="" @click.prevent="name='demo2'">注册2</a>
        <!--Vue提供 component ，来展示对应的名称的组件  -->
        <!-- componentId 是一个占位符 :is=属性，可以用来展示组件的名称 -->

        <!-- 多个组件的过渡简单很多 - 我们不需要使用 key 特性。相反，我们只需要使用动态组件： -->
        <!-- 过度的模式mode="out-in" 先out在in -->
        <transition mode="out-in">
            <component :is="name"></component>
        </transition>
    </div>
    <script>
        Vue.component("demo1", {
            template: "<h2>登陆</h2>"
        })
        Vue.component("demo2", {
            template: "<h2>注册</h2>"
        })
        var vm = new Vue({
            el: "#app",
            data() {
                return {
                    flag: true, name: "demo1"
                }
            },
            methods: {

            },
        })
    </script>
</body>

</html>

<!-- Vue的标签 -->
<!-- component组件  template   transition   transition-group-->
```