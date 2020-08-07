### JWT组成

* header(头)

    * typ: (type)类型可以自己定义, 没有限制
    * alg: (algorithm)算法,可以设置固定的几种算法

* payload(有效荷载): 存储token的具体内容, 这里的字段是一些标准字段, 也可以自己添加, 这种声明被称为claims, 下面是jwt默认的一些claims:

    * iss(Issuser): 签发主题
    * sub(Subject): 这个JWT的所有人
    * aud(Audience): 接收对象
    * exp(Expiration time): 代表这个JWT的过期时间
    * nbf(Not Before): 代表这个JWT开始生效的时间
    * iat(Issued at): 签发的时间
    * jti(JWT ID): 唯一标识

* signature(签名)

    ```shell
    signature=HS256(base64(header)+"."+base64(payload),secret_key)
    ```

    

> * 数字签名: 一般实现是, 对原始数据进行一次hash摘要, 然后使用非对称算法进行加密
> * 数字摘要/指纹: 对数据进行hash值.
> * 加密算法: 对数据进行加密, 与摘要算法最大的不同是是否可逆.

### jwt中的集中算法的支持

* HS256:  采用对称加密
* RS256: 非对称加密
* ES256: 非对称加密, 长度短一些

