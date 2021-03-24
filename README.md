---
title: 'CHSH AI2 Server Note'
tags: AI2
---

# CHSH AI2 Server Note
https://ai2.linwebs.tw/


[TOC]

## 說明
以下範例  
目錄以 `/data-disk/ai2-server/appinventor-sources-20200523/` 為例  
Domain以 [ai2.linwebs.tw](https://ai2.linwebs.tw/) 為例

## AI2 Server 登入網址 (Proxy)
指定使用者名稱登入網址

[https://ai2.linwebs.tw/login/google](https://ai2.linwebs.tw/login/google)

## AI2 Server Web Root Path
> /data-disk/ai2-server/appinventor-sources-20200401/appinventor/appengine/build/war

## AI2 Server 轉址到Apache系統登入
需新增以下網址轉址至login.jsp，以免從AI2 Server登出後，看到原生登入畫面

> /data-disk/ai2-server/appinventor-sources-20200401/appinventor/appengine/build/war/login.jsp

```
<%
response.sendRedirect("https://ai2.chsh.chc.edu.tw");
%>
```

## AI2 Server Logo
Logo更改需更改以下6個檔案

Logo原始檔位址: https://ai2.chsh.chc.edu.tw/images/codi_long.png

https://ai2.chsh.chc.edu.tw/images/logo2.png

小圖 ([AI2](https://ai2.linwebs.tw/)): 
![AI2 Logo](https://ai2.chsh.chc.edu.tw/images/codi_long.png)

小圖(英文) ([AI2](https://ai2.linwebs.tw/)): 
![AI2 Logo](https://ai2.chsh.chc.edu.tw/images/codi_long-en.png)

大圖 ([reference](https://ai2.linwebs.tw/reference/)): 

![AI2 Logo](https://ai2.chsh.chc.edu.tw/images/logo2.png)

修改請參照下方 [AI2系統檔案存放位置](#AI2%E7%B3%BB%E7%B5%B1%E6%AA%94%E6%A1%88%E5%AD%98%E6%94%BE%E4%BD%8D%E7%BD%AE) 指示進行修改


## Build Server Status (Proxy)
BuildServer 運行狀況
[https://ai2.linwebs.tw/status/buildserver/health](https://ai2.linwebs.tw/status/buildserver/health)

BuildServer 即時狀況
[https://ai2.linwebs.tw/status/buildserver/vars](https://ai2.linwebs.tw/status/buildserver/vars)

## 彰中AI2 TinyWebDB
網址:
[https://ai2.linwebs.tw/db/](https://ai2.linwebs.tw/db/)

自動啟動TinyWebDB腳本位置
```bash
$ sudo vim /etc/rc.local
```

用root權限新增一個名稱為[ai2db]的 screen，並執行指令


## AI2 Server自動啟動
自動啟動彰中AI2 Server腳本位置
```bash
$ sudo vim /etc/rc.local
```

用root權限新增
一個名稱為[ai2sys]的screen，執行AI2 Server  
一個名稱為[ai2pack]的screen，執行LocalBuildServer  
另外一個名稱為[ai2db]的 screen，是執行TinyWebDB (非必要)
```shell=
screen -dmS ai2-sys sh
screen -S ai2-sys -p 0 -X stuff "/data-disk/ai2-server/appengine-java-sdk-1.9.77/bin/dev_appserver.sh -p 8888 -a 0.0.0.0 /data-disk/ai2-server/appinventor-sources-20200523/appinventor/appengine/build/war/
"

screen -dmS ai2-pack sh
screen -S ai2-pack -p 0 -X stuff "cd /data-disk/ai2-server/appinventor-sources-20200523/appinventor/buildserver/
sleep 30
ant RunLocalBuildServer address=ai2.chsh.chc.edu.tw
"

screen -dmS ai2-db sh
screen -S ai2-db -p 0 -X stuff "python2.7 /data-disk/ai2db/appengine_py/dev_appserver.py --port 9980 --host 0.0.0.0 /data-disk/ai2db/appinventordb/
"
```
## Apache Proxy 設定
設定檔位置
> /etc/httpd/conf.d/vhost-3-ai2-linwebs-tw.conf

```
# ai2.linwebs.tw
<VirtualHost *:80>
    ServerName ai2.linwebs.tw
    ServerAdmin admin@linwebs.tw
    ProxyRequests Off
    ProxyPreserveHost On
    <Location /status/>
        ProxyPass http://localhost:9990/
    </Location>
    <Location />
        ProxyPass http://localhost:8888/
    </Location>
    ErrorLog logs/ai2-linwebs-error_log
    CustomLog logs/ai2-linwebs-access_log common
</VirtualHost>

# ai2.linwebs.tw SSL
<VirtualHost *:443>
    ServerName ai2.linwebs.tw
    ServerAdmin admin@linwebs.tw
    ProxyRequests Off
    ProxyPreserveHost On
    <Location /status/>
        ProxyPass http://localhost:9990/
        ProxyPassReverse http://localhost:9990/
    </Location>
    <Location />
        ProxyPass http://localhost:8888/
        ProxyPassReverse http://localhost:8888/
    </Location>
    ErrorLog logs/ai2-linwebs-ssl-error_log
    CustomLog logs/ai2-linwebs common
    SSLEngine on
    SSLProxyEngine on
    SSLProtocol ALL -SSLv2 -SSLv3 -TLSv1 -TLSv1.1
    SSLHonorCipherOrder on
    SSLCipherSuite ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA
    SSLCertificateFile /etc/letsencrypt/live/ai2.linwebs.tw/cert.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/ai2.linwebs.tw/privkey.pem
    SSLCertificateChainFile /etc/letsencrypt/live/ai2.linwebs.tw/chain.pem
    SetEnvIf User-Agent ".*MSIE.*" \
         nokeepalive ssl-unclean-shutdown \
         downgrade-1.0 force-response-1.0
</VirtualHost>
```

SSL 自動更新腳本
`# sudo crontab -e`

 每周一凌晨3:10自動更新SSL憑證
```
10 3 * * 1 /opt/letsencrypt/letsencrypt-auto certonly --apache --renew-by-default -d ai2.linwebs.tw && service httpd restart
```

## AI2系統檔案存放位置
檔案存放對應路徑:  
Web Path:
https://ai2.linwebs.tw/static/images/codi_long.png
Document Root:
> /data-disk/ai2-server/appinventor-sources-20200523/appinventor/appengine/build/war/static/images

## AI2 Server 更新
切換至ai2 server目錄
```bash
$ cd /data-disk/ai2-server
```

取得source檔:
```bash
$ git clone https://github.com/mit-cml/appinventor-sources.git
```

更改source檔名稱:
(yyyymmdd填入年-月-日)
```bash
$ mv appinventor-sources appinventor-sources-yyyymmdd
```

取得最新google app engine java 網址
```bash
$ wget https://cloud.google.com/appengine/docs/standard/java/download
```

解壓縮google app engine
(xx填入版本號)
https://cloud.google.com/appengine/docs/standard/java/download
```bash
$ unzip appengine-java-sdk-1.9.xx.zip
```

更改兩目錄權限為777
```bash
$ chmod 777 appengine-java-sdk-1.9.xx appinventor-sources-yyyymmdd -R
```

進入ai2 source目錄
```bash
$ cd appinventor-sources-yyyymmdd/
```

submodule update
```bash
$ git submodule update --init
```

進入ai2 目錄
```bash
$ cd appinventor
```

編譯ai2 source檔
```bash
$ ant
```

複製先前版本使用者資料
```bash
$ cp /data-disk/ai2-server/appinventor-sources-20180124/appinventor/appengine/build/war/WEB-INF/appengine-generated /data-disk/ai2-server/appinventor-sources-yyyymmdd/appinventor/appengine/build/war/WEB-INF/appengine-generated -r
```

## ant MakeAuthKey
:::warning
:warning: 此指令已經不須再執行了，執行會使原本的使用者都無法登入
:::

AI2 系統須有 auth key 才可運作
預設是存在執行的使用者根目錄，所以使用 sudo 執行時是放在以下資料夾內
> /root/.appinventor

執行指令
```bash
$ cd /data-disk/ai2-server/appinventor-sources-yyyymmdd/appinventor/
$ ant MakeAuthKey
```

## 更新時須修改檔案(撰寫中)
以下內容撰寫中，請暫時不要使用
註: 以下 `$AI2_SOURCE_PATH` 的值為 `/data-disk/ai2-server/appinventor-sources-yyyymmdd`


* 語言翻譯
	> $AI2_SOURCE_PATH/appinventor/appengine/src/com/google/appinventor/client/languages.json

	* 繁「体」中文 => 繁體中文

* web 設定檔
	> $AI2_SOURCE_PATH/appinventor//appengine/war/WEB-INF/appengine-web.xml
	* 登入參數
	* 靜態網址... 等

## 新 google-cloud-sdk
/opt/google-cloud-sdk/bin/java_dev_appserver.sh

## 路徑
* logo.png
	* /appinventor/appengine/war/static/images/
* codi_long.png
	* /appinventor/appengine/war/static/images/codi_long.png
	* /appinventor/docs/markdown/reference/blocks/images/codi_long.png
	* /appinventor/docs/markdown/reference/components/images/codi_long.png
	* /appinventor/docs/html/reference/blocks/images/codi_long.png
	* /appinventor/docs/html/reference/components/images/codi_long.png
* 圖片 /appinventor/appengine/src/com/google/appinventor/images/
* 語言 /appinventor/appengine/src/com/google/appinventor/client/

## 舊 crontab
00 4 1 4,8,12 * sh /data-disk/ai2-server/update/update.sh > /data-disk/ai2-server/update/update.log

## 翻譯
欢迎使用 App Inventor 2 测试版！

## 模擬器 error code

solution: https://community.appinventor.mit.edu/t/aistarter-update-mit-ai2-companion2-to-2-58au/7587

```
Error from Companion: java.lang.RuntimeException: invalid syntax in eval form: :9:1: caught exception in inliner for # - java.lang.RuntimeException: no such class: com.google.appinventor.components.runtime.com.google.appinventor.components.runtime.Label gnu.bytecode.ObjectType.getReflectClass(ObjectType.java:148) gnu.bytecode.ClassType.getModifiers(ClassType.java:103) gnu.bytecode.ClassType.isInterface(ClassType.java:471) gnu.expr.InlineCalls.checkType(InlineCalls.java:56) gnu.expr.InlineCalls.visit(InlineCalls.java:49) gnu.expr.InlineCalls.visitSetExpValue(InlineCalls.java:363) gnu.expr.InlineCalls.visitSetExpValue(InlineCalls.java:28) gnu.expr.ExpVisitor.visitSetExp(ExpVisitor.java:114) gnu.expr.InlineCalls.visitSetExp(InlineCalls.java:369) gnu.expr.InlineCalls.visitSetExp(InlineCalls.java:28) gnu.expr.SetExp.visit(SetExp.java:406) gnu.expr.ExpVisitor.visit(ExpVisitor.java:55) gnu.expr.InlineCalls.visit(InlineCalls.java:46) gnu.expr.InlineCalls.visitBeginExp(InlineCalls.java:272) gnu.expr.InlineCalls.visitBeginExp(InlineCalls.java:28) gnu.expr.BeginExp.visit(BeginExp.java:156) gnu.expr.ExpVisitor.visit(ExpVisitor.java:51) gnu.expr.InlineCalls.visit(InlineCalls.java:46) gnu.expr.InlineCalls.visitBeginExp(InlineCalls.java:272) gnu.expr.InlineCalls.visitBeginExp(InlineCalls.java:28) gnu.expr.BeginExp.visit(BeginExp.java:156) gnu.expr.ExpVisitor.visit(ExpVisitor.java:51) gnu.expr.InlineCalls.visit(InlineCalls.java:46) gnu.expr.InlineCalls.visitLetExp(InlineCalls.java:317) gnu.expr.InlineCalls.visitLetExp(InlineCalls.java:28) gnu.expr.LetExp.visit(LetExp.java:207) gnu.expr.ExpVisitor.visit(ExpVisitor.java:51) gnu.expr.InlineCalls.visit(InlineCalls.java:46) gnu.expr.InlineCalls.visit(InlineCalls.java:28) gnu.expr.LambdaExp.visitChildrenOnly(LambdaExp.java:1664) gnu.expr.LambdaExp.visitChildren(LambdaExp.java:1651) gnu.expr.InlineCalls.visitScopeExp(InlineCalls.java:279) gnu.expr.InlineCalls.visitLambdaExp(InlineCalls.java:349) gnu.expr.InlineCalls.visitLambdaExp(InlineCalls.java:28) gnu.expr.LambdaExp.visit(LambdaExp.java:1640) gnu.expr.ExpVisitor.visit(ExpVisitor.java:55) gnu.expr.InlineCalls.visit(InlineCalls.java:46) gnu.expr.InlineCalls.visit(InlineCalls.java:28) gnu.expr.ExpVisitor.visitAndUpdate(ExpVisitor.java:162) gnu.expr.ExpVisitor.visitExps(ExpVisitor.java:176) gnu.expr.ApplyExp.visitArgs(ApplyExp.java:416) gnu.kawa.reflect.CompileInvoke.validateApplyInvoke(CompileInvoke.java:23) java.lang.reflect.Method.invokeNative(Native Method) java.lang.reflect.Method.invoke(Method.java:521) gnu.expr.InlineCalls.maybeInline(InlineCalls.java:467) gnu.expr.QuoteExp.validateApply(QuoteExp.java:150) gnu.expr.ReferenceExp.validateApply(ReferenceExp.java:191) gnu.kawa.functions.CompilationHelpers.validateApplyToArgs(CompilationHelpers.java:66) java.lang.reflect.Method.invokeNative(Native Method) java.lang.reflect.Method.invoke(Method.java:521) gnu.expr.InlineCalls.maybeInline(InlineCalls.java:467) gnu.expr.QuoteExp.validateApply(QuoteExp.java:150) gnu.expr.ReferenceExp.validateApply(ReferenceExp.java:191) gnu.expr.InlineCalls.visitApplyExp(InlineCalls.java:119) gnu.expr.InlineCalls.visitApplyExp(InlineCalls.java:28) gnu.expr.ApplyExp.visit(ApplyExp.java:411) gnu.expr.ExpVisitor.visit(ExpVisitor.java:55) gnu.expr.InlineCalls.visit(InlineCalls.java:46) gnu.expr.QuoteExp.validateApply(QuoteExp.java:162) gnu.expr.ReferenceExp.validateApply(ReferenceExp.java:191) gnu.kawa.functions.CompilationHelpers.validateApplyToArgs(CompilationHelpers.java:66) java.lang.reflect.Method.invokeNative(Native Method) java.lang.reflect.Method.invoke(Method.java:521) gnu.expr.InlineCalls.maybeInline(InlineCalls.java:467) gnu.expr.QuoteExp.validateApply(QuoteExp.java:150) gnu.expr.ReferenceExp.validateApply(ReferenceExp.java:191) gnu.expr.InlineCalls.visitApplyExp(InlineCalls.java:119) gnu.expr.InlineCalls.visitApplyExp(InlineCalls.java:28) gnu.expr.ApplyExp.visit(ApplyExp.java:411) gnu.expr.ExpVisitor.visit(ExpVisitor.java:51) gnu.expr.InlineCalls.visit(InlineCalls.java:46) gnu.expr.InlineCalls.visitBeginExp(InlineCalls.java:272) gnu.expr.InlineCalls.visitBeginExp(InlineCalls.java:28) gnu.expr.BeginExp.visit(BeginExp.java:156) gnu.expr.ExpVisitor.visit(ExpVisitor.java:51) gnu.expr.InlineCalls.visit(InlineCalls.java:46) gnu.expr.InlineCalls.visit(InlineCalls.java:28) gnu.expr.LambdaExp.visitChildrenOnly(LambdaExp.java:1664) gnu.expr.LambdaExp.visitChildren(LambdaExp.java:1651) gnu.expr.InlineCalls.visitScopeExp(InlineCalls.java:279) gnu.expr.InlineCalls.visitLambdaExp(InlineCalls.java:349) gnu.expr.InlineCalls.visitLambdaExp(InlineCalls.java:28) gnu.expr.ExpVisitor.visitModuleExp(ExpVisitor.java:103) gnu.expr.ModuleExp.visit(ModuleExp.java:482) gnu.expr.ExpVisitor.visit(ExpVisitor.java:51) gnu.expr.InlineCalls.visit(InlineCalls.java:46) gnu.expr.InlineCalls.inlineCalls(InlineCalls.java:33) gnu.expr.Compilation.walkModule(Compilation.java:994) gnu.expr.Compilation.process(Compilation.java:1965) gnu.expr.ModuleInfo.loadByStages(ModuleInfo.java:330) gnu.expr.ModuleExp.evalModule1(ModuleExp.java:238) gnu.expr.ModuleExp.evalModule(ModuleExp.java:198) gnu.expr.Language.eval(Language.java:943) gnu.expr.Language.eval(Language.java:883) gnu.expr.Language.eval(Language.java:865) com.google.appinventor.components.runtime.util.AppInvHTTPD.serve(AppInvHTTPD.java:188) com.google.appinventor.components.runtime.util.NanoHTTPD$HTTPSession.run(NanoHTTPD.java:470) java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1068) java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:561) java.lang.Thread.run(Thread.java:1096)
```