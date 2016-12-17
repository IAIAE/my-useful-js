#React-router中路由的写法
##配置在独立文件中
这种办法是用一个router.js文件写清所有路由组件的从属关系。例如：

```javascript
<Router>
    <Route path="/" component={App}>
        <Route path="user" component={UserMain} >
            <Route path="avator" component={Avator} />
            <Route path="detail" component={Detail} />
        </Route>
        <Route path="about" component={AboutMain} />
        <Route path="message" component={MessageMain} >
            <Route path="email" component={Email} />
            <Route path="qq" component={QQ} />
        </Route>
    </Route>
</Router>
```
从上面的代码中很明显看出这个web应用的内容框架结构。缺点是一个大型的单页这个会非常复杂，一个页面中内容改动后还需要再改动路由才能生效。

##按组件划分，路由逻辑写在组件中
还有一种办法是将路由的逻辑实现在各自组件中。一个路由组件需要管理自己的所有子路由组件。还是上面的例子：

```javascript
var rootRoutes = {
    childRoutes: [
        path:'/',
        component: App,
        childRoutes:[
            UserMain,
            AboutMain,
            MessageMain
        ]
    ]
};
<Router routes={rootRoute} />

// UserMain/index.js
export default {
    path:'user',
    component: UserMainComponent
    childRoutes:[
        Avatar,
        Detail
    ]
}
```
文件结构是这样的：

```segement
-----UserMain
       |----components
       |        |----index.jsx
       |        |----index.scss
       |----index.js
       |----routes
                |----Avator
                |      |----components
                |      |        |----index.jsx
                |      |        |----index.scss
                |      |----index.js
                |
                |----Detail
                       |----components
                       |        |----index.jsx
                       |        |----index.scss
                       |----index.js         
```
这样做的好处是文件结构非常清晰，命名方式简单。缺点是很难从单一文件上对应用的理由结构有个总览的把握。不过路由结构可以通过查看目录结构也把握十之八九。

#代码分块，按需加载
运用第二种路由写法可以很方便的进行按需加载的改造，只需要把路由组件的入口文件改两个方法，那上面的UserMain/index.js来举例：

```javascript
// UserMain/index.js
export default {
    path:'user',
    getComponent: (nextState, done)=>{
        require.ensure([],require=>{
            done(null, require('./components/index.jsx'));
        });
    },
    getChildRoutes:(partialNextState, done) => {
        require.ensure([],require=>{
            done(null,
                require('./routes/Avator'),
                require('./routes/Detail')
            );
        });
    }
}
```
真心觉得优雅！！

个人觉得这样的文件组织结构弱化了组件的复用。比如上面例子的Detail组件，很难想到有人会require这么深的层级去复用这个组件。需要复用的组件应该都是放在外层的一个common文件夹中。所以，我觉得以路由path的值来命名文件夹更好，点开目录结构就可以看请url的层级：

```segement
-----user
       |----components
       |        |----index.jsx
       |        |----index.scss
       |----index.js
       |----routes
                |----avator
                |      |----components
                |      |        |----index.jsx
                |      |        |----index.scss
                |      |----index.js
                |
                |----detail
                       |----components
                       |        |----index.jsx
                       |        |----index.scss
                       |----index.js  
```
上例中，一看就知道url的组成为`/user/detail`和`/user/avator`，并不用去在意构成user这个路由的组件名叫做`UserMain`，因为这个组件不是拿来复用的，它只是构成user的index，这些组件的标识符不是`UserMain`而是`/user`

#向子元素传递属性
当已知一个子元素的类型，不如

```javascript
render(){
    return <Hello>hello world</Hello>;
}
```
我可以很方便的给其传递属性：

```javascript
render(){
    return <Hello remove={this.removeItem}>hello world</Hello>
}
```
当并不知道子组件的具体类型的时候：

```javascript
render(){
    return this.props.children;
}
```
可以通过`React.cloneElement`来赋值属性：

```javascript
render(){
    return React.cloneElement(
        this.props.children,
        {remove:this.removeItem});
}
```

#url中的参数与查询
当一个url匹配一个组件的时候，url中的附加信息会以props的形式传递给组件。

```javascript
<Route path="user/(:name)" component={User} />

// 访问user/bob?age=26

// User.jsx
const User = ({children, params:{name}, location:{query}}) => {
    // name === 'bob'
    // query === {age:26}
}
```
#404
配置`path="*"`可以设置404的页面。

```javascript
<Route path="/" component={App}>
    <Route path="*" component={PageNotFound} />
</Route>
```


#API
##Router.createElement(Component, props)
当Router匹配一个路由时，会调用`createElement`方法创建对应组件。例如：

```javascript
<Router>
    <Route path="/" component={App} />
</Router>
```
当匹配路由`/`时，Router会调用默认的`createElement`方法创建App组件，默认的`createElement`方法实现如下：

```javascript
var createElement = (Component, props) => {
    return <Component {...props} />;
}
```
如果覆写这个方法，可以实现一些包装：

```javascript
<Router createElement={createElement}>
    <Route path="/" component={App} />
</Router>
var createElement = (Component, props) => {
    return <RelayContainer component={Component} routerProps={props} />
}
```



