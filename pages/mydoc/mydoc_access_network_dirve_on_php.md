---
title: PHP(CGI)에서 Windows Network Drive에 파일 쓰기
keywords: php, windows, network, drive, write
tags: Web
published: true
sidebar: mydoc_sidebar
permalink: mydoc_access_network_dirve_on_php.html
folder: mydoc
---

IIS에서 CGI로 PHP를 사용할 때 PHP에서 네트워크 드라이브(i.e. \\192.168.0.1)에 읽기/쓰기를 하려고 하면 접근권한이 없어 실패한다.(Permission denied)  

먼저 접근하려는 네트워크 드라이브를 연결한 후 **네트워크 드라이브 속성 > 보안 탭 > 그룹 또는 사용자 이름**에 있는 목록을 확인한다. 네트워크 드라이브가 연결돼있다는 건 이 목록에 있는 어떤 계정의 ID/PW로 접속했다는 증거다. 예를 들어 이 ID/PW가 myuser/myuser라고 한다면 윈도우 탐색기에서 네트워크 드라이브를 연결하여 사용할때는 이 ID/PW만 알아도 된다. 그러나 PHP에서 이 폴더에 접근할 때는 IIS를 통해 접근하기 때문에 IUSR라는 계정으로 접근한다(익명인증을 디폴트로 사용한다면). 그래서 네트워크 드라이브의 **그룹 또는 사용자 이름**에는 IUSR가 없기 때문에 이를 거부한다.

>**_해결책은?_**

IIS의 서버 혹은 가상디렉토리에서 **인증** 기능에 들어가면 **익명 인증**이 사용상태일 것이다. 익명 인증 편집 메뉴에 들어가면 사용자 지정이 IUSR로 돼있는 것을 볼 수 있다. 설정 버튼을 누르면 사용자 이름과 암호를 입력하도록 하는데 여기서 네트워크 드라이브에 연결할 때 사용했던 myuser/myuser 계정을 입력한다. 단, 이 계정이 내 로컬에도 존재해야 하므로 **내컴퓨터 > 관리 > 로컬 사용자 및 그룹 > 사용자 > 새 사용자**에 해당 계정을 먼저 추가한 후 IIS에서 위 작업을 해줘야 한다.

>**_궁금증1: 네트워크 드라이브 그룹 또는 사용자 이름에 IUSR를 추가하면?_**

인증에 실패한다. 네트워크 드라이브에서 IUSR를 추가하면 해당 네트워크의 IUSR를 추가하게 되는데 이는 내 로컬의 IUSR와 다른 사용자다. 

해당 네트워크의 도메인을 A라고 하고 내 도메인을 B라고 가정하자. 해당 네트워크 드라이브의 **그룹 또는 사용자 이름**에 IUSR를 추가하면 **A\IUSR**를 추가한 것이다. 그러나 내 IIS에서 접근할 때는 **B\IUSR**로 접근하는 것이므로 두 사용를 다르게 보는 것이다.

>**_궁금증2: 네트워크 드라이브의 그룹 또는 사용자 이름에 로컬의 사용자를 추가하면?_**

이 역시 불가능하다. 해당 네트워크 드라이브의 그룹 또는 사용자 이름을 편집해보면 알겠지만 해당 네트워크에서는 다른 도메인의 사용자를 알 수 없다.

>**_사족_**

위에서 A\IUSR와 B\ISUR는 다른 사용자라고 했는데 A\myuser와 B\myuser 역시 다른 사용자가 아닌가라는 의문이 든다. 아마도 윈도우에서는 비밀번호까지 같을 때 도메인이 달라도 같은 사용자로 인식하는 것 같다(IUSR는 비밀번호를 설정한 적은 없지만 컴퓨터마다 IUSR의 비밀번호가 다를 것 같다). 그렇다해도 아주 우연히 서로 다른 도메인에서 같은 ID/PW를 사용하는 사용자가 있을 수도 있는데 이를 같은 사용자로 보는 것이 타당한지..