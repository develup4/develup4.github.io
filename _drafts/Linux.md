# Linux
>> id
나는 누구인가?

>>who
이 컴퓨터에는 누구누구가 접속해 있는가?

일반유저
develup4@ubuntu:(물결)$ 달러로 표시

슈퍼유저
root@ubuntu:-# 샾으로 표시

>> su
슈퍼유저로 로그인

>> sudo passwd -u root
-u는 unlock의 의미이다. 비밀번호 재설정.

슈퍼유저의 홈디렉토리
/root

일본유저
/home/{user}

>> sudo useradd -m {username}
유저추가

>> sudo usermod -a -G sudo {username}
유저를 sudo group에 넣는다. 즉 sudo사용이가능한 계정으로 만든다.

>> chmod u+r perm.txt
유저에게 리드 권한 부여

>> chmod 777 perm.txt
숫자로보통 표현한다(유저, 그룹, other)

>> ip addr
공인아이피 주소 확인

ipinfo.io/ip
아이피주소를 확인해볼 수 있는 사이트