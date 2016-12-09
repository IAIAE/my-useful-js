#关于React的测试
FB官方推荐使用Jest配合react-addons-test-utils进行React工程的单元测试。

Jest是一个CLI工具，运行环境是node。使用方法是创建`sum.test.js`，然后命令行运行jest。`sum.test.js`语法如下：
```javascript
var sum = (a, b) => a + b;

test('test 1 plus 2 equals 3',() => {
    expect(sum(1, 2)).toBe(3);
});
```
注意node环境并不自带jsx语法和import。所以如果想在测试脚本里面写这些语法糖的话，需要引入babel.
```
npm install --save-dev babel-jest babel-polyfill
```
然后创建`.babelrc`配置babel的插件内容。最好和webpack.config.js中的一致。
```javascript
{
  "presets": ["es2015", "react"]
}
```
然后就可以运用Jest测试React组件了：
```javascript
const renderer = ReactTestUtils.createRenderer();
renderer.render(<MyComponent />);
const result = renderer.getRenderOutput();

expect(result.type).toBe('div');
expect(result.props.children).toEqual([
  <span className="heading">Title</span>,
  <Subcomponent foo="bar" />
]);
```
详细的请看[文档](https://facebook.github.io/react/docs/test-utils.html#simulate)

##一个不错的例子
下面是一个运用Jest和React-addons-test-utils测试React组件的例子，我们先测试组件显示的内容是否正确，然后模拟一个点击事件，看更改后的组件内容是否正确。
```javascript
import React from 'react'
import ReactDOM from 'react-dom'
import ReactTestUtils from 'react-addons-test-utils';
import CheckboxWithLabel from './CheckboxWithLabel.jsx'

describe('CheckboxWithLabel.test.js: ',()=>{
    const checkbox = ReactTestUtils.renderIntoDocument(
        <CheckboxWithLabel labelOn="On" labelOff="Off" />
        );
    const node = ReactDOM.findDOMNode(checkbox);

    // 首先检查这个组件的textContent是否正确
    test('component content is ok',()=>{
        expect(node.textContent).toEqual('Off');
    });
    
    //然后模拟了两次checkbox的change事件
    describe('test onchange', () => {
        var input  = ReactTestUtils.findRenderedDOMComponentWithTag(checkbox,'input');
        //第一次change，看textContent是否变为了On
        test('first onchange ok:',()=>{
            ReactTestUtils.Simulate.change(input);
            expect(node.textContent).toEqual('On');
        })
        //再change一次，看内容是否变回去了
        test('second onchange ok:',()=>{  
            ReactTestUtils.Simulate.change(input);
            expect(node.textContent).toEqual('Off');
        });
    });

});
```
注意一下从一个组件实例寻找对应dom节点的方法。要找组件对应的整个DOM，可以用`ReactDOM.findDOMNode(component_instance)`，寻找组件包含的子DOM元素用`ReactTestUtils.findRenderdDOMComponentWith(Tag|Class)`。

##类型的判断
React组件涉及到的类型有：纯DOM元素，React元素，React组件。ReactTestUtils库给出了对这三种类型的判断方法：

1、判断一个元素是不是纯DOM元素。
```javascript
const node = ReactDOM.findDOMNode(checkbox);
console.info(ReactTestUtils.isDOMComponent(node));  //true
```

2、判断一个元素是不是React元素
```javascript
var testNode=<div>hello world</div>;
console.info(ReactTestUtils.isElement(testNode));  //true
```

3、判断一个元素是不是React组件
```javascript
const checkbox = ReactTestUtils.renderIntoDocument(
    <CheckboxWithLabel labelOn="On" labelOff="Off" />
);
console.info(ReactTestUtils.isCompositeComponent(checkbox));  //true
```
