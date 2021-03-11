# SSO 单点登录系统

一次登录，全站通行。

## 登录流程

先看登录流程

![process](https://wepie-fe.github.io/docs-admin/sso/static/process.png)

## 结构

SSO 单点登录系统存在 3 个页面和 1 个接口

**页面：登录页**

- host/?redirect=xxx&code=xxxx

  - Query：
    - { string }: redirect 必须
    - { string }: code 可选
  - 说明：
    - redirect 为登录后的回调地址，登录成功后将拼接 token 返回此页面，通常为管理系统的登录页面，需 encodeURIComponent
    - code 可选，为钉钉授权登录后的 code，以用来换取用户信息
  - 过程：
    1. 扫码登录后，带得到钉钉授权 code，若 query 已带钉钉授权 code 则无需扫码，会自动执行下一步。
    2. 页面将带 redirect 和 code 跳转至 SSO 单点登录系统的授权页。

**页面：认证页**

- host/auth?redirect=xxx&code=xxxx

  - Query：
    - { string }: redirect 必须
    - { string }: code 必须
  - 说明：
    - redirect 为登录后的回调地址，登录成功后将拼接 token 返回此页面，通常为管理系统的登录页面，需 encodeURIComponent
    - code 可选，为钉钉授权登录后的 code，以用来换取用户信息
  - 过程：
    1. 进入后，将通过 code 换取用户的钉钉信息，并存储至 session，并生成一个一次性 token
    2. 解析页面的 redirect，拼接此 token 并跳回。

!> 如果你有 code，虽然可以直接访问此页，但还是不建议，建议从登录页自动跳转过来。

**页面：校验页**

- host/check?redirect=xxx

  - Query：
    - { string }: redirect 必须
  - 说明：
    - redirect 为登录后的回调地址，登录成功后将拼接 token 返回此页面，通常为管理系统的登录页面，需 encodeURIComponent
  - 过程：
    1. 进入后若未登录，则跳转至 SSO 单点登录系统的登录页，走后续一系列的流程
    2. 进入后若已登录，则生成一个一次性 token，解析页面的 redirect，拼接此 token 并跳回。

**接口：获取身份信息**

- host/check?redirect=xxx

  - 参数：
    - { string }: token 必须
  - 说明：
    - token 为认证页或校验页生成的 token，一次有效
  - 返回：

    - 用户于钉钉的完整信息，完成返回示例：

      ```json
      {
        "code": 200,
        "msg": "",
        "data": {
          "active": true,
          "admin": true,
          "avatar": "xxxxxxxxxxxx",
          "boss": false,
          "dept_id_list": [1234567890],
          "dept_order_list": [{ "dept_id": 1234567890, "order": 10000000000000 }],
          "email": "xxxxxx@xxx.com",
          "exclusive_account": false,
          "extension": "{}",
          "hide_mobile": false,
          "hired_date": 1575216000000,
          "job_number": "A1",
          "leader_in_dept": [{ "dept_id": 1234567890, "leader": true }],
          "mobile": "12345678901",
          "name": "张三",
          "real_authed": true,
          "remark": "",
          "role_list": [{ "group_name": "默认", "id": 90999641, "name": "主管" }],
          "senior": false,
          "state_code": "86",
          "telephone": "",
          "title": "前端开发工程师",
          "unionid": "xxxxxxxxx",
          "userid": "xxxxxxxxx",
          "work_place": "地球"
        }
      }
      ```

## 如何对接

### 入口

入口设置为 SSO 单点登录系统的登录页，回调地址按上述文档进行 encodeURIComponent

### 前端

前端需要首次进入、刷新页面的时候，判断系统是否登录，若登录则忽略，若未登录，则跳转至 SSO 单点登录系统的校验页，各个参数如上述文档所拼接。

> 待后续更新了 Template 通用管理系统模板，将封装此功能，就无需多次开发。

### 服务端

服务端原本是获取钉钉的 code，进行换取身份等一系列信息。

现在使用 SSO 单点登录系统，则只需要将对接的对象改完 SSO 单点登录系统，而且过程简单很多，就是 token 换用户信息。

调用上面的接口：获取身份信息。

token 要么前端使用 api 调用，要么服务端从 url 的 query 获取。
