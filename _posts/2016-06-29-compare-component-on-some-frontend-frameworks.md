---
layout: post
title:  "前端框架组件化比较"
date:   2016-06-29 11:40:21
categories: frontend component
---

第一次接触到前端组件的概念是在学习Extjs的时候，有一个名叫Ext.Component的对象，所有的组件都是继承自这个对象。Extjs有很多自带的组件，种类齐全，可以应对大部分后台页面的需求，而且实现了页面全部由组件组成的宏伟目标。组件的编写几乎都是靠JS，而弱化了HTML和CSS。虽然现在已经不用Extjs了但有机会一定要好好研究下它对组件概念的理解。

花了近一天的时间，用我认为比较主流的前端框架分别实现了同一个简单的功能，看图就能明白这个功能，这里也只比较简单的单层组件的实现。

![](https://raw.githubusercontent.com/xxapp/xxapp.github.io/master/assests/component-prototype.png)

当然几种框架的实现都使用了组件化的方式，并实例化三次，所以你看到图上有三行且每行的初始文字不同。接下来我将分别展示Angular2、Vue、React和Molecule的组件实现。注：[Molecule]不是流行组件化框架，但是用与众不同的方式实现组件。

## Angular2

``` typescript
import { Component, Input } from '@angular/core';

@Component({
    selector: 'amazing-angular2',
    template: `<input [(ngModel)]="text">
            <button (click)="changeToUpperCase()">Upper Case</button>`,
    styles: [``]
})
export class ItemComponent {
    @Input()
    text = 'Hello Wolrd';

    changeToUpperCase() {
        this.text = this.text.toUpperCase();
    }
};
```
Angular2用ES7的装饰器特性将一个类标记为组件。

* template属性用HTML定义了组件的结构
* styles属性用CSS定义了组件的样式（Scoped CSS，即这里写的样式只作用于此组件）
* 组件的逻辑实现和Angular1类似，在类中声明成员来与HTML结构绑定
* Input装饰器让组件接收外部参数，这里外部的text参数会覆盖text成员的原始值，缺省则为text成员的原始值
* 在HTML结构中通过selector指定的标签来实例化组件，`<amazing-angular2 [text]="'Hello Angular2!'">Loading...</amazing-angular2>`

## Vue

``` javascript
var Vue = require('vue');

Vue.component('amazing-vue', {
    props: ['text'],
    template: '<div><input v-model="text"><button @click="changeToUpperCase">Upper Case</button></div>',
    data: function () {
        return { };
    },
    methods: {
        changeToUpperCase: function () {
            this.text = this.text.toUpperCase();
        }
    }
});
```
这里用了Vue的语法糖把组件定义和注册合成了一步。

* 在template属性中定义组件的HTML结构
* 奇怪的是没有找到定义组件样式的API，可能是我没找到。但在Vue单文件组件中是实现Scoped CSS
* 与其它MVVM框架一样，定义数据属性和操作属性与HTML结构绑定，实现各种业务逻辑，避免了繁琐的DOM操作
* props属性定义了组件可以接受的外部参数的数组，如果传入外部参数，则可以直接当做数据属性来使用
* 组件的实例化可以通过标签来实现，`<amazing-vue text="Hello Vue!"></amazing-vue>`

## React

``` jsx
var AmazingReact = React.createClass({
    getInitialState: function () {
        return {
            text: this.props.text
        };
    },
    changeToUpperCase: function () {
        this.setState({
            text: this.state.text.toUpperCase()
        });
    },
    render: function () {
        return (
            <div>
                <input type="text" value={this.state.text} />
                <button onClick={this.changeToUpperCase}>Upper Case</button>
            </div>
        );
    }
});
```
creatClass创建组件，由于采用了单向数据流，所以与Angular和Vue的实现有一些差别。

* 在render中用jsx的语法定义组件的HTML结构
* React似乎也没有实现组件内部样式
* getInitialState初始化内部state，自定义方法操作state，state改变会导致重新渲染，以此来实现业务逻辑
* 外部参数可直接通过this.props获取
* 通过react-dom的render方法以标签的形式实例化组件，`<AmazingReact text="Hello React!"/>`

## Molecule

``` html
<div molecule-def="AmazingMolecule">
    <input value="Hello World"><button id="bnUpperCase">Upper Case</button>
    <script>
        // MOLECULE_DEF
        function AmazingMolecule(){
            this.$el.find('input').val(this.$el.data('text'));
            this.$el.find('#bnUpperCase').click(function(){
                var $tx = $(this).parent().find('input');
                $tx.val($tx.val().toUpperCase());
            })
        }
        // MOLECULE_DEF_END
        Molecule.create(AmazingMolecule);
    </script>
</div>
```
[Molecule]采用了基于HTML定义组件的方式。

* 由于是HTML主场，所以组件HTML结构直接在在里面
* 猜你一定想到了，组件的样式可以写在style标签内。不过Molecule目前还未实现Scoped CSS，所以定义样式要多加小心
* JS到了客场也不得不包一层script标签，使用jquery实现业务逻辑
* 外部参数也用jquery来获取
* 在HTML中通过标签属性来实例化组件，`<div molecule="AmazingMolecule" data-text="Hello Molecule!"></div>`

## 总结

可以看出虽然各个框架实现组件的方式各不相同，但总体思想是一致的，内部都包含组件`结构`和组件`行为`的定义，外部则都提供了`接口`实现参数传递。至于有些框架没有实现内部的CSS，我猜应该是考虑了`表现`的多样性。而`实例化`的方式则都采用了HTML标签的方式，利用HTML的树形结构，让多个组件的组织更加方便和一目了然。

[Molecule]:      https://github.com/inshua/d2js/blob/master/WebContent/guide/molecule.md
