# 面试记录、总结

## 编程题（字节跳动）
### 题目

函数a，当偶数次调用时输出1，奇数次调用时输出2

### 示列

    function a () {
      let count = 1;
      return function () {
        if (count % 2 === 0) {
          console.log(1);
        } else {
          console.log(2);
        }
        count++;
      }
    }


## 防抖与节流（字节跳动）
### 防抖

触发一个事件后，n秒后才会执行，如果在n秒内又触发了这个事件，就以新的事件的时间为准，n秒后执行

#### 示列

    function debounce (event, time) {
      let timer = null;
      return function (...args) {
        clearTimeout(timer);
        timer = setTimeout(() => {
          event.apply(this, args);
        }, time);
      }
    }

### 节流

不管频率多高，单位时间内只执行一次

#### 示列

    function throttle (event, time) {
      let pre = 0;
      return function (...args) {
        if (Date.now() - pre > time) {
          pre = Date.now();
          event.apply(this, args);
        }
      }
    }

    function throttle (event, time) {
      let timer = null;
      return function (...args) {
        if (!timer) {
          timer = setTimeout(() => {
            timer = null;
            event.apply(this, args);
          }, time);
        }
      }
    }


## 编程题（字节跳动）
### 题目

    var total = 0;
    var a = 3;
    var result = [];
    function foo (a) {
      for (var i = 0; i < 3; i++) {
        result[i] = function () {
          total += i * a;
          console.log(total)
        }
      }
    }

    foo(1);
    result[0]();
    result[1]();
    result[2]();

### 结果

3 6 9

### 分析

当调用foo(1)时，a的值为1，result数组里面放入了3个元素，且为函数，如result[0] = function () { total += i * 1; console.log(total) }，函数只是被定义，并未被执行。当调用`result[0]`时，循环体早已结束，i的值变为了3，total = 0 + 3 * 1，且total为全局变量，会被修改，故输出3，6，9。函数只有在被调用时才会执行内部的函数体。


## 概念题
### 题目

js异步加载的方式

### 参考

1）动态脚本加载 document.createElement("script")  
2）defer `<script defer />`  
3）async `<scrpt async />`


## 概念题
### 题目

opacity:0、visibility: hidden、display:none的区别

### 参考

opacity:0 元素不可见，但仍然在页面上，绑定的事件依然会触发  
visibility: hidden 元素不可见，但仍然在页面上，绑定的事件不会触发  
display:none 元素不可见，页面上已消失