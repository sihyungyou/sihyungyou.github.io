---
layout: post
title: "gitignore 활용하기"
tags: [개발, 일상]
comments: true
---

> 새로 배웠당  

### .gitignore  
로컬에서 깃헙 레포지토리의 변경사항을 커밋하고 푸쉬하다 보면 꼭 깃헙에 올리지 않아도 되는 파일이나 폴더들이 있다. VSC를 쓰다보면 .vscode 폴더가 그렇고 한번쯤은 봤을 .DS_Store 파일도 그 중 하나다. 이것들을 깃헙에 커밋할 때 자동으로 무시하도록 정의해놓는 문서가 .gitignore 파일이다. 필요없기도 하지만 깃헙 레포에 안이쁘게(?) 올라가서 자주 사용했는데 오늘은 이미 커밋된 사항들에 대해서 gitignore를 적용하는 방법을 배웠다.  

### 사용법  
사용법은 간단하다. 그냥 .gitignore 파일을 열고 내가 원치 않는 파일들을 써주면 된다.  
~~~
.DS_Store*
a.out*
.vscode/
.idea/
~~~

이렇게 폴더의 경우는 뒤에 /를 붙여주고 특정 이름의 파일들 모두에 대해 적용하려면 *를 붙여주면 된다.  

### 그런데!!  
이렇게 추가를 하고 다시 커밋을 해도 뻔뻔하게 깃헙 레포지토리에 함께 푸쉬되어있는 경우가 있다. 이는 이미 커밋된 사항에 대해서 gitignore가 적용되지 않았기 때문이다. 이럴 땐 git cached 사항을 삭제하고 다시 올려주면 된다. 명령어는 .vscode 폴더를 예시로 사용해보겠다.  
~~~
git rm --cached .vscode/settings.json
~~~

혹은 폴더 내에 여러 경로가 더 있고 다른 파일들이 있을 경우엔 -r 옵션을 추가해주면 된다. 아래는 내 블로그 레포지토리에 있는 폴더를 예시로 들어보았다.  
~~~
git rm -r --cached .sass-cache/
~~~

이렇게 이미 캐시되어있는 커밋사항(?)을 삭제하고 다시 .gitignore 파일을 푸시하면  

![Center example image](https://user-images.githubusercontent.com/35067611/69910060-571de800-1448-11ea-8cb0-dc4a14591673.png "Center"){: .center-image}  
이렇게 이쁘게 사라진 모습을 볼 수 있다!  