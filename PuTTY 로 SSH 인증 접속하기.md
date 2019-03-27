# PuTTY 로 SSH 인증 접속하기



## 1. 공개키, 개인키 만들기



####  PuTTY-gen 사용

![1552454461867](C:\Users\tkdlq\AppData\Roaming\Typora\typora-user-images\1552454461867.png)

키 생성기의 모습이다.  

PuTTY-gen은 공개키와 개인키를 생성할 수 있는 툴이다  

작업 탭의 생성버튼을 누르면  



![1552454523456](C:\Users\tkdlq\AppData\Roaming\Typora\typora-user-images\1552454523456.png)

빈칸에 마우스커서를 열심히 비벼야한다..!



![1552454563286](C:\Users\tkdlq\AppData\Roaming\Typora\typora-user-images\1552454563286.png)

공개키와 개인키가 만들어진다!



![1552454584389](C:\Users\tkdlq\AppData\Roaming\Typora\typora-user-images\1552454584389.png)

아래의 작업탭에서 개인키와 공개키를 저장할 수 있다.



#### Linux 내에서 생성

리눅스 자체에서 공개키와 내부키를 만들 수 있다.

~~~
$ ssh-keygen
~~~

명령어를 입력하면 키가 생성된다. 확인해보자.



~~~
$ cd .ssh
$ ls -l
~~~

키를 생성하면 숨김폴더인 .ssh폴더가 생성된다 cd 로 이동하여 ls 로 목록을 출력해보자  

개인키인` id_rsa `파일과 공개키인` id_rsa.pub`파일이 생성되었음을 확인할 수 있다.



~~~
$ cp id_rsa.pub authorized_keys
~~~

공개키를 `authorized_keys`파일로 복사해주자.



`ssh-keygen`으로 공개키/비밀키 한 쌍을 생성하고, 공개키 내용을 접속할 서버에 `~/.ssh/authorized_keys`에 저장하면 해당 서버에 비밀번호 없이 ssh 접속이 가능하다.



유의할 점은 리눅스 내부에서 만든 개인키는 PuTTY로 접속할시 양식이 달라 PuTTY-gen을 이용하여 형식을 바꾸어 준다.  



![1552455057892](C:\Users\tkdlq\AppData\Roaming\Typora\typora-user-images\1552455057892.png)

 PuTTY-gen의 작업탭에 있는 개인키 파일 불러오기로 개인키를 불러오자.



![1552455092524](C:\Users\tkdlq\AppData\Roaming\Typora\typora-user-images\1552455092524.png)

해당 작업완료창이 팝업되고 이제 개인키를 저장해서 사용하면된다.



## 2. Azure 공개키 입력

Azure에서 사용할 시 공개키를 입력해주어야한다.

![1552455216916](C:\Users\tkdlq\AppData\Roaming\Typora\typora-user-images\1552455216916.png)

![1552455225220](C:\Users\tkdlq\AppData\Roaming\Typora\typora-user-images\1552455225220.png)

가상머신 리소스로 들어가서 왼쪽하단 지원 및 문제해결 탭을 보면 `암호 다시 설정` 기능이 있다.



![1552455387293](C:\Users\tkdlq\AppData\Roaming\Typora\typora-user-images\1552455387293.png)

기본적인 문자 암호와 SSH 공개 키를 설정할 수 있다. 이번에는 SSH공개키를 설정하고.

사용자 이름과 공개키를 입력하자.

이제 Azure 가상머신에서 SSH키를 이용한 로그인이 가능하다.



## 3. PuTTY로 접속

![1552455493433](C:\Users\tkdlq\AppData\Roaming\Typora\typora-user-images\1552455493433.png)

Azure의 경우 DNS 이름을, 온프로미스의 경우 IP주소로 접속한다.

* Azure 가상머신의 IP를 입력해 접속하고 싶다면 방화벽을 설정해주어야한다.

이제 PuTTY를 켜보자.



![1552455608315](C:\Users\tkdlq\AppData\Roaming\Typora\typora-user-images\1552455608315.png)

PuTTY 기본 세션 설정 창 호스트이름에 주소를 입력해주고 포트도 입력해준다

* SSH포트는 22 번 포트



![1552455692058](C:\Users\tkdlq\AppData\Roaming\Typora\typora-user-images\1552455692058.png)

SSH 키 접속을 위해 좌측 연결탭 - SSH -Auth 항목으로 이동하고 인증 개인키 파일에 개인키 경로를 입력해준다.

이후 하단 열기버튼을 입력하면 연결이 시작된다.



![1552455476132](C:\Users\tkdlq\AppData\Roaming\Typora\typora-user-images\1552455476132.png)

사용자 이름만 입력하면 바로 접속됨을 알 수 있다.

~.~ 올레~













