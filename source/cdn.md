---
title: CDN
---
# CDN

CDN （Content Delivery Network）内容发布网络，其目的是通过在现有的 Internet 中增加一层新的网络架构 CACHE (缓存)层，将网站的
内容发布到最接近用户的网络‘边缘‘的节点，使用户可以就近取得所需的内容，提高用户访问网站的响应速度。从技术上解决由于网络带宽小、用户
访问量大、网点分布不均等原因，提高用户访问网站的响应速度。


## 工作原理
CDN 的最基本的原理：CDN 网络是在用户和服务器之间增加 Cache 层，通过接管 DNS 实现将用户的请求引导到 Cache 上获得源服务器的数据。

### 传统访问过程

1. 用户向浏览器提供要访问的域名。
2. 浏览器调用域名解析函数库对域名进行解析，以得到此域名对应的 IP 地址。
3. 浏览器使用所得到的 IP 地址，域名的服务主机发出数据访问请求。
4. 浏览器根据域名主机返回的数据显示网页的内容。

### CDN 访问过程
对于 CDN 客户，不需要改动网站架构，只需要修改自己的 DNS 解析，设置一个 CNAME 指向 CDN 服务商。

1. 用户向浏览器提供要访问的域名。
2. 浏览器调用域名解析库对域名进行解析，由于 CDN 对域名解析过程进行了调整，所以解析函数库得到的是该域名对应的 CNAME 记录（由于现在已
经是使用了 CDN 服务，CNAME 为 CDN 服务商域名），为了得到实际 IP 地址，浏览器需要再次对获得的 CNAME 域名进行解析以得到实际的 IP 地址；
在此过程中，使用的全局负载均衡 DNS 解析，如根据地理位置信息解析最适合的CDN节点ip地址。
3. 此次解析得到 CDN 缓存服务器的 IP 地址，浏览器在得到 IP 地址以后，向缓存服务器发出访问请求。
4. 缓存服务器根据浏览器提供的要访问的域名，通过 Cache 内部专用 DNS 解析得到此域名的实际 IP 地址，再由缓存服务器向此实际 IP 地址提交访
问请求。
5. 缓存服务器从实际 IP 地址得得到内容以后，一方面在本地进行保存，以备以后使用，二方面把获取的数据返回给客户端，完成数据服务过程。
6. 客户端得到由缓存服务器返回的数据以后显示出来并完成整个浏览的数据请求过程。