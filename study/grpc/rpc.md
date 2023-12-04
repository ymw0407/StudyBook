---
description: gRPC에 대해서 공부하기 이전, RPC에 대해서 익혀보고자 한다.
---

# RPC란

### RPC(Remote Procedure Call, 원격 프로시저 호출)

RPC는 별도의 원격 제어를 위한, 코딩 없이 다른 주소 공간에서 함수나 프로시저를 실행할 수 있게 하는 프로세스 간 통신기술을 말합니다.

여기서, 프로세스 간의 통신을 가능하게 해주는 것을 **IPC통신**이라고 하는데, 이에 대해서 먼저 다뤄보도록 하겠습니다.

#### IPC

우선, 프로세스에 대해서 먼저 살펴보면... 다음과 같습니다.

우리가 만드는 **프로그램**들은 보조 기억장치(하드 디스크, SSD)에 존재하며, 실행 되기를 기다리는 명령어(코드)와 정적인 데이터의 묶음입니다. 예를 들어, 현재 보고 계시는 웹 브라우저부터 Zoom, 카카오톡 모두 이에 해당하죠.

이런 프로그램들은 한 번에 여러 개 실행될 수도 있는데, 이렇게 실행된 프로그램 한 개가 각각의 **프로세스**가 되어 CPU를 차지하며 수행하는 수행 주체가 되어 동작합니다. 즉, 프로그램 한 개의 인스턴스가 프로세스라고 생각할 수 있습니다.

{% embed url="https://velog.io/@jhon3242/Process-%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80" %}
Process란 무엇인가? - 참고 사이트
{% endembed %}

이렇게 생성된 프로세스들은 독립적으로 실행되며, 다른 프로세스들에게 영향을 받지 않습니다. 이런 독립적으로 실행되는 프로세스간의 통신이 필요할 때도 있는데, 이때 필요한 것이 **IPC(Inter-Process Communication)**입니다.

예를 들어, 카메라 앱을 AndroidOS로 개발한다고 가정해봅시다. 이때, 앱에서 카메라 프리뷰를 보여주기 위해서는 Camera2와 같은 API를 사용할 것입니다. 내부적으로 좀 더 자세히 살펴보게 되면, 카메라 프리뷰를 가져오는 것도 하나의 프로세스임을 알 수 있습니다. 하지만, 프로세스는 독립적입니다. \
음... 제가 직접 Android 개발은 안해봐서 잘 모르지만! LG의 webOS는 개발해봤죠. webOS의 [`com.webos.service.camera2`](https://www.webosose.org/docs/reference/ls2-api/com-webos-service-camera2/)를 예시를 보면 다음과 같습니다. webOS에서는 Camera Preview를 제공하기 위해서, Live Camera data를 **shared memory**에 지속적으로 스트리밍하게 되고 API에서는 SystemV나 Posix를 통해서 데이터를 제공받습니다. 이렇게, 우리가 카메라 프리뷰를 볼 수 있게 만들 수 있게 되는 것이죠.

webOS에서 활용한 Shared Memory와 같은 프로세스간 통신을 IPC라고 하며, 종류는 다음과 같습니다.

* `Shared Memory`
  * 프로세스간 공유되는 메모리 영역을 만들어 사용하는 방법
  * 프로세스들은 읽기/쓰기를 통해서 공유 영역을 수정할 수 있다.
  * 주로 부모/자식 프로세스 간에 사용된다.
* `Message Passing`
  * 다른 프로세스에 message를 보내 정보를 교환하는 방법
  * 주로 작은 데이터를 교환하며 구현이 쉽다.
  * system call이 잦아 자칫 느려질 수 있다.
* `Sockets`
  * Network Communication의 Endpoint 간의 통신이다.
  * IP Address, Port를 이용해 직접 통신한다.
* `Pipe`
  * Message나 Shared Memory는 서로 다른 process간 memory에서 구조체 또는 데이터를 주고받았지만
  * Pipi는 양방향 간의 버퍼를 통해서 데이터를 읽거나 쓰는 구조이다.
  * 연속적인 byte stream 을 교환할 때 많이 사용된다.

{% embed url="https://coder-in-war.tistory.com/entry/OS-01-%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4%EA%B0%84-%ED%86%B5%EC%8B%A0-IPC-RPC" %}
IPC - 참고 사이트
{% endembed %}

#### RPC의 등장 배경

위에서 말했던 IPC의 종류 중엔 RPC도 포함되어있습니다.

RPC는 Socket이 발전된 형태인데... **Socket**이란, OSI 7 Layer 구조의 Application Layer에서 Transport Layer의 TCP나 UDP를 사용하기 위한 수단입니다. 목적지와의 통신이 컴퓨터 내부가 아니라 온라인 범위에서 이루어지기 때문에 네트워크 간 통신이라고 구분하기도 하지만, 실질적으로는 로컬 컴퓨터의 프로세스와 원격지 컴퓨터의 프로세스가 IPC 통신을 하는 것입니다.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption><p>Internet Protocol Stack의 Application Layer에서 Transport Layer와의 통신</p></figcaption></figure>

하지만, 이런 소캣을 활용한 IPC 통신에는 **발생 가능한 모든 예외 상황들을 미리 예측하고 대비하며 구현해야**는 문제가 있었습니다. 이를 해결하고자, RPC(Remote Procedure Call)라는 기술이 등장합니다. 이름 그대로 네트워크로 연결된 서버 상의 프로시저(함수, 메서드 등)를 원격으로 호출할 수 있는 기능입니다. 네트워크 통신을 위한 작업 하나하나 챙기기 귀찮으니, 통신이나 call 방식에 신경쓰지 않고 원격지의 자원을 내 것처럼 사용할 수 있죠. IDL(Interface Definication Language) 기반으로 다양한 언어를 가진 환경에서도 쉽게 확장이 가능하며, 인터페이스 협업에도 용이하다는 장점이 있습니다.

{% embed url="https://medium.com/naver-cloud-platform/nbp-%EA%B8%B0%EC%88%A0-%EA%B2%BD%ED%97%98-%EC%8B%9C%EB%8C%80%EC%9D%98-%ED%9D%90%EB%A6%84-grpc-%EA%B9%8A%EA%B2%8C-%ED%8C%8C%EA%B3%A0%EB%93%A4%EA%B8%B0-1-39e97cb3460" %}
RPC - 참고 사이트
{% endembed %}

RPC의 궁극적인 목표는 다음과 같습니다.

* 클라이언트-서버 간의 커뮤니케이션에 필요한 상세정보는 최대한 감춘다.
* 클라이언트는 일반 메소드를 호출하는 것처럼 원격지의 프로지서를 호출할 수 있다.
* 서버도 마찬가지로 일반 메소드를 다루는 것처럼 원격 메소드를 다룰 수 있다.

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption><p>출처: Geek for Geeks</p></figcaption></figure>

RPC도 위의 사진과 같이 Client-Server 모델을 사용하고 있습니다.

1. Client측에서 프로시저 호출을 위한 매개변수(parameter)을 `Client stub`에 전달.
2. Client stub에서 받은 매개변수들을 Server측에 전달하기 위해 Packet으로 변환
3. 해당 Packet을 Trasport layer(`RPC runtime`)으로 주고 Server 측에 전달.
4. Server측 Trasport layer에서 이를 전달 받아 Server stub으로 전달.
5. `Server stub`는 해당 Packet을 변환해 전달하며 Server에 Client측이 요구한 프로시저 호출.
6. Server는 요구한 프로시저를 처리하고 결과 값을 다시 `Server stub`에 전달
7. `Server stub`는 결과 값을 Packet으로 만들어 Transport layer를 통해 Client에 전달.
8. `Client stub`는 전달 받은 Packet에서 결과 값을 추출해 Client측이 결과 값 획득

위에서 Stub, RPC Runtime 등의 생소한 단어들이 있는데, 자세히 살펴보도록 하겠습니다.

#### Stub

위의 과정을 살펴보다 보면 stub이라는 개념이 생소할 수도 있습니다. stub이란 Client와 Server 간의 매개 변수와 같은 데이터를 각 머신에 맞게 변환해주는 코드의 조각입니다. 만약 stub이 없이 포인터가 Client측에서 전달되는 매개변수로 사용되면 어떻게 될까요? Client와 Server는 전혀 다른 메모리 공간을 사용하고 있기 때문에 Client측 에서 가리키던 포인터가 Server측으로 넘어가게 되면서 전혀다른 곳을 가리키고 있게 될 것입니다. 또 만약 Client는 리틀 엔디안 형식을 사용하지만 Server측은 빅 엔디안 형식을 사용한다면 어떨까요? 의도한 바와 전혀 다른 데이터들이 전달이 될 것 입니다. 때문에 각 데이터들은 stub을 거쳐 Packet의 형태로 전달이 되게 됩니다.

#### RPC runtime

`RPC runtime`이라는 부분도 보입니다. `RPC runtime`은 RPC 사용을 위해 네트워크 통신에 사용되는 함수와 루틴을 모아놓은 라이브러리입니다. 주로 네트워크 간 주고받은 Packet들을 관리하고 전송, 승인, 라우팅 및 암호화, 오류 처리 등 다양한 작업이 처리 가능합니다.

{% embed url="https://hackyboiz.github.io/2021/10/22/poosic/rpc/" %}
RPC - 참고 자료
{% endembed %}

#### 일반 Server-Client Model과의 차이점

RPC 서버는 일반 서버-클라이언트 모델의 서버와는 몇 가지 다른 점이 있습니다.

일반 서버-클라이언트 모델에서는 클라이언트가 서버에 직접 연결하여 요청(request)을 보내고, 서버는 해당 요청을 받아 처리한 후 클라이언트에게 응답(response)을 보내는 방식으로 동작합니다.

반면 RPC 서버는 클라이언트와 서버 간의 통신에서 Remote Procedure Call (RPC) 프로토콜을 사용하기 때문에, 클라이언트가 서버의 메소드(method)를 로컬에서 호출하는 것과 유사한 방식으로 동작하며, 클라이언트는 서버의 메소드를 직접 호출하지 않고, 클라이언트 스텁(client stub)을 통해 서버의 메소드를 호출합니다.

서버는 클라이언트로부터 요청을 받아 해당 메소드를 실행한 후, 결과를 클라이언트에게 응답합니다.즉, RPC 서버는 일반 서버-클라이언트 모델의 서버보다는 높은 수준의 추상화(abstraction)를 제공하며, 클라이언트-서버 간의 통신을 쉽게 처리할 수 있도록 도와줍니다.

이러한 RPC 프로토콜은 대부분의 프로그래밍 언어에서 지원되며, 다양한 RPC 프레임워크가 존재합니다.

{% embed url="https://loveinside79.tistory.com/246" %}
RPC - 참고 자료
{% endembed %}
