## Linux
- 리눅스 토발즈의 linux 로 부터 많은 linux os들이 파생

### 왜 Server엔 Linux OS를?
- Windows 같은 경우 유료 OS며, MacOS 같은 경우 비싼 Mac 장비를 사용해야 하므로 무료 OS인 linux를 많이 사용

### Linux Kernel
- 'Linux Kernel이 곧 Linux다'라고 생각해도 무방하다.
- OS의 최하단, 기계를 상대하는 최전선에서 직접 다루고 관리함
- 운영체제라는 software의 존재이유이자 핵심, user가 computer에 명령을 내리고 result를 받아올 수 있는건 이 kernel 덕분!
![Demystifiying The Linux Kernel – Digilent Blog](https://blog.digilentinc.com/wp-content/uploads/2015/05/1280px-Kernel_Layout.svg_.png)

### Linux 주요 Directory
 directory | description
  ---|:---:
  bin | 기본 명령어들이 저장 된 디렉토리
  boot | 부팅에 필요한 file들이 저장되는 곳
  dev | system device 관련 file들이 저장되는 곳
etc|System setting에 관련 된 각종 file들이 저장되는 곳
home|User의 Home directory가 생성되는 곳
lib|Kernel과 Program에 필요한 각종 library가 저장되는 곳
media|CD,USB같은 external device를 연결하는 곳
mnt|탈부착 가능한 device들을 임시로 연결하는 곳(WSL의 경우 Windows의 directory와 연결)
opt|추가 package가 설치되는 곳
root|root(최고 관리자)계정의 홈 디렉토리
run|실행중인 service와 관련된 file들이 저장되는 곳
sbin|System 관리자용 command들이 저장되는 곳
sys|Linux Kernel 관련 정보가 있는 곳
tmp|System 사용중 발생하는 temporary data가 저장되는 곳
usr|기본 실행 file, library, header file등이 저장되는 곳
var|System 운영중 발생하는 data,log가 저장되는 곳
proc|실행중인 process및 kernel 정보가 저장되는 곳, disk상이 아닌 memory에 존재

### Linux 주요 Command
#### Clear
화면 clear
#### ls
option| description
  ---|:---:
  -F|directory는 /, 실행가능 file은 *, socket file은 =, link인 경우 @를 file 뒤에 표시
 -l | 각 항목의 상세 정보들을 함께 표시
  > option을 합성하는 것도 가능
  > ```shell
  > ls -lF
  > ```
#### wget
w(Web)get : web에서 download 한다.

## WSL
- MacOS 가 개발에 적합하다 불리우는 것은 linux 와의 호환성이 크다. linux와 MacOS 모두 unix로 부터 발전되어서 terminal 명령어들이 공통된 것이 많다. 또한 linux software를 Mac에서 바로 돌려볼 수도 있다.
- 허나 2015년 MS에서 WSL을 발표했고 Windows에서 Linux를 CLI로 돌려볼 수 있게 됬다. 타사 software를 다운 받을 필요 없이 바로 linux를 깔아서 쓸 수 있다.

> **wsl 설치**
> https://docs.microsoft.com/ko-kr/windows/wsl/install-win10

## Ubuntu
Debian 계열 linux
