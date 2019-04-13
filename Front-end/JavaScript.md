# JavaScript
## 指定 `Array` 的元素类型
不好意思做不到！js 是弱类型语言，我们不能真正的做到指定元素类型，但是可以先指定元素长度，然后再循环赋值哈哈哈～

## `HttpOnly`
产品上出现了一个奇怪的问题，一个 H5 页面在测试环境下（hostName 为 xxx-test.xxx.com）完全没有任何问题，`Cookie` 可以取到所有信息，但是发布到线上后，发现了一个十分捉鸡的问题，线上环境（xxx.xxx.com）居然取不到 `Cookie` 中的 `xxx_ticket` 信息！！！

刚开始吭哧吭哧的整了一波后，始终在怀疑是自己写的逻辑有问题，查来查去，还是没啥头绪，没办法只能去请教下前辈。一波操作后，发现了如下情况：

线上环境：
![httpOnly.png](https://i.loli.net/2018/12/17/5c172f77cb08f.png)

测试环境：
![ticket.png](https://i.loli.net/2018/12/17/5c172f9c3382b.png)

原因就在这个 `HttpOnly` 上，关于 `HttpOnly` 的解释如下：
>>> HttpOnly: HttpOnly is an additional flag included in a Set-Cookie HTTP response header. Using the HttpOnly flag when generating a cookie helps mitigate the risk of client side script accessing the protected cookie (if the browser supports it).

换句话说，客户端/浏览器是无法对 `Cookie` 中的这个 `key` 进行操作的，并且不可见。

## 连接两个数组
不可使用 `+=`，而是 `concat`。