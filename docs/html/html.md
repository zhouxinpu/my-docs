## BFC

# 标题1
***
## 标题2
### 标题3
#### 标题4
##### 标题5
###### 标题6

*斜体文字*
## **粗体文字**
~~删除线~~
<u>下划线</u>

* 列表1
* 列表2
* 列表3


1. 有序列表1
    * 列表11
    * 列表22
    * 列表33
2. 有序列表2
3. 有序列表3

矛盾[1]

这是一个连接[百度](http://www.baidu.com)

```html
<div>This is HTML</div>
```

```js
console.log('123')
```

```js
{
  data(){
    return {
      message: 'zhouxinpu'
    }
  }
}
```

<!-- 在docsify中隐藏，在其他地方显示（如GitHub）。 -->
```html
<p v-if="false">Text for GitHub</p>
```

<!-- Sequenced content (i.e. loop)-->
```html
<ul>
  <li v-for="i in 3">Item {{ i }}</li>
</ul>
```

<!-- JavaScript expressions -->
```js
<p>2 + 2 = {{ 2 + 2 }}</p>
```


<!-- 在docsify中显示消息，在其他地方显示 "{{ message }}"（例如GitHub） -->
```js
{{ message }}
```

<!-- 在docsify中显示消息，在其他地方隐藏（例如GitHub） -->
```html
<p v-text="message"></p>
```

<!-- 在docsify中显示消息，在其他地方显示 text（例如GitHub） -->
```html
<p v-text="message">Text for GitHub</p>
```

> 周信普的测试引用代码

![图片](./../dog.jpg)

~~文字~~

    title 项目开发流程
    section 项目确定
        需求分析       :a1, 2016-06-22, 3d
        可行性报告     :after a1, 5d
        概念验证       : 5d
    section 项目实施
        概要设计      :2016-07-05  , 5d
        详细设计      :2016-07-08, 10d
        编码          :2016-07-15, 10d
        测试          :2016-07-22, 5d
    section 发布验收
        发布: 2d
        验收: 3d

## 测试

    A[Hard edge] -->|Link text| B(Round edge)
    B --> C{Decision}
    C -->|One| D[Result one]
    C -->|Two| E[Result two]






[1]: 我国著名的文学作家