# 操作步骤

## 1 取AlexaSetup的代码

```
git clone ssh://hongmei.kuang@scgit.amlogic.com:29418/avs/AlexaSetup
```

## 2 修改信息

AlexaSetup/alexa_setup_service/settoken.cpp这个文件里边配置的product的信息

在这个文件中需要修改的地方有两个，

1）、clientSecret 修改成新绑定账户的clientSeret;

2)、在新绑定账户的Security Profile中新建一个profile （如果有就不用新建）

然后在**Other devices and platforms** 中ADD ANOTHER ，输入Client ID name(名字自取)，会生成一个随机的Client ID。

**注意**：这一步操作在要amazon developer账户登录开发账户操作。

3）、将步骤2生成的client ID修改到settoken.cpp文件中

## 3 编译生成新的cgi文件

修改AlexaSetup/alexa_setup_service中的Makefile文件，编译生成两个**.cgi**文件，将这两个新生成的**.cgi**替换buildroot/package/amlogic/avs-sdk/cgi-bin/目录下的两个.cgi文件

## 4 重新编译avs-sdk并配置avs

```
make avs-sdk-dirclean
make avs-sdk-rebuild
make
```

将新生成的镜像烧录到板端，用手机app的方式配置avs,配置成功后，打印信息会出现一个Amazon网址和和一个字符串，登录这个网站根据提示将字符串输入，即可完成作者认证。avs能够正常工作。

认证完成之后，avs 就成功更换绑定账户了。

## 5 保存 json文件和相关库

在上一步avs 配置完成之后，为了下次操作简便，可以将板端`/etc/AlexaClientSDKConfig.json`和`/data/share/avs/`下的**.db**文件保存到本地，在下次配置avs时使用。