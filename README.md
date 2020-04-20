# 海关对接 php xml加签
## 报文介绍
海关申报有进口申报和出口申报等，根据你的业务类型决定需要对接报文，具体可以咨询客服，本文是以进口订单申报来介绍报文加签申报的过程。

报文有两种：一种是CEB开头的这种是总署统一版的报文，例如：CEB311Message 这个是进口电子订单报文；还有一种是KJ开头的这种是公共平台的报文，例如：KJ881111，这个也是进口电子订单报文。
这两种报文都能实现申报电子订单的申报，建议还是选择CEB格式的报文，这个是新版的报文，KJ好像是老版的，对接群里有用KJ报文对接三单对碰有问题转CEB的，所以最好还是用CEB报文。本文的加签也是用CEB报文来加签的。

## 加签申报步骤
报文申报分两步：第一步是报文加签推送到公共平台，第二步是在在第一步的基础上加海关签来申报。

第一步的报文加签：
海关提供两种加签方式：一种是用他们提供的客户端加签申报；另一种是使用http程序来加签申报。本文是介绍http程序加签申报的。
这个客户端即可以给第一步的报文加签，也可以给第二步的报文加海关签，具体的客户端配置可以看文档，是以报文文件形式来处理的，不如http程序加签灵活。

第二步的海关加签:
配置好海关的客户端插上UKEY，运行客户端，客户端会自动下载你第一步提交到公共平台的报文文件，自动加海关签申报，所以给报文加海关签是不需要我们处理的。当然你也可以用加密机来加海关签，这个要申请的。
加海关签其实也是给报文加签，只不过用的客户端来加签。原理我猜测也是用公钥私钥加签，公钥私钥集成在UKEY中，所以海关签必须用海关提供的客户端或是加密机。
## http程序加签

1. 生成对应类型报文，注意一定要按照海关提供的格式来，稍有不同就不对。我的例子是生成的CEB311Message的订单报文是"./xml/CEB311Message.xml"。

2. 转成http申报格式，具体见'./xml/unsigned-ceb.xml',这个是待加签文件。

3. 对待加签内容加签，加签后的文件是'./xml/signed-ceb.xml'。

4. 采用post 方式提交到公共平台。返回的结果只是提交的情况，申报成功与否还要看公共平台的回执信息。

## 加签说明
海关提供的是java用的pkcs8格式的公钥私钥，要转换成php用的pkcs1格式的公用私钥。

转换命令：
私钥 openssl rsa -inform DER -in privatekey.key  -outform PEM -out privatekey.pem

公钥 openssl rsa -inform DER -in publickey.key  -pubin -outform PEM -out publickey.pem

xml目录下的privatekey.key、publickey.key 是海关提供的猜测环境的pkcs8格式的私钥公钥，privatekey.pem、publickey.pem是我转化后的pkcs1格式私钥公钥。

xml加签用的是这个包：https://github.com/robrichards/xmlseclibs

加签示例运行index.php 文件即可，配置的是测试环境的参数；正式环境只需要更改参数，开启客户端就行。

测试环境不能加海关签，到http程序申报成功就可以了。

## 注意问题

客户端配置要注意密码，一般有个初始密码，也可以更改。这个一定要配置对，配置错了客户端启动几次会锁卡，好像是4次吧；锁卡后就要到发卡处解卡，解卡费260，别问我是怎么知道的，掉进去过。

查看申报状态可以去公共平台的测试环境查看，状态是转发成功表示提交到公共平台成功了；状态是无需转发表示报文编号或是订单号已经申报过，想要在测试就要更改订单号和报文编号（guid）。

海关加签其实也是给'./xml/CEB311Message.xml'的内容加签，只不过是由客户端来加签，所以http程序加签里是不用对这个文件的内容加签的。
