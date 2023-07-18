## 阿里云盘每日签到



> 基于 Node.js 实现的阿里云盘每日签到
>
> 说明：本文档参考青龙作者的文献，在此基础上补充一些部署流程



### TODO

- [x] 阿里云盘签到
- [x] 青龙面板支持
- [ ] 本地运行
- [ ] ~~github action 支持~~

### Use 使用 {#third-paragraph}

#### 第一步：获取 refresh_token

- 自动获取: 登录[阿里云盘](https://www.aliyundrive.com/drive/)后，控制台粘贴 `JSON.parse(localStorage.token).refresh_token`
  ![](/assets/refresh_token_1.png)

- 手动获取: 登录[阿里云盘](https://www.aliyundrive.com/drive/)后，可以在开发者工具 ->
  Application -> Local Storage 中的 `token` 字段中找到。  
  注意：不是复制整段 JSON 值，而是 JSON 里 `refresh_token` 字段的值，如下图所示红色部分：
  ![refresh token](/assets/refresh_token_2.png)

#### 第二步：安装青龙面板

**推荐使用docker安装**

```shell
# mac安装docker
brew install --cask --appdir=/Applications docker

# 拉取qinglong镜像
docker pull whyour/qinglong:latest

# 启动青龙，默认端口5600（本地端口为占用该端口）启动后访问青龙管理后台返回code401，后修改成5700就可以正常访问后台了，具体原因未知
docker run -dit \
-v $PWD/ql/config:/ql/config \
-v $PWD/ql/log:/ql/log \
-v $PWD/ql/db:/ql/db \
-p 5700:5700 \
--name qinglong \
--hostname qinglong \
--restart always \
whyour/qinglong:latest

# 查看启动的容器，状态为 healthy 代表成功
docker ps -a |grep qinglong
f7188ce50179   whyour/qinglong:latest   "./docker/docker-ent…"   8 hours ago   Up 8 hours (healthy)   0.0.0.0:5700->5700/tcp   qinglong
```



访问青龙后台地址: `localhost:5700`，根据页面引导配置用户名和密码



**青龙面板添加依赖项 axios**

![image-20230718170156610](https://cdn.jsdelivr.net/gh/opsbear2/ImagesForBlog@master/default/2023-07-18/20230718-170156.png)



#### 第三步：添加环境变量

> `CLIENT_ID` 需添加 `环境变量` 权限

| 参数          | 说明                                             |
| ------------- | ------------------------------------------------ |
| refreshToken  | 阿里云盘 refresh_token, 添加多个可支持多账户签到 |
| CLIENT_ID     | 可选项, 用于青龙面板 API 更新 refreshToken 字段  |
| CLIENT_SECRET | 可选项, 用于青龙面板 API 更新 refreshToken 字段  |

`CLIENT_ID` 和 `CLIENT_SECRET` 可在 `青龙面板 -> 系统设置 -> 应用设置 -> 新建应用` 新增, 用于自动更新环境变量内 `refreshToken` 配置

#### 第四步：添加订阅

> 添加订阅后可在定时任务列表发现新增任务, 可自行调整任务执行时间

```shell
# 命令/脚本
ql repo https://github.com/mrabit/aliyundriveDailyCheck.git "autoSignin" "" "qlApi"
```

##### 新版本:

`青龙面板 -> 订阅管理 -> 新建订阅`, 在名称输入框粘贴命令并执行

##### 旧版本:

`青龙面板 -> 定时任务 -> 新建任务` 添加命令并执行

![aliyundriveDailyCheck.png](/assets/aliyundriveDailyCheck.png)



#### 第五步：消息推送

这里选择推送媒介是【企业微信应用】

##### **企业微信创建应用**

1. 在企业微信APP中创建企业，类型可以选择【其他】，补全信息
2. 登录到企业微信后台，在【应用管理】中创建应用

![image-20230718171025721](https://cdn.jsdelivr.net/gh/opsbear2/ImagesForBlog@master/default/2023-07-18/20230718-171025.png)

3. 补全应用信息，选择成员为企业组织，上传企业LOGO

![image-20230718171144188](https://cdn.jsdelivr.net/gh/opsbear2/ImagesForBlog@master/default/2023-07-18/20230718-171144.png)



4. 设置企业可信IP，这里流程比较复制，需要先添加一个可信域名，如果你没有域名可能没法继续操作，设置可信域名之后再设置可信IP，下面是一堆截图内容

![image-20230718171802422](https://cdn.jsdelivr.net/gh/opsbear2/ImagesForBlog@master/default/2023-07-18/20230718-171802.png)



![image-20230718171909493](https://cdn.jsdelivr.net/gh/opsbear2/ImagesForBlog@master/default/2023-07-18/20230718-171909.png)

![image-20230718172108357](https://cdn.jsdelivr.net/gh/opsbear2/ImagesForBlog@master/default/2023-07-18/20230718-172108.png)

可信域名验证是指 通过这个域名可以下载该文件，我的域名是解析到github的blog的，所以在blog仓库新加这个文件即可

![image-20230718172145053](https://cdn.jsdelivr.net/gh/opsbear2/ImagesForBlog@master/default/2023-07-18/20230718-172145.png)

验证OK添加IP白名单即可

![image-20230718172404907](https://cdn.jsdelivr.net/gh/opsbear2/ImagesForBlog@master/default/2023-07-18/20230718-172404.png)



5. 统计属性信息，包括AgentId、企业ID，成员ID，用于配置消息通知

- 企业ID

![image-20230718172645463](https://cdn.jsdelivr.net/gh/opsbear2/ImagesForBlog@master/default/2023-07-18/20230718-172645.png)

- AgentId

![image-20230718172724196](https://cdn.jsdelivr.net/gh/opsbear2/ImagesForBlog@master/default/2023-07-18/20230718-172724.png)



- 成员ID

![image-20230718172849293](https://cdn.jsdelivr.net/gh/opsbear2/ImagesForBlog@master/default/2023-07-18/20230718-172849.png)



6. 安装微信插件，扫码关注后可在微信中收发企业微信的工作消息和通知 

![image-20230718192244803](https://cdn.jsdelivr.net/gh/opsbear2/ImagesForBlog@master/default/2023-07-18/20230718-192244.png)



##### **青龙面板配置推送**

1. 【系统配置】--》【通知设置】，选择【企业微信应用】

![image-20230718173100755](https://cdn.jsdelivr.net/gh/opsbear2/ImagesForBlog@master/default/2023-07-18/20230718-173100.png)

填写规则

> corpid,corpsecret,touser(注:多个成员ID使用|隔开),agentid,消息类型(选填,不填默认文本消息类型) 注意用,号隔开(英文输入法的逗号)，例如：wwcfrs,B-76WERQ,qinglong,1000001,2COat



```shell
# 参考【第五步】-【企业微信创建应用】-【5. 统计属性信息】
corpid： 企业ID
corpsecret：应用的secret
touser：成员ID
agentid：应用的agentid
消息类型：默认
```



2. 测试消息发送，点击【保存】即可触发测试，测试消息将发送到微信中

![image-20230718192531661](https://cdn.jsdelivr.net/gh/opsbear2/ImagesForBlog@master/default/2023-07-18/20230718-192531.png)



3. 修改配置文件，填写推送参数

  	说明：在【通知设置】中配置的推送参数不生效，无法推送正常阿里云敲到的通知，需要在配置文件中单独配置



**修改参数说明**

【配置文件】中找到【6. 企业微信应用】，修改`QYWX_AM`参数字符串

直接复制 【系统配置】--》【通知设置】--》【企业微信应用】--》【weWorkAppKey】，然后点击保存即可



![image-20230718192936871](https://cdn.jsdelivr.net/gh/opsbear2/ImagesForBlog@master/default/2023-07-18/20230718-192936.png)



复制红圈的参数字符串到`QYWX_AM`中

![image-20230718193221884](https://cdn.jsdelivr.net/gh/opsbear2/ImagesForBlog@master/default/2023-07-18/20230718-193221.png)

4. 测试发送签到消息

在【定时任务】中点击执行，查看日志可看到推送内容，任务的执行可以使用定时任务来配置

![image-20230718193533870](https://cdn.jsdelivr.net/gh/opsbear2/ImagesForBlog@master/default/2023-07-18/20230718-193533.png)



同事在企业微信中的应用可以查收到上面的推送信息，至此消息推送配置结束

![image-20230718193658311](https://cdn.jsdelivr.net/gh/opsbear2/ImagesForBlog@master/default/2023-07-18/20230718-193658.png)



### 申明

- 本项目仅做学习交流, 禁止用于各种非法途径
- 项目中的所有内容均源于互联网, 仅限于小范围内学习参考, 如有侵权请第一时间联系 [本项目作者](https://github.com/mrabit) 进行删除

### 鸣谢

#### 特别感谢以下作者及所开发的程序，本项目参考过以下几位开发者代码及思想。

- @Anonym-w: [Anonym-w/autoSigninAliyun](https://github.com/Anonym-w/autoSigninAliyun)
- @ImYrS: [ImYrS/aliyun-auto-signin](https://github.com/ImYrS/aliyun-auto-signin)
