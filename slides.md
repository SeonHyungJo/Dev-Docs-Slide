---
theme: 'default'
title: 'Pipes'
download: true
highlighter: 'prism'
monaco: 'dev'
---

# Pipes

<div class="pt-8">
  <span @click="next" class="px-2 p-1 rounded cursor-pointer hover:bg-white hover:bg-opacity-10">
    시작하기 <carbon:arrow-right class="inline"/>
  </span>
</div>

---
layout: center
---

## Pipes

Pipe란 두 프로세스가 통신할 수 있는 통로(도관)다.

초기 UNIX 시스템에서 최초의 IPC 메커니즘 중 하나였다. 일반적으로 몇 가지 제한사항이 있지만, 프로세스가 서로 통신할 수 있는 간단한 방법을 제공한다. 

Pipe를 구현할 때는 4가지를 고려해야한다.

- 양방향 통신을 허용해야 하나, 단방향 통신으로 하나?
- 양방향 통신이 허용되는 경우 반이중(데이터는 한 번에 한 방향으로만 이동할 수 있음) 또는 전이중(데이터가 동시에 양방향으로 이동할 수 있음)인가?
- 통신 프로세스간에 관계 (예 : 부모-자식)가 존재해야하나?
- 네트워크를 통해 통신 할 수 있나? 아니면 통신 프로세스가 동일한 시스템에 있어야 하나?

---
layout: center
---

## **Ordinary Pipes**

Ordinary Pipes를 사용하면 두 프로세스가 표준 producer–consumer 방식으로 통신할 수 있다. 

Producer는 Pipe의 한쪽 끝(쓰는 쪽)에서 쓰고, Consumer는 다른 끝(읽는 쪽)에서 읽는다.

결과적으로 Ordinary Pipes는 단방향이므로 단방향 통신만 허용한다. 양방향 통신이 필요한 경우 서로 다른 방향으로 데이터를 전송하는 두 개의 Pipes를 사용한다. 

---
layout: center
---

$$
pipe(int fd[])
$$

UNIX 시스템에서 Ordinary Pipes는 이 함수를 사용하여 구성된다.

이 함수는 `int fd []` 파일 서술자에 액세스되는 Pipe를 만든다. `fd [0]` 은 파이프의 읽는 쪽이고 `fd [1]` 은 쓰는 쪽이다. UNIX는 Pipe를 특수한 유형의 파일로 취급합니다. 따라서 일반적인 `read ()` 및 `write ()` 시스템 호출을 사용하여 Pipe에 액세스할 수 있다.

Ordinary Pipe는 생성한 프로세스 외부에서 액세스할 수 없다. 
일반적으로 부모 프로세스는 Pipe를 생성하고, `fork ()`를 통해 자식 프로세스와 통신하는 Pipe를 만들어 통신한다. 

> **파일 서술자(file descriptor)**: 컴퓨터 프로그래밍 분야에서 파일 서술자(file descriptor) 또는 파일 기술자는 특정한 파일에 접근하기 위한 추상적인 키이다. 이 용어는 일반적으로 POSIX 운영 체제에 쓰인다.

---
layout: center
---

<img class="w-1/2" border="rounded" src="https://user-images.githubusercontent.com/24274424/117546193-32c63200-b064-11eb-9dc7-7678ffbb6dc5.png" alt="3-20">

그림 3.20은 fd 배열의 파일 서술자와 부모 및 자식 프로세스의 관계를 보여준다. 이 그림에서 알 수 있듯이 부모가 Pipe의 쓰기 끝 (fd [1])에 쓰는 모든 쓰기는 Pipe의 읽기 끝 (fd [0])에서 자식이 읽을 수 있다.

---
layout: center
---

## 또 다른 특징

- Windows 시스템의 Ordinary pipes를 익명 파이프라고하며 UNIX와 유사하게 작동한다. 

- Ordinary Pipes에는 UNIX 및 Windows 시스템 모두에서 통신 프로세스간에 상위-하위 관계가 필요하다. 

- Ordinary Pipe는 동일한 시스템의 프로세스 간 통신에만 사용할 수 있다.

---
layout: center
---

## **Named Pipes**

Ordinary pipes는 한 쌍의 프로세스가 통신할 수 있도록 하는 간단한 메커니즘을 제공한다. 그러나 프로세스가 서로 통신하는 동안에만 존재한다. UNIX 및 Windows 시스템 모두에서 프로세스가 통신을 마치고 종료되면 Ordinary pipes는 더 이상 존재하지 않는다.

Named pipes는 훨씬 더 강력한 통신 도구를 제공한다. 통신은 **양방향**일 수 있으며, **부모-자녀 관계가 필요하지 않는다.** Named pipes가 설정되면 **여러 프로세스 사이를 통신**할 수 있다. 실제로 일반적인 시나리오에서 Named pipes에는 여러 작성자(Writer)가 있다. 또한 Named pipes는 **통신 프로세스가 완료된 후에도 계속 존재**한다. 

UNIX 및 Windows 시스템 모두 Named Pipes를 지원하지만 구현 세부사항은 크게 다르다.

---
layout: center
---

Named pipes는 UNIX 시스템에서 FIFO라고 한다. 

일단 생성되면 파일 시스템에서 일반적인 파일로 나타난다. FIFO는 `mkfifo ()` 시스템 호출로 생성되고 일반적인 `open ()`, `read ()`, `write ()` 및 `close ()` 시스템 호출로 조작된다.

파일 시스템에서 명시적으로 삭제될 때까지 계속 존재한다. FIFO는 양방향 통신을 허용하지만 **반이중 전송**만 허용된다. 데이터가 양방향으로 이동해야하는 경우 일반적으로 두 개의 FIFO가 사용된다. 또한 통신 프로세스는 **동일한 시스템**에 있어야 한다. 기계 간 통신이 필요한 경우 소켓 (섹션 3.8.1)을 사용해야 한다.

Windows 시스템의 Named pipes는 UNIX에 비해 더 풍부한 통신 메커니즘을 제공한다. 전이중 통신이 허용되며 통신 프로세스는 동일하거나 다른 시스템에 상주할 수 있습니다. 또한 UNIX FIFO는 byte-oriented 데이터만 전송할 수 있는 반면, Windows 시스템은 byte-oriented 또는 message-oriented 데이터를 허용한다.
