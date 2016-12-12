#ReactTransitionGroup源码分析
首先看ReactCSSTransitionGroup的使用方法：

```javascript
<ReactCSSTransitionGroup
    transitionName={{
        enter:'itemEnter',
        leave:'itemLeave',
        appear:'itemAppear',
        enterActive:'itemEnterActive',
        leaveActive:'itemLeaveActive',
        appearActive:'itemAppearActive'
    }}
    transitionAppear={true}
    transitionAppearTimeout={500}
    transitionEnterTimeout={200}
    transitionLeaveTimeout={200}>
    {items}
</ReactCSSTransitionGroup>
```
##源码分析
ReactCSSTransitionGroup基于ReactTransitionGroup。我们首先分析ReactCSSTransitionGroup的代码，然后再深入了解ReactTransitionGroup。
我们先创建一个空的组件，命名为ReactCSSTransitionGroup。

```javascript
import {Component} from 'react';

class ReactCSSTransitionGroup extends Component{
    render(){
        return (
            <div></div>
        )
    }
}
```
接下来我们来完善这个组件。
###prop类型验证
ReactCSSTransitionGroup的prop属性包括

- `transitionName`
- `transitionEnter`
- `transitionEnterTimeout`
- `transitionLeave`
- `transitionLeaveTimeout`
- `transitionAppear`
- `transitionAppearTimeout`

七个。其中`transition(Enter|Appear|Leave)`是一个布尔值，用于标识是否开启这个过度动画。`transition(Enter|Appear|Leave)Timeout`是一个数值，用于指定对应过渡动画的总时长。如果`transition(Enter|Appear|Leave)`等于true，那么对应的`transition(Enter|Appear|Leave)Timeout`就必须存在并且类型为`number`。根据以上论述，ReactCSSTransitionGroup组件的propTypes应该为：

```javascript
ReactCSSTransitionGroup.propTypes = {
    transitionName: (prop)=>undefined/* 先搁置这部分 */,
    transitionAppear: React.PropTypes.bool,
    transitionEnter: React.PropTypes.bool,
    transitionLeave: React.PropTypes.bool,
    transitionAppearTimeout: createTransitionTimeoutPropValidator('Appear'),
    transitionEnterTimeout: createTransitionTimeoutPropValidator('Enter'),
    transitionLeaveTimeout: createTransitionTimeoutPropValidator('Leave')
}
```
`createTransitionTimeoutPropValidator`方法用于根据过渡类型生成验证函数，其代码为：
```javascript
var createTransitionTimeoutPropValidator = type => {
    var transitionName = `transition${type}`,
        transitionTimeoutName = transitionName+'Timeout';
    return (prop) => {
        // 当过渡为true时
        if(prop[transitionName]){
            //如果对应timeout不存在，验证不通过
            if(prop[transitionTimeoutName] == null){
                return new Error('...');
            //如果timeout不为数值类型，验证不通过
            }else if(typeof prop[transitionTimeoutName] !== 'number'){
                return new Error('...');
            }
        }
    }
}
```
现在来看`transitionName`属性的类型。`transitionName`可以是一个字符串

```javascript
transitionName={'todoListTransition'}
```
也可以是一个对象，如果是对象的话，必须包含以下六个属性。
```javascript
transitionName={{
   enter:'itemEnter',
   leave:'itemLeave',
   appear:'itemAppear',
   enterActive:'itemEnterActive',
   leaveActive:'itemLeaveActive',
   appearActive:'itemAppearActive'
}}
```
所以transitionName的属性值限制为：

```javascript
transitionName: React.PropTypes.oneOfType([
    React.PropTypes.string,
    React.PropTypes.shape({
        enter: React.PropTypes.string,
        leave: React.PropTypes.string,
        appear: React.PropTypes.string,
        enterActive: React.PropTypes.string,
        leaveActive: React.PropTypes.string,
        appearActive: React.PropTypes.string
    })
])
```
###过渡实现原理
ReactTransitionGroup会在children数量发生变化时候调用对应的钩子方法。在下面的例子中：

```javascript
<ReactTransitionGroup>
    <MyListItem />
</ReactTransitionGroup>
```
初次加载的时候ReactTransitionGroup会调用MyListItem的`componentWillAppear`方法。当第一个MyListItem组件被移除的时候，会调用这个组件的`componentWillEnter`方法。所以，运用此原理，只要给每个列表元素包装一个组件用于接受这些钩子方法，就可以知道何时对列表元素运用过度效果了。例如

假设一个原始的组件是这样的：
```javascript
<ReactCSSTransitionGroup /*省略属性配置*/>
    <span>first one</span>
    <span>second one</span>
    <span>the third</span>
</ReactCSSTransitionGroup>
```
其实，它最终会被渲染成这样：
```javascript
<ReactTransitionGroup>
    <ReactCSSTransitionGroupChild>
        <span>first one</span>
    </ReactCSSTransitionGroupChild>
    <ReactCSSTransitionGroupChild>
        <span>second one</span>
    </ReactCSSTransitionGroupChild>
    <ReactCSSTransitionGroupChild>
        <span>the third</span>
    </ReactCSSTransitionGroupChild>
</ReactTransitionGroup>
```
所以，当有列表元素添加或删除的时候，其实是ReactCSSTransitionGroupChild组件接收到钩子函数的方法。比如，当添加了第四个列表元素时，实际上时添加了

```javascript
<ReactCSSTransitionGroupChild>
    <span>the forth</span>
</ReactCSSTransitionGroupChild>
```
ReactCSSTransitionGroupChild组件接收到`componentWillEnter`方法调用，给`<span>the forth</span>`添加一个class，变成了

```javascript
<span class="item-enter">the forth</span>
```
然后又在下一个帧添加另一个class：
```javascript
<span class="item-enter item-enter-active">the forth</span>
```
这时过度动画就产生了！！然后在timeout的时间后，ReactCSSTransitionGroupChild组件负责消除`item-enter`和`item-enter-active`这两个类名。

###实现
我们需要两个组件来实现以上功能。第一个是ReactCSSTransitionGroup组件，什么也不做，只是一个外部封装，并且告诉ReactTransitionGroup用ReactCSSTransitionGroupChild来包装每个元素。

```javascript
class ReactCSSTransitionGroup extends Component{
    _wrapChild(child){
        return <ReactCSSTransitionGroupChild>
            {child}
       </ReactCSSTransitionGroupChild>
    }
    render(){
        // 封装函数用childFactory传递给ReactTransitionGroup
        return <ReactTransitionGroup {...this.props} childFactory={this._wrapChild}/>
    }
}
```
然后就实现ReactCSSTransitionGroupChild组件，实现三个钩子方法：`componentWillEnter`、`componentWillLeave`、`componentWillAppear`。

```javascript
class ReactCSSTransitionGroupChild extends Component{
    componentWillEnter(done){
        // 设置并消除enter的过渡
        done();
    }
    componentWillLeave(done){
        // 设置并消除leave的过渡
        done();
    }
    componentWillAppear(done){
        // 设置并消除appear的过渡
        done();
    }
    render(){
        return this.props.children;
    }
}
```
整个组件的实现方法就是大体如此。



