# Http 认证

* 发送无认证信息请求
  * Http status  200/404
    * 无认证
  * http status 非401
    * 请求出错
  * http status  401
    * 服务端发起质询
      * 客户端解析质询，获取认证方式，realm等信息
        * 认证方式Basic
          * 客户端发起认证请求，-H ‘auth’
        * 认证方式Bearer
          * 客户端根据质询中提供的token url.请求url 获取token
            * http status 非2xx
              * 获取token失败，无权限
            * http status 200
              * 取得token，客户端发起资源认证请求 
                * http status 非2xx
                  * 无权限或无资源等
                * http status 200 
                  * 资源认证请求成功
* 如果提供了用户名密码
  * 第一次尝试basic 
    * 返回200 说明是basic 方式
    * 返回401 
      * 用户名密码不正确
        * 退出
      * 认证方式不正确
        * 尝试bearer 认证方式
          * 获取token url
          * 获取token
          * 请求资源

​    