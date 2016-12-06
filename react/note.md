#React全部过一遍
##JSX转义
中注入的js变量渲染时候都会被转义，这样可以防止XXS攻击。例如
```javascript
var element = (
    <h1>{test}</h1>
);
```
其中的test的内容如果是一个script标签怎么办？react渲染它之间会将其转义。可以避免一定的被攻击风险。

##react element
一个react元素是一个简单的对象。利用jsx语法创建一个react元素：
```javascript
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);
```
此时`element`元素相当于用`React.createElement`创建：
```javascript
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
```
最终，`element`只是一个简单的js对象而已。
```javascript
// Note: this structure is simplified
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world'
  }
};
```
一个React元素相当于js中的一个表达式，React元素中也可以通过插值插入一个完整的js表达式。注意js表达式的含义。
```javascript
//这是可以的，因为插值是一个表达式
var element = <h1>{(function(){return 'hello world';})()}</h1>  
//类似jsp的这种写法是不可以的，因为if(ture){不是一个完整的表达式
var element = <h1>{if(ture)\{}<div>{test}</div>{\}else\{\}}</h1> 
```
一个React元素和一个React组件是不同的。组件有组件的生命周期，组件是由React元素组成。从js的角度看，一个React元素是一个简单的js对象，而一个组件是一个方法，这个方法接受props的输入，然后输出对应的React元素。所以，根据这样的概念，以下方法其实就是一个React组件。
```javascript
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

##渲染react元素
要将一个react元素渲染为DOM，react提供了一个方法：`ReactDOM.render`
```javascript
ReactDOM.render(
    element,
    document.getElementById('root')
);
```
注意，react元素是Immutable的，所以要改变一个React元素的某个属性，目前为止只能重新创建一个元素：
```javascript
function click(){
var element = <h1>{new Date().getTime()}<span>hello world</span></h1>
    ReactDOM.render(
        element,
        document.getElementById('root')
    );
}
```
注意，react只会渲染修改的东西。上例中，即使每次click调用后element都是完全不同的一个元素，但是每次渲染时都只有时间会被重新渲染，span中的内容不会重新渲染。
##react组件
根据前面所述，react组件是一个方法，这个方法接受props输出一个react元素。这样的话就很好理解了：
```javascript
ReactDOM.render(
  <Welcome name="IAIAE" />,
  document.getElementById('root')
);
```
运行以上代码，当运行至`<Welcome />`组件时，ReactDOM会执行组件的方法，取得对应的React元素，再进行渲染。

##拆分组件
组件可以嵌套，在需要一个React元素的时候，你都可以用一个组件来代替。组件嵌套可以减轻复杂度，代码看起来也更加方便。

##state和生命周期
一个组件如何正确的设置state？

state一般是在组件的constructor中进行初始化：

```javascript
class Clock extends Component{
  constructor(props){
    super(props)
    this.state = {date: new Date()}
  }
}
```
然后用`setState`设置状态时对state的更新是异步的。所以设置state的时候不要依赖上一个state的值，因为上一个state的值可能还没有更新。所以以下的代码是不牢靠的：
```javascript
this.setState({
  counter: this.state.counter + this.props.increment,
});
```
但是我们需要在设置state的时候获取当前state的值怎么办？可以用`setState`的另一个重载，向setState传入一个方法：
```javascript
this.setState((prevState, props) => ({
  counter: prevState.counter + props.increment
}));
```
##处理事件
关于事件处理函数中的this。React组件中默认并不绑定自定义函数的执行环境。所以一下代码中handleClick中的this是模糊不清的。
```javascript
class Test extends Component{
  handleClick(){
    console.info(this);
  }
  render(){
    return <button onClick={this.handleClick}></button>
  }
}
```
要解决此问题，有三个方法，第一种是利用bind函数绑定this
```javascript
class Test extends Component{
  constructor(props){
    super(props);
    var that = this;
    ;['handleClick'].forEach(handlerName=>{
      that[handlerName] = that[handleName].bind(that);
    });
  }
  //或者在onClick中bind
  render(){
    return <button onClick={this.handleClick.bind(this)}></button>
  }
}
```
第二种，不要将handleClick写成一个实例方法。但是这样的做法还是实验性的，并不是一种最佳实践。
```javascript
class Test extends Component{
  handleClick = () => {
    // 由于箭头函数是词法作用域，此this还是指向了外部
    console.info(this);
  }
  //或者在onClick中bind
  render(){
    return <button onClick={this.handleClick}></button>
  }
}
```
第三种方法是用一个匿名函数调用handleClick。
```javascript
class Test extends Component{
  render(){
    return <button onClick={(e)=>this.handleClick(e)}></button>
  }
}
```
