#基本的React
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
##隐藏一个组件
根据一些条件，可能需要一个组件隐藏或者显示。我以前一般的做法是用`style={{display:'none'}}`，其实还有更加符合标准的做法。

React定义一个组件的render返回null的时候就不渲染这个组件。所以
```javascript
class Warning extends Component{
  render(){
    if(this.props.show){
      return <span>warning!</span> 
    }else{
      return null; 
    }
  }
}
//或者
function Warning(props){
  return props.show? <span>warning!</span> : null;
}
```
还有，在React元素中注入一个js表达式为false时候，React会忽略这个false
```javascript
function Mailbox(props) {
  const unreadMessages = props.unreadMessages;
  return (
    <div>
      <h1>Hello!</h1>  
      // 如果返回false的话，会直接忽略这里的注入
      {unreadMessages.length > 0 &&
        <h2>
          You have {unreadMessages.length} unread messages.
        </h2>
      }
    </div>
  );
}
```
##条件渲染
在jsp中我们可以利用if条件语句选择进行渲染的方式。React中的注入脚本必须是一个js表达式，这点一定要注意。

条件渲染的方式多种多样，这里写两种：
```javascript
function Test(props){
  return (
    <h1>Hello! </h1>
    {
      props.length>0 && <span>you have {props.length} messages</span>
    }
  );
}
```
```javascript
function Test(props){
  return (
    <h1>Hello! </h1>
    {
      props.length>0?(
        <span>you have messages</span> 
      ):(
        <span>you have no messages</span>
      )
    }
  );
}
```
##列表和key
当React中包含相同React元素的列表的时候，需要给每一个列表元素加上一个key。key属性是React用于辨识列表元素的一个特殊的属性，最好不要将prop属性名写为key：
```javascript
const content = postsmap((post)=>
  <Post 
    key={post.id}
    id={post.id}
    title={post.title} />
);
```
上面的代码中，`props.key`是无法读取到的。
##form表单
html中的form表单拥有一些自动的处理逻辑，比如，可以为input标签设置name，提交的时候根据name来获取数据，还有type等于submit的标签默认有post提交功能等等。在React中，官方给出了一种限制这些原生能力的办法，称为：“受控组件”（Controlled Components）。受控组件主要针对form表单中的各个控件，为各控件添加监听方法，限制控件的改动，并且按需进行渲染。例如：
```javascript
class ControlledForm extends Component{
  handleChange(e){
    this.setState({value:e.target.value});
  }
  handleSubmit(e){
    console.info(this.state.value);
    e.preventDefault();
  }
  render(){
    <form onSubmit={this.handleSubmit}>
      <input type="text" value={this.state.value} onChange={this.handleChange} />
      <input type="submit" value="提交">
    </form>
  }
}
```
以上组件中，input标签是受限制的，它渲染的值是`this.state.value`，所有用户输入都会经过`handleChange`事件的处理，然后再将最终数据返回给input标签渲染出来，这样，就可以利用js脚本控制所有用户的输入，包括提交了。

然当，如果不使用受控组件，还可以通过ref获取input标签的DOM，从而获取input的值。这种办法称为“非受控组件”。组件是“受控”的好还是“非受控”好，这个问题没有固定的结论，官方给出的意见是，如果需要实时的监控用户输入的话，建议使用“受控组件”。

##提升状态变化的控制权
有时候，子组件的state变换不想用子组件本身去处理，需要将子组件状态变化的控制权交给父组件来处理，这种技巧叫做“控制权提升”。做法是将父组件的一个方法通过prop的方式传递给子组件，子组件在需要state发生变化时调用父组件传过来的方法。就这么简单。

##组件组合与组件继承
组件可以通过`props.children`访问自己囊括的其他组件。例如
```javascript
function MainView(props){
  return (
    <div id="main">
      {props.children}
    </div>
  )
}
ReactDOM.render(
  <MainView>
    <p>this is content</p>
  </MainView>,
  document.getElementById('root')
);
```
以上代码中，React会将MainView囊括的所有内容传递给props.children参数。

如果你觉得一个chilren不够用个，当然可以自己定义属性。例如：
```javascript
function MainView(props){
  return (
    <div id="main">
      <div id="left">
        {props.left}
      </div>
      <div id="right">
        {props.right}
      </div>
    </div>
  );
}
ReactDOM.render(
  <MainView 
  left={<LeftContentView />}
  right={<RightContentView />} />,
  document.getElementById('root')
);
```
FB官方文档上不推荐用组件继承。

#高级指引
##JSX语法探析
jsx语法只是创建React元素的一个语法糖。对于jsx语法：
```javascript
<MyButton color="blue" shadowSize={2}>
  Click me  
</MyButton>
```
它其实等于：
```javascript
React.createElement(
  MyButton,
  {color:'blue',shadowSize:2},
  'Click me'
);
```
对于那些自闭合的标签，其实等于：
```javascript
<div className="sidebar" />
React.createElement(
  'div',
  {className:'sidebar'},
  null
);
```
根据上面的代码已经知道，JSX是一个语法糖，jsx中的标签其实是一个组件方法，一个组件方法接受props，返回React元素。所以，jsx标签不仅仅限定使用固定的变量名，它也可以是某个属性，只要这个属性是一个组件方法就是了，例如
```javascript
const MyComponents = {
  DatePicker: function DatePicker(props) {
    return <div>Imagine a {props.color} datepicker here.</div>;
  }
}

function BlueDatePicker() {
  return <MyComponents.DatePicker color="blue" />;
}
```
但是，jsx的组件标签不能是一个求值表达式，例如`<MyComponent[props.type] />`这样是不行的。如果一定要这样用，事先求值并赋值给一个变量的做法是可以的：
```javascript
function MyComponentA(props){
  var SpecificComponent = MyComponent[props.type];
  return <SpecificComponent />;
}
```
##关于props
jsx中的属性，默认为true，以下两者是相等的：
```javascript
<MyTextBox autocomplete />
<MyTextBox autocomplete={true} />
```
对于字符串字面量的属性，React都会对其转义，以下两者是相等的：
```javascript
<MyComponent message="&lt;3" />
<MyComponent message={'<3'} />
```
##关于children
如果显式指定了children，标签内部包含的内容优先级大于在属性栏指定的内容
```javascript
// props.children 等于 children2
<TestView name="caorunqi" children={'children1'}>
    children2
</TestView>
// props.children 等于 children1
<TestView name="caorunqi" children={'children1'} />
```
##函数Children
奇妙的是，可以让一个组件包裹一个函数，那么，这个组件的children就是一个函数。运用这个函数children可以做很多有趣的事：
```javascript
function Repeat(props){
  let items = [];
  for(let i=0;i<props.length;i++){
    items.push(props.children(i));
  }
  return <div>{items}</div>;
}
<Repeat length={10}>
  {(index)=><li key={index}>this is {index} item.</li>}
</Repeat>
```
##用propType监测组件属性类型
一个组件的prop是调用时候显式指定的。如果需要在定义时候指定属性的类型，对今后组件调用时传入属性的有效性进行监测，就需要运用组件的propTypes特性了。

给一个组件制定propTypes，可以为组件的属性添加约束：
```javascript
var TestView = (props)=>{
  return <div>hello {props.name}
    {props.children}
  </div>
}
```
从上面的TestView组件的定义可以看出，它需要`name`和`children`两个属性，`name`应该是个字符串，`children`是一个React元素，两者都是必填的。那么可以这样约束：
```javascript
TestView.propTypes = {
  name: React.PropTypes.string.isRequired,
  children: React.PropTypes.element.isRequired
};
```
这样，使用TestView时候如果没有传入name和children两个属性，控制台是会报错的。更加详细的属性限制写法参见：[这里](https://facebook.github.io/react/docs/typechecking-with-proptypes.html)。

##Refs
组件有一个特殊的属性，叫做ref，如果一个组件使用时定义了ref属性，分为两种情况：

- 当ref的值是一个字符串时，例如`<Hello ref="hello">`，这个ref的值就相当于一个id，组件内部中可以通过`this.refs.hello`得到这个组件的DOM
- 如果ref的值是一个函数，那么当在组件挂载和卸载时候React会调用这个函数，挂载时传入参数为这个组件的DOM，卸载时传入null.

例如：
```javascript
// 当组件加载时候，初始化了一个新的属性this.inputNode。
render(){
  return <input ref={thisNode => this.inputNode = thisNode;} />
}
```
其实用`ref="inputNode"`的形式很方便，但是官方还是推荐使用传入一个方法的方式。

