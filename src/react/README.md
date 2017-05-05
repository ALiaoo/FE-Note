# 使用context传递数据
1.组件间跨级传递数据  
2.子组件获取父组件的属性值

## 1.背景
update react-element checkbox时，新增需求是对CheckboxGroup进行可选项目数的控制，使用min和max两个属性来限制可以被勾选的项目数量。实现方式是当点击CheckboxGroup中的某个Checkbox时，在onChange事件中判断当前已勾选item的数量是否在min和max的范围内，来控制当前Checkbox的checked属性。所以在子元素checkbox中需要获取到父元素CheckboxGroup的属性min和max值。

## 2.使用

### 2.1 在顶层元素中定义和赋值
```jsx
class CheckboxGroup extends React.Component {
  //为context对象的属性赋值
  getChildContext() {
    return {
      group: this
    }
  }
}
//定义
CheckboxGroup.childContextTypes = {
  group: React.PropTypes.any
}

CheckboxGroup.propTypes = {
  min: React.PropTypes.number,
  max: React.PropTypes.number
}
```

### 2.2 在子组件中声明contextTypes及使用context获取数据
```jsx
class CheckBox extends React.Component {
  onChange(e) {
    //通过this.context来调用
    const group = this.context.group;
    const min = group.props.min;
    const max = group.props.max;
  }
}
//声明contextTypes,就可以通过组件实例的context来访问数据
Checkbox.contextTypes = {
  group: React.PropTypes.any
}
```

## 3. 总结
*   该例中组件开发，使用context来进行数据传递更为直接方便。
*   React的三个数据载体：state，props，context。state用来更新内部状态。props与context用在组件间传递数据。当组件层级多，不想使用props属性自上而下一级一级传递，考虑使用context进行跨级传递数据。
*   注意context的使用场景。context的使用会降低组件的复用性。

## 4. 参考
[React的数据载体](https://zhuanlan.zhihu.com/p/24655661)
[React中的Context](http://blog.csdn.net/liwusen/article/details/53408906)
