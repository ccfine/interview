# 面试记录、总结

## 编程题

### 题目（字节跳动）

函数a，当偶数次调用时输出1，奇数次调用时输出2

#### 代码

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


### 题目（字节跳动）

防抖：触发一个事件后，n秒后才会执行，如果在n秒内又触发了这个事件，就以新的事件的时间为准，n秒后执行

#### 代码

    function debounce (event, time) {
      let timer = null;
      return function (...args) {
        clearTimeout(timer);
        timer = setTimeout(() => {
          timer = null;
          event.apply(this, args);
        }, time);
      }
    }


### 题目（字节跳动）

节流：不管频率多高，单位时间内只执行一次

#### 代码

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


### 题目（字节跳动）

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

#### 参考

3 6 9

当调用foo(1)时，a的值为1，result数组里面放入了3个元素，且为函数，如result[0] = function () { total += i * 1; console.log(total) }，函数只是被定义，并未被执行。当调用`result[0]`时，循环体早已结束，i的值变为了3，total = 0 + 3 * 1，且total为全局变量，会被修改，故输出3，6，9。函数只有在被调用时才会执行内部的函数体。


### 题目

手动实现call

#### 代码

    Function.prototype.call = function (context, ...args) {
      if (this === Function.prototype) {
        return undefined;
      }
      context = context || window;
      const fn = Symbol();
      context[fn] = this;
      const result = context[fn](...args);
      delete context[fn];
      return result;
    }


### 题目

手动实现apply

#### 代码

    Function.prototype.apply = function (context, ...args) {
      if (this === Function.prototype) {
        return undefined;
      }
      context = context || window;
      const fn = Symbol();
      context[fn] = this;
      let result;
      if (Array.isArray(args)) {
        result = context[fn](...args);
      } else {
        result = context[fn]();
      }
      delete context[fn];
      return result;
    }


### 题目

手动实现bind

#### 代码

    Function.prototype.bind = function (context, ...args1) {
      if (this === Function.prototype) {
        throw new TypeError("error");
      }
      const _this = this;
      return function F (...args2) {
        if (this instanceof F) {
          return new _this(...args1, ...args2);
        } else {
          return _this.apply(context, args1.concat(args2));
        }
      }
    }


### 题目

手动实现promise

#### 代码

    function Promise (executor) {
      this.state = "pending";
      this.value = null;
      this.reason = null;
      this.onFulfilledCallbacks = [];
      this.onRejectedCallbacks = [];

      const resovle = value => {
        if (this.state === "pending") {
          this.state = "fulfilled";
          this.value = value;
          this.onFulfilledCallbacks.forEach(func => {
            func();
          });
        }
      }

      const reject = reason => {
        if (this.state === "pending") {
          this.state = "rejected";
          this.reason = reason;
          this.onRejectedCallbacks.forEach(func => {
            func();
          })
        }
      }

      try {
        executor(resovle, reject);
      } catch (reason) {
        reject(reason);
      }
    }

    Promise.prototype.then = function (onFulfilled, onRejected) {
      if (typeof onFulfilled !== "function") {
        onFulfilled = function (value) {
          return value;
        }
      }
      if (typeof onRejected !== "function") {
        onRejected = function (reason) {
          throw reason;
        }
      }
      return new Promise((resolve, reject) => {
        switch (this.state) {
          case "fulfilled":
            setTimeout(() => {
              try {
                const result = onFulfilled(this.value);
                resolve(result);
              } catch (reason) {
                reject(reason);
              }
            }, 0);
            break;
          case "rejected":
            setTimeout(() => {
              try {
                const result = onRejected(this.reason);
                resolve(result);
              } catch (reason) {
                reject(reason);
              }
            }, 0);
            break;
          case "pending":
            this.onFulfilledCallbacks.push(() => {
              setTimeout(() => {
                try {
                  const result = onFulfilled(this.value);
                  resolve(result);
                } catch (reason) {
                  reject(reason);
                }
              }, 0);
            });
            this.onRejectedCallbacks.push(() => {
              setTimeout(() => {
                try {
                  const result = onRejected(this.reason);
                  resolve(result);
                } catch (reason) {
                  reject(reason)
                }
              }, 0);
            });
            break;
        }
      });
    }

    Promise.prototype.catch = function (onRejected) {
      return this.then(null, onRejected);
    }

    Promise.resolve = function (value) {
      return new Promise((resolve, reject) => {
        resolve(value);
      });
    }

    Promise.reject = function (reason) {
      return new Promise((resolve, reject) => {
        reject(reason);
      });
    }

    Promise.all = function (promises) {
      return new Promise ((resolve, reject) => {
        if (promises.length === 0) {
          resolve([]);
        } else {
          let result = [];
          let index = 0;
          for (let i = 0; i < promises.length; i++) {
            promise[i].then(data => {
              result[i] = data;
              index++;
              if (index === promises.length) {
                resolve(result);
              }
            }, err => {
              reject(err);
              return;
            });
          }
        }
      });
    }

    Promise.race = function (promises) {
      return new Promise((resolve, reject) => {
        if (promises.length === 0) {
          resolve();
        } else {
          for (let i = 0; i < promises.length; i++) {
            promises[i].then(data => {
              resolve(data);
            }, err => {
              reject(err);
              return;
            });
          }
        }
      });
    }


### 题目

random7()函数返回1-7随机整数，用random7()构造random10()，返回1-10随机整数

#### 代码

    function random10 () {
      let i;
      do {
        i = (random7() - 1) * 7 + random7();
      }
      while (i > 40);
      return i % 10 + 1;
    }


### 题目（腾讯）

ipv4地址（数字 . 空格组成）转化为32位整数

#### 代码

    function transIpv4 (ipv4) {
      ipv4 = ipv4.replace(/\s/g, ".").split(".");
      let ipv4Str = "";
      for (let i = 0; i < ipv4.length; i++) {
        let item = "";
        let itemPrefix = "";
        let itemStr = Number(ipv4[i]).toString(2);
        for (j = 0; j < 8 - itemStr.length; j++) {
          itemPrefix += "0";
        }
        item = itemPrefix + itemStr;
        ipv4Str += item;
      }
      return parseInt(ipv4Str, 2);
    }


### 题目（腾讯）

给定两个大小为 m 和 n 的有序数组 nums1 和 nums2。请你找出这两个有序数组的中位数，并且要求算法的时间复杂度为 O(log(m + n))。

#### 代码

    function findMedianSortedArrays (nums1, nums2) {
      var m = nums1.length;
      var n = nums2.length;
      var l = Math.floor((m + n + 1) / 2);
      var r = Math.floor((m + n + 2) / 2);
      return (getMid(nums1, 0 , nums2, 0, l) + getMid(nums1, 0, nums2, 0, r)) / 2;
    }

    function getMid (nums1, i, nums2, j, k) {
      if (i >= nums1.length) {
        return nums2[j + k - 1];
      }
      if (j >= nums2.length) {
        return nums1[i + k - 1];
      }
      if (k === 1) {
        return Math.min(nums1[i], nums2[j]);
      }
      var nums1Mid = i + Math.floor(k / 2) - 1 < nums1.length ? nums1[i + Math.floor(k / 2) - 1] : Infinity;
      var nums2Mid = j + Math.floor(k / 2) - 1 < nums2.length ? nums2[j + Math.floor(k / 2) - 1] : Infinity;
      if (nums1Mid < nums2Mid) {
        return getMid(nums1, i + Math.floor(k / 2), nums2, j, k - Math.floor(k / 2));
      } else {
        return getMid(nums1, i, nums2, j + Math.floor(k / 2), k - Math.floor(k / 2)); 
      }
    }

#### [参考](https://leetcode-cn.com/problems/median-of-two-sorted-arrays/solution/man-hua-ru-guo-qi-ta-de-ti-jie-ni-kan-bu-dong-na-j/)


### 题目（腾讯）

实现一个方法decodeStr，输入一个字符串，根据约定规则输出编码结果。约定规则如下：str = "2[a]1[bc]", 返回 "aabc"；str = "2[e2[d]]", 返回 "eddedd"；str = "3[abc]2[cd]ff", 返回 "abcabcabccdcdff"

#### 代码

    function decodeStr (str) {
      const left = str.lastIndexOf("[");
      const right = str.indexOf("]", left);
      if (left === -1 && right === -1) {
        return str;
      }
      const repeat = !isNaN(str[left - 1]) ? str[left - 1] * 1 : 0;
      let repeatStr = "";
      let newStr = "";
      for (let i = repeat; i > 0; i--) {
        repeatStr += str.slice(left + 1, right);
      }
      newStr = str.slice(0, left - 1) + repeatStr + str.slice(right + 1);
      decodeStr(newStr);
    }


### 题目（腾讯）

    function F () {}
    F.a = 100;
    F.prototype.b = 200;
    var z = new F();
    z.a;
    z.b;

#### 参考

undefined 200


### 题目（腾讯）

    function fun (n, k) {
      console.log(k);
      return {
        fun: function (m) {
          return fun(m, n);
        }
      };
    }

    var a = fun(0);
    a.fun(1);
    a.fun(2);
    a.fun(3);
    var b = fun(0).fun(1).fun(2).fun(3);

#### 参考

undefined 0 0 0 undefined 0 1 2

值来自于上一次的传参


## 概念题

### 题目

js异步加载的方式

#### 参考

* 动态脚本加载 document.createElement("script")  
* defer `<script defer />`  
* async `<scrpt async />`


### 题目

opacity:0、visibility: hidden、display:none的区别

#### 参考

opacity:0 元素不可见，但仍然在页面上，绑定的事件依然会触发  
visibility: hidden 元素不可见，但仍然在页面上，绑定的事件不会触发  
display:none 元素不可见，页面上已消失


### 题目（腾讯）

浏览器是多线程还是单线程

#### 参考

浏览器是多线程，分别有：
* js引擎线程（多线程，但有一个主线程）
* ui渲染线程（js可操作dom，所以js引擎线程与ui渲染线程互斥）
* 事件触发线程
* http请求线程
* 定时触发器线程
* 事件轮询处理线程