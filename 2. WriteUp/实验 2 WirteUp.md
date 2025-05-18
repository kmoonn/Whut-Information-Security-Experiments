对 Pikachu 靶场弱口令漏洞的暴力破解

1. 打开靶场页面
	![image.png](https://image.kmoon.fun/2025/202505181728415.png)
2. 输入账号、密码点击登录，抓取登录数据包。
	前提要按照实验报告书要求，配置好浏览器代理和 burpsuite 工具。
	开启拦截：
	![image.png](https://image.kmoon.fun/2025/202505181730565.png)
	 抓取到登录请求的数据包：
	 其中 username 为用户输入的用户名，password 为用户输入的密码。
	 ![image.png](https://image.kmoon.fun/2025/202505181732382.png)
	 查看该请求的响应：
	 ![image.png](https://image.kmoon.fun/2025/202505181735334.png)

	将该数据包发送到 **Intruder** 攻击模块
	![image.png](https://image.kmoon.fun/2025/202505181734682.png)
	添加payload 的位置，我们将 password 标记为payload的位置，因为是在已知账号的情况下爆破密码
	![image.png](https://image.kmoon.fun/2025/202505181737362.png)
	点击 Load 加载弱口令密码字典，点击开始攻击
	![image.png](https://image.kmoon.fun/2025/202505181739452.png)
	开始爆破，通过查看响应体的长度可以判断哪个请求是不同于其他请求的，一般登录成功的响应数据包都较短。
	![image.png](https://image.kmoon.fun/2025/202505181742640.png)
	我们发现 payload 等于 123456 时，响应数据包的长度最小，我们查看该请求的响应结果。
	![image.png](https://image.kmoon.fun/2025/202505181744484.png)
	登录成功，说明 admin 用户的密码为 123456。