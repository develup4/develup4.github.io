---
title:  Modern C++ in Android native - Binder driver

categories: C++ 
tags: Android  Binder
 
---

  
  
   
이제 C++이 아닌 Android specific한 내용이 나왔다.  
Android는 Linux 위에 구현되어 있기 때문에 Linux kernel의 동작부터 살펴보면,  
  
Linux kernel은 User mode와 Kernel mode로 나뉘어 있다(32bit 보호모드)  
시스템 프로그래밍을 공부했다면 반드시 들어봤을 내용이다.  
  
![]({{ site.url }}{{ site.baseurl }}/assets/images/2021-06-26-Modern C++ in Android native - Binder driver/%E1%84%83%E1%85%A1%E1%84%8B%E1%85%AE%E1%86%AB%E1%84%85%E1%85%A9%E1%84%83%E1%85%B3.jpeg.jpg)  
대표사진 삭제  
사진 설명을 입력하세요.  
  
Hardware를 조작하는 device drive 등은 Kernel mode에 존재하며,  
User mode에서는 그 driver에 해당하는 파일을 조작하여 HW를 조작한다.  
(물론 그 driver 파일에 대한 권한이 있어야 한다)  
  
예를 들어 LED를 켜는 driver가 있을때 권한이 아래와 같다고 가정해보자.  
led  root  rw-rw——  
  
권한이 root에게만 열려있으며 권한이 있을 경우 아래와 같이 조작해서 LED를 켤 수 있다.  
fd = open("/dev/led", O_WRONLY);  
write(fd, ledon, 1);  
close(fd);  
  
  
이제 Android 영역까지 확장해보면 User mode는 App 영역과 Service 영역으로 나뉘게 된다.  
(Service 영역에서는 Surface flinger, Audio flinger, better service 등 여러가지를 제공한다)  
  
이제 사용자의 app 영역에서는 HW를 조작하기 위해 Service 영역에게 우선 요청해야 한다.  
왜냐하면 아까처럼 root 권한은 Service가 가지고 있기 때문이다.  
  
하지만 여기서 다시 운영체제 수업을 기억해보면, User 영역의 각각의 Process는 고유의 주소체계를 가지게 되며(가상메모리)  
직접적으로 서로에게 통신할 수 없다. IPC를 위한 방법이 필요한 것이다.  
(mmap을 쓰면 kernel mode의 특정 메모리 영역이 user mode의 가상메모리 공간으로 매핑되어서 그곳을 통해 서로 통신할 수 있다)  
  
여기서 Android가 제공하는 방식이 Binder driver이다.  
이 녀석은 User에게도 권한이 열려있는 파일이다.  
led  /dev/binder  rw-rw-rw-  
  
따라서 User는 이 파일을 조작하여 Service 영역의 LED service와 통신한다.  
fd = open("dev/binder, O_RDWR);  
ioctl(fd, cmd=LED_ON, option);  
close(fd);  
  
순서로 따지면,  
**APP(User mode) ——> Binder driver(Kernel mode) ——> LED service(User mode) ——> LED driver(Kernel mode)**  
가 될것이다.  
  
  
여기서 App 영역은 또 layer가 나뉘게 되는데 우리가 알다시피 Android app은 JAVA로 구현되므로(요새는 죄다 Kotlin이지만!)  
C++ Native 영역과 VM(Dalvik Machine) 위의 JAVA 영역이 분리되고, JNI를 통해 서로 통신하게 되는 것이다.  
  
다시 최종 순서를 살펴보면,  
**JAVA App(JAVA) ——> VM(Dalvik) ——> JNI ——> C++ Native(User mode) ——> Binder driver(Kernel mode) ——> LED service(User mode) ——> LED driver(Kernel mode)가 될 것이다.**  
  
  
**물론 간략하게 나타낸 것으로 Android의 layer 구조는 아래처럼 복잡하다.**  
###    
  
![]({{ site.url }}{{ site.baseurl }}/assets/images/2021-06-26-Modern C++ in Android native - Binder driver/1302280253750484.jpg)  
  
사진 삭제  
사진 설명을 입력하세요.  
  
   