Azure CentOS 7.6 Putty 접속

1. Azure Portal 을 이용하여 DataBase 및 가상머신 생성 

![echo "# R" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/kdongxie/R.git
git push -u origin master1552455644716](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1552455644716.png)

2. 생성된 가상머신의 DNS를 설정 

![1552455705088](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1552455705088.png)

3. Local Computer의 Putty를 이용하여 Azure Portal 의 가상머신에 로그인 한다. 

4. 로그인 한 후 명령어를 통해 ssh key를 생성한다.

   ~~~
   ssh-keygen
   ~~~

5. 가상머신의 기본 루트에서 `.ssh` 의 디렉토리로 이동

   ~~~
   cd .ssh
   ~~~

6. dir을 통해 ssh key의 존재 여부를 확인한 후 `cat`를 통해 key를 확인한다. 

   ~~~
   dir
   ~~~

   ~~~
   cat .ssh/id_rsa.pub 
   ~~~

7. ssh key는 두가지로 Private Key 와 Public Key로 나뉘며, 파일은 Private : `id-rsa`  Public : `id_rsa.pub` 로 나뉜다.

8. 두개의 파일을 `cat` 의 명령어를 통해 확인 후 notepad에 `ctrl+c` `ctrl+v` 한 후 저장을 실시 한다. 

9.  putty를 실행하여 저장된 Private Key.txt 파일을 putty에서 읽을 수 있는 형식으로 변환해준다.

10. 이후 Privaate Key를 저장해 준다.

    ![1552456516193](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1552456516193.png)

11. 생성된 ssh Key를  putty에 적용 시킨 후 Azure 가상머신에 접속한다.

![1552456607459](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1552456607459.png)

12. 접속 후 Private Key에 권한을 주는 작업을 진행 한다. 

    ~~~
    chmod 700 .ssh
    ~~~

    ~~~
    chmod 600 .ssh/authorized_keys
    ~~~

13. 이 과정을 통해 Private Key를 가지고 있는 가상머신 환경에서 Password 가 없이 Azure 가상머신의 환경에 원격으로 로그인이 가능하다. 

    ![1552456824175](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1552456824175.png)

