## R Studio Server 설치

PuTTY를 이용해 접속하자.

잊어버렸다면 `Tool폴더의 PuTTY로 SSH인증접속`을 확인해보자.



그전에 사용한 R 프로그램은 R Client 였고 오늘은 Server에 R을 설치에 더 많은 자원을 사용하는 R server에 대해서 알아보자



```
yum install epel-release
```

epel-release 설치

- epel-release는 Fedora Project에서 제공되는 저장소로 각종 패키지의 최신 버전을 제공하는 community 기반의 저장소이다.

su 권한으로 실행해야함을 잊지말자.



```
yum update
```

업데이트를 실시한다.



```
shutdown -r now
```

적용시키기 위해서 재시작.



```
yum install R -y
```

R server를 사용하기 위해서는 기본적으로 R이 설치되어 있어야 한다. R 설치



```
wget https://download2.rstudio.org/rstudio-server-rhel-1.1.463-x86_64.rpm
```

rstudio-server의 다운로드 주소이다. wget으로 설치파일을 다운로드 받자.



```
yum install rstudio-server-rhel-1.1.463-x86_64.rpm
```

다운로드 받은 설치파일로 rstudio-server 설치



```
yum groupinstall "Development Tools" -y
```

개발도구 설치



```
systemctl status rstudio-server
```

rstudio-server 서비스의 상태를 확인한다.

active 상태 (실행중 상태)가 아닐 경우 아래 명령어를 입력하여 서비스를 실행시켜준다.



```
systemctl start rstudio-server
```

rstudio-server 서비스 실행 ~



```
rstudio-server
```

정상적으로 설치되었는지 확인하자.

설치되었다면 입력할 수 있는 명령어 목록이 출력된다.



```
rstudio-server active-sessions
```

세션을 활성화해준다.



```
http://localhost 또는 ip:8787/ -> R Web GUI
```

나의 경우는 192.168.137.101:8787 을 웹브라우저에 입력하면

Web GUI R 로 접속된다