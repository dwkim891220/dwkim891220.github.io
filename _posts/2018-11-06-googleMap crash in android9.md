---
title: "Android9 버전에서 GoogleMap 사용시 issue"
categories: 
- Issue
tags:
- GoogleMap
- Android9
last_modified_at: 2018-11-06
toc: true
---

- 문제 발생

Crashlytics에 새로운 이슈가 발생해 살펴보니 아래와 같은 예외가 발생 했다.

`Fatal Exception: java.lang.NoClassDefFoundError Failed resolution of: Lorg/apache/http/ProtocolVersion`

- 문제 해결

`AndroidManifest.xml`의 `<application>`tag 안에 `<uses-library android:name="org.apache.http.legacy" android:required="false" />` 추가.

출처 : [stackOverflow](https://stackoverflow.com/questions/50461881/java-lang-noclassdeffounderrorfailed-resolution-of-lorg-apache-http-protocolve)

- 마무리

문제가 발생한 기기는 Sony Xperia XZ2 (H8314) 에서 발생하는 이슈로, 안드로이드 최신버전을 사용하는 다른 기기에서 발생하는지는 확인 할 수 없었습니다.