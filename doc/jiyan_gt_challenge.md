### 项目介绍

数据采集的过程中难免会遇到一些验证码需要解决，这里介绍一下我遇到jiyan平台的验证码是如何进行分析的。

**声明**: 本项目仅供学习交流使用，严禁用于商业和违法行为，否则产生的一切后果与本人无关。

利益相关，无法开放源码，为了方便说明，这里使用一些文字代替是必要的。

内容比较草率，不过我相信真正研究过的人可以看懂，伸手党估计不合适。

**如果对您有用，请给项目一个star，谢谢**

> 感兴趣或者看不懂可以加项目首页的微信交流，记得备注

### 参数分析

jiyan的关键参数是`gt`和`challenge`

1. 获取第一段加密JS和uid参数，解密后获得新的JS代码

   ```python
   s1 = re.findall('var x="(.*?)"', first_js)[0]
   s2 = re.findall(',y="(.*?)"', first_js)[0]
   second_js = self.wc_js.call('first_se_js', s1, s2)
   ```

2. 新的JS运行后生成的`__jsl_clearance`参数和第一步的uid，请求得到JSESSIONID

   这一步关键的两个参数是`__jsluid_h`和`__jsl_clearance`

   在cookie中携带请求，可获得JSESSIONID

   ```python
   headers['Cookie'] = '__jsluid_h=__jsluid_h; __jsl_clearance=__jsl_clearance;'
   ```

3. 此时我们有了三个关键参数，携带三个参数去请求验证码接口，获取新的JSESSIONID

   ```python
   headers['Cookie'] = '__jsluid_h=__jsluid_h; __jsl_clearance=jsl_clearance;JSESSIONID=JSESSIONID'
   ```

4. 这时候有了所有的参数了，再携带访问jiyan的接口，即可获得`gt`和`challenge`

5. 根据`gt`和`challenge`分析验证码类型，进行相应的处理，这里可以通过统一接口进行处理，也可以对`滑动验证`、`点选验证码`进行分别处理（这部分内容涉及比较广，新手可以使用打码平台，或者像我一样通过机器学习，产生`轨迹数据`以及训练`识别模型`）