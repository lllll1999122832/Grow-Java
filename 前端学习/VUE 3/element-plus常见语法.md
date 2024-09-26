# el-row e-col

![image-20240910212620555](E:/Markdown/%E5%9B%BE%E7%89%87/image-20240910212620555.png)

![image-20240910214731879](E:/Markdown/%E5%9B%BE%E7%89%87/image-20240910214731879.png)

![image-20240910220322411](E:/Markdown/%E5%9B%BE%E7%89%87/image-20240910220322411.png)

# Router 拦截

![image-20240913210253297](E:/Markdown/%E5%9B%BE%E7%89%87/image-20240913210253297.png)

# await 作用

![image-20240913214712858](E:/Markdown/%E5%9B%BE%E7%89%87/image-20240913214712858.png)

## async和await的作用

## async

- `async`是一个用于函数前的关键字，用于声明一个异步函数。
- 异步函数意味着函数的执行不会阻塞其他代码的执行，它会立即返回一个Promise对象。
- 如果异步函数返回一个值，那么Promise会立即以该值resolve。
- 如果异步函数抛出异常，Promise会以该异常reject。

例子：

```js
async function fetchData() {
  // ...异步操作
  return data;
}
```

## await

- `await`关键字用于等待一个Promise完成，它只能在`async`函数内部使用。
- 使用`await`可以暂停当前`async`函数的执行，等待右侧的表达式（通常是一个返回Promise的函数调用）返回一个结果或者抛出异常。
- 如果等待的Promise被resolve，`await`表达式返回的是resolve的值。
- 如果Promise被reject，`await`会抛出一个错误，通常需要用try…catch语句来捕获处理。

例子：

```js
async function getData() {
  try {
    const response = await fetch('url'); // 等待fetch请求完成
    const data = await response.json();  // 等待将响应转换为JSON
    return data;
  } catch (error) {
    console.error('Error fetching data: ', error);
  }
}
```

![image-20240919215112686](E:/Markdown/%E5%9B%BE%E7%89%87/image-20240919215112686.png)