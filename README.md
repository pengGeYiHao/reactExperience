#  react+redux+reactRouter+sails 项目完成后技术心得

##  生命周期

####  第一次渲染
+  componentWillMount
	-  这里setState初始数据,如果在componentDidMount中setState虽然效果一样但是其实是经历了两次render,第一次render是没有数据的
+  render
+  componentDidMount
	
#### 非第一次渲染
+  componetWillReceiveProps
+  shouldComponentUpdate
	-  这里可以更好的控制组件的更新,尽量不要在这里setState否则有可能会造成死循环,因为每次setState都会走这里,如果在这里setState的话又会进行重新渲染
	-  接受两个参数nextProps,nextState，这里可以更好的控制组件的渲染，（return false/true）
+  componentWillUpdate
+  render
+  componentDidUpdate
	
#### 卸载
	componentWillUnmount
	
##  语法

+  组件最外层必须有唯一一个标签包裹
+  标签中属性变量使用{},不变量使用{{}}
+  注释使用{/**/}
+  组件首字母要大写
+  jsx中只允许使用三元运算符进行逻辑判断，在标签中需要使用{}包裹js代码

##  数据传递

+  props是用来父子传递数据的，不要修改，可以setState来达到更新组件的目的，每个组件应该操作自己的state，避免子组件操作父组件的state，这样会很乱
+  非父子组件传递数据可以使用redux，redux就相当于弄了个全局的store，修改它需要去dispatch触发相应的reducers，reducers在根据对应的action返回新的store，因为所有的reducers和action都是统一维护的，所以就不会出现混乱的情况

##  路由

4.0之前路由分段可以使用getComponent
4.0 之后react-router没有了这个api

可以使用以下方式
>  bundle.js 文件
 ``` javascript
 
    import React, {Component} from 'react'  
    export default class Bundle extends Component {
        constructor(props){
            super(props)
            this.state={
                mod:null
            }
        }
        componentWillMount() {
            this.load(this.props)
        }
        componentWillReceiveProps(nextProps) {
            if (nextProps.load !== this.props.load) {
                this.load(nextProps)
            }
        }
        load(props) {
            this.setState({mod: null})
            // props.load((mod) => {
            //     this.setState({
            //         mod: mod.default ? mod.default : mod
            //     })
            // })
            props.load().then((mod) => {
                this.setState({
                    mod: mod.default ? mod.default : mod
                });
            });
        }
        render() {
            if (!this.state.mod) return false;
            return this.props.children(this.state.mod)
        }
    }
```


> routers.js 文件
```javascript

import { matchRoutes, renderRoutes } from 'react-router-config'

import Home from 'CONTAINERS/Home'
import Bundle from 'UTILS/bundle'

const routes =[
    {
        path:'/',
        exact:true,
        component:Home,
    },
    {
        path:'/login',
        component:() => (
                    <Bundle load={()=> import('CONTAINERS/Login') }>
                        {(Component) => <Component />}
                    </Bundle>
                )
    },
    {
        path:'/admin',
        component:() => (
                    <Bundle load={()=> import('COMPONENTS/Main/Test1') }>
                        {(Component) => <Component />}
                    </Bundle>
                )
    },
]

export default routes
```

> webpack出口配置中添加
```javascript
chunkFilename: '[name]-[id].[chunkhash:8].bundle.js'
```


##  antDesign  UI库

在Tree组件中，title属性会是一个a标签，所以你如果在往title中加a或者Link标签会报错。a标签中不能包含a标签，可以使用Object标签包裹的方式解决

##  sails

传递数据的时候他会把数据中的"+"号给替换为空格

## babel

> .babelrc文件
```javascript
{
  "presets": [
    ["es2015", {"modules": false}],   //转译jsx语法和es6，es7语法

    "es2017",

    "stage-2",  //stage-3  import() 语法报错

    "es2016",

    "react"
  ],
  "plugins": [
    "react-hot-loader/babel",
	//  解析  async  await  语法
    "babel-plugin-transform-async-to-generator",
    "babel-plugin-external-helpers",
    "transform-runtime",
    "babel-plugin-transform-es2015-modules-commonjs"
  ]
}
```

##  对比vue和angular1.X

###  angular1.x

$scope变量中使用脏值检查来实现.

$scope.$watch函数，监视一个变量的变化。函数有三参数，”要观察什么”，”在变化时要发生什么”,以及你要监视的是一个变量还是一个对象。

使用ng-model时，你可以使用双向数据绑定。 
使用$scope.$watch（视图到模型）以及$scope.$apply（模型到视图），还有$scope.$digest

调用$scope.$watch时只为它传递了一个参数，无论作用域中的什么东西发生了变化，这个函数都会被调用。在ng-model中，这个函数被用来检查模型和视图有没有同步，如果没有同步，它将会使用新值来更新模型数据。


在AngularJS双向绑定中，有2个很重要的概念叫做dirty check，digest loop，dirty check（脏检测）是用来检查绑定的scope中的对象的状态的，例如，在js里创建了一个对象，并且把这个对象绑定在scope下，这样这个对象就处于digest loop中，loop通过遍历这些对象来发现他们是否改变，如果改变就会调用相应的处理方法来实现双向绑定

Vue 也支持双向绑定，默认为单向绑定，数据从父组件单向传给子组件。在大型应用中使用单向绑定让数据流易于理解。

###  vue

Vue.js 有更好的性能，并且非常非常容易优化，因为它不使用脏检查。Angular，当 watcher 越来越多时会变得越来越慢，因为作用域内的每一次变化，所有 watcher 都要重新计算。并且，如果一些 watcher 触发另一个更新，脏检查循环（digest cycle）可能要运行多次。 Angular 用户常常要使用深奥的技术，以解决脏检查循环的问题。有时没有简单的办法来优化有大量 watcher 的作用域。Vue.js 则根本没有这个问题，因为它使用基于依赖追踪的观察系统并且异步列队更新，所有的数据变化都是独立地触发，除非它们之间有明确的依赖关系


###  react

React 的渲染建立在 Virtual DOM 上——一种在内存中描述 DOM 树状态的数据结构。当状态发生变化时，React 重新渲染 Virtual DOM，比较计算之后给真实 DOM 打补丁。

Virtual DOM 提供了函数式的方法描述视图，它不使用数据观察机制，每次更新都会重新渲染整个应用，因此从定义上保证了视图与数据的同步。它也开辟了 JavaScript 同构应用的可能性。

在超大量数据的首屏渲染速度上，React 有一定优势

React 的 Virtual DOM 也需要优化。复杂的应用里可以选择 1. 手动添加 shouldComponentUpdate 来避免不需要的 vdom re-render；
