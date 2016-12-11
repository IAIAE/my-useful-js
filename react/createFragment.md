#自动创建key
React中的key属性前面已经说过了，key属性用来判断当一个组件顺序改变时是否需要销毁。

有一个addon方法可以自动创建key，虽说没什么大用处，但是可以用：


```javascript
import createFragment from 'react-addons-create-fragment';

function Swapper(props){
    let children;
    children = createFragment({
        left: props.leftComponent,
        right: props.rightComponent
    });
    return (
        <div>{children}</div>    
    );
}

//以上createFragment创建出来的组件就如
[
    <LeftComponent key="left" />
    <RightComponent key="right" />
]
```



