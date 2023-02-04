---
layout: single
title: vrziny.github.io 오픈
date: 2023-02-04 11:11 +0900
category: Flutter
---

Github Pages에 Jekyll로 이미 구축된 사이트에 Flutter App을 Web으로 빌드하여 배포하는 방법에 대해서 기술합니다.

&nbsp;
>**NOTE**
>
> Private 저장소에서는 Pages를 만드려면 구독을 해야해서 빌드된 결과물만을 배포하기 위한 문서입니다. Public 저장소이면 해당 저장소에서 Pages를 생성하세요.

&nbsp;&nbsp;
# 1. Flutter에서 Web으로 빌드
아래 명령 수행 후 build 폴더에서 web 폴더가 만들어졌는지 확인해주세요.
```
$ flutter build web
```

&nbsp;&nbsp;
# 2. Jekyll에서 subpage 및 app 폴더 생성
VSCode에서 하셔도 되고 윈도우즈 환경이면 그냥 탐색기에서 폴더 만들어주면 됩니다.

주의하셔야 할 점은 폴더 이름이 "test"와 같으면 제외될 가능성이 있으니 동작하지 않는다면 Actions의 Workflow에서 로그를 잘 확인해보세요.

```
$ mkdir subpage && cd subpage
$ mkdir my_flutter_app

or

$ mkdir -p subpage/my_flutter_app
```

&nbsp;&nbsp;
# 3. Flutter의 Web 빌드 결과물을 Jekyll로 복사
플러터 프로젝트에서 만들어진 build/web 폴더 내부를 복사하여 subpage 폴더 하위의 앱 폴더에 복사합니다.

&nbsp;&nbsp;
# 4. index.html 파일의 base href를 수정

복사한 파일 중 index.html 파일을 열어 base href 를 수정합니다.

수정 이전
```html
<base href="/">
```
수정 이후
```html
<base href="/subpage/my_flutter_app/">
```

&nbsp;&nbsp;
# 5. 배포 및 테스트
Jekyll 프로젝트를 Commit하고 Push 합니다.

Deploy가 완료되면 아래와 같이 본인이 작성한 App을 테스트해보세요.
```
https://vrziny.github.io/subpage/my_flutter_app
```

&nbsp;&nbsp;
# 참고
https://jekyllrb.com/docs/pages/