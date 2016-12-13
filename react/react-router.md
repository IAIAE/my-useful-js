#React单页应用
我们一般使用`react-router`库来完成React页面的前端路由工作，以此构建React的单页应用。
##路由配置

```javascript
var spa = ReactDOM.render(
    <Provider store={store}>
        <Router history={hashHistory} routes={SPARouter} />
    </Provider>,
    document.getElementById('root')
);

// SPARouter.jsx
<Route path="/" component={SPAMain}>
   <Route path="index" component={Index}></Route>
   <Route path="users" component={Users}></Route>
</Route>
```



