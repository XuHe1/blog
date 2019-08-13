---
title: jvm安装证书中遇到的问题
date: 2019-08-13 14:52:58
tags:
	- JVM
    - CA
---

java程序在调用https接口时会报错:
<!-- more -->

```
Caused by: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
```

解决方法很简单,stackoverflow上给出了:

https://stackoverflow.com/questions/21076179/pkix-path-building-failed-and-unable-to-find-valid-certification-path-to-requ

但是由于同事也在同一台机器上安装了证书文件`jssecacerts`, 导致我的证书没起作用, 所以我怀疑是不是jvm读取证书有个优先顺序, 经过一番google后,终于证明了我的观点:

https://docs.oracle.com/javase/8/docs/technotes/guides/security/jsse/JSSERefGuide.html#X509TrustManager