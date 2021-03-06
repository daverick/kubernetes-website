---
title: 파드(Pod) 개요
content_template: templates/concept
weight: 10
card:
  name: concepts
  weight: 60
---

{{% capture overview %}}
이 페이지는 쿠버네티스 객체 모델 중 가장 작은 배포 가능한 객체인 `파드` 에 대한 개요를 제공한다.
{{% /capture %}}


{{% capture body %}}
## 파드에 대해 이해하기

*파드* 는 쿠버네티스의 기본 구성 요소이다. 쿠버네티스 객체 모델 중 만들고 배포할 수 있는 가장 작고 간단한 단위이다. 파드는 {{< glossary_tooltip term_id="cluster" >}} 에서의 Running 프로세스를 나타낸다. 

파드는 애플리케이션 컨테이너(또는, 몇몇의 경우, 다중 컨테이너), 저장소 리소스, 특정 네트워크 IP 그리고, {{< glossary_tooltip text="container" term_id="container" >}} 가 동작하기 위해 만들어진 옵션들을 캡슐화 한다.
파드는 배포의 단위를 말한다. 아마 단일 컨테이너로 구성되어 있거나, 강하게 결합되어 리소스를 공유하는 소수의 컨테이너로 구성되어 있는 *쿠버네티스에서의 애플리케이션 단일 인스턴스* 를 의미함.

[도커](https://www.docker.com)는 쿠버네티스 파드에서 사용되는 가장 대표적인 컨테이너 런타임이지만, 파드는 다른 컨테이너 런타임 역시 지원한다.


쿠버네티스 클러스터 내부의 파드는 주로 두 가지 방법으로 사용된다.

* **단일 컨테이너만 동작하는 파드**. "단일 컨테이너 당 한 개의 파드" 모델은 쿠버네티스 사용 사례 중 가장 흔하다. 이 경우, 한 개의 파드가 단일 컨테이너를 감싸고 있다고 생각할 수 있으며, 쿠버네티스는 컨테이너가 아닌 파드를 직접 관리한다고 볼 수 있다.
* **함께 동작하는 작업이 필요한 다중 컨테이너가 동작하는 파드**. 아마 파드는 강하게 결합되어 있고 리소스 공유가 필요한 다중으로 함께 배치된 컨테이너로 구성되어 있을 것이다. 이렇게 함께 배치되어 설치된 컨테이너는 단일 결합 서비스 단위일 것이다. 한 컨테이너는 공유 볼륨에서 퍼블릭으로 파일들을 옮기고, 동시에 분리되어 있는 "사이드카" 컨테이너는 그 파일들을 업데이트 하거나 복구한다. 파드는 이 컨테이너와 저장소 리소스들을 한 개의 관리 가능한 요소로 묶는다.


[쿠버네티스 블로그](http://kubernetes.io/blog)에는 파드 사용 사례의 몇 가지 추가적인 정보가 있다. 더 많은 정보를 위해서 아래 내용을 참조하길 바란다.

  * [분산 시스템 툴킷: 복합 컨테이너를 위한 패턴](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns)
  * [컨테이너 디자인 패턴](https://kubernetes.io/blog/2016/06/container-design-patterns)


각각의 파드는 주어진 애플리케이션에서 단일 인스턴스로 동작을 하는 것을 말한다. 만약 애플리케이션을 수평적으로 스케일하기를 원하면(예를 들면, 다중 인스턴스 동작하는 것), 각 인스턴스 당 한 개씩 다중 파드를 사용해야 한다. 쿠버네티스에서는, 일반적으로 이것을 _복제_ 라고 한다. 복제된 파드는 주로 컨트롤러라고 하는 추상화 개념의 그룹에 의해 만들어지고 관리된다. 더 많은 정보는 [파드와 컨트롤러](#pods-and-controllers)를 참고하길 바란다.



## 어떻게 파드가 다중 컨테이너를 관리하는가

파드는 결합도가 있는 단위의 서비스를 형성하는 다중 협력 프로세스(컨테이너)를 지원하도록 디자인 되었다. 파드 내부의 컨테이너는 자동으로 동일한 물리적 또는 가상의 머신의 클러스터에 함께 배치되고 스케쥴된다. 컨테이너는 리소스와 의존성 공유, 다른 컨테이너와의 통신 그리고 언제,어떻게 조절하는지를 공유할 수 있다.

단일 파드 내부에서 함께 배치되고 관리되는 컨테이너 그룹은 상대적으로 심화된 사용 예시임에 유의하자. 컨테이너가 강하게 결합된 특별한 인스턴스의 경우에만 이 패턴을 사용하는게 좋다. 예를 들어, 공유 볼륨 내부 파일의 웹 서버 역할을 하는 컨테이너와 원격 소스로부터 그 파일들을 업데이트하는 분리된 "사이드카" 컨테이너가 있는 경우 아래 다이어그램의 모습일 것이다.


{{< figure src="/images/docs/pod.svg" alt="example pod diagram" width="50%" >}}

몇몇의 파드는 {{< glossary_tooltip text="init containers" term_id="init-container" >}} 뿐만 아니라 {{< glossary_tooltip text="app containers" term_id="app-container" >}} 도 가진다. 초기 컨테이너는 앱 컨테이너 시작이 완료되기 전에 동작한다.

파드는 같은 파드 안에 속한 컨테이너에게 두 가지 공유 리소스를 제공한다. *네트워킹* 과 *저장소*.

#### 네트워킹

각각의 파드는 유일한 IP주소를 할당 받는다. 한 파드 내부의 모든 컨테이너는 네트워크 네임스페이스와 IP주소 및 네트워크 포트를 공유한다. *파드 안에 있는* 컨테이너는 다른 컨테이너와 `localhost`를 통해서 통신할 수 있다. 특정 파드 안에 있는 컨테이너가 *파드 밖의* 요소들과 통신하기 위해서는, 네트워크 리소스를 어떻게 쓰고 있는지 공유 해야 한다(예를 들어 포트 등).

#### 저장소

파드는 공유 저장소 집합인 {{< glossary_tooltip text="Volumes" term_id="volume" >}} 을 명시할 수 있다. 파드 내부의 모든 컨테이너는 공유 볼륨에 접근할 수 있고, 그 컨테이너끼리 데이터를 공유하는 것을 허용한다. 또한 볼륨은 컨테이너가 재시작되어야 하는 상황에도 파드 안의 데이터가 영구적으로 유지될 수 있게 한다. 쿠버네티스가 어떻게 파드 안의 공유 저장소를 사용하는지 보려면 [볼륨](/docs/concepts/storage/volumes/)를 참고하길 바란다.

## 파드 작업

직접 쿠버네티스에서 싱글톤 파드이더라도 개별 파드를 만들일이 거의 없을 것이다. 그 이유는 파드가 상대적으로 수명이 짧고 일시적이기 때문이다. 파드가 만들어지면(직접 만들거나, 컨트롤러에 의해서 간접적으로 만들어지거나), 그것은 클러스터의 {{< glossary_tooltip term_id="node" >}} 에서 동작할 것이다. 파드는 프로세스가 종료되거나, 파드 객체가 삭제되거나, 파드가 리소스의 부족으로 인해 *제거되거나*, 노드에 장애가 생기지 않는 한 노드에 남아있는다. 

{{< note >}}
파드 내부에서 재시작되는 컨테이너를 파드와 함께 재시작되는 컨테이너로 혼동해서는 안된다. 파드는 자기 스스로 동작하지 않는다. 하지만 컨테이너 환경은 그것이 삭제될 때까지 계속 동작한다.
{{< /note >}}

파드는 스스로 자신을 치료하지 않는다. 만약 파드가 스케줄링된 노드에 장애가 생기거나, 스케쥴링 동작이 스스로 실패할 경우 파드는 삭제된다. 그와 비슷하게, 파드는 리소스나 노드의 유지 부족으로 인해 제거되는 상황에서 살아남지 못할 것이다.
쿠버네티스는 상대적으로 일시적인 파드 인스턴스를 관리하는 작업을 처리하는 *컨트롤러* 라고 하는 고수준의 추상적 개념을 사용한다. 즉, 파드를 직접적으로 사용가능 하지만, 컨트롤러를 사용하여 파드를 관리하는 것이 쿠버네티스에서 훨씬 더 보편적이다. 쿠버네티스가 어떻게 파드 스케일링과 치료하는지 보려면 [파드와 컨트롤러](#pods-and-controllers)를 참고하길 바란다.

### 파드와 컨트롤러

컨트롤러는 다중 파드를 생성하고 관리해 주는데, 클러스터 범위 내에서의 레플리케이션 핸들링, 롤아웃 그리고 셀프힐링 기능 제공을 한다. 예를 들어, 만약 노드가 고장났을 때, 컨트롤러는 다른 노드에 파드를 스케줄링 함으로써 자동으로 교체할 것이다.  

한 가지 또는 그 이상의 파드를 보유한 컨트롤러의 몇 가지 예시.

* [디플로이먼트](/docs/concepts/workloads/controllers/deployment/)
* [스테이트풀 셋](/docs/concepts/workloads/controllers/statefulset/)
* [데몬 셋](/docs/concepts/workloads/controllers/daemonset/)

일반적으로, 컨트롤러는 책임을 지고 제공한 파드 템플릿을 사용한다.

## 파드 템플릿
파드 템플릿은 [레플리케이션 컨트롤러](/docs/concepts/workloads/controllers/replicationcontroller/), [잡](/docs/concepts/jobs/run-to-completion-finite-workloads/), [데몬 셋](/docs/concepts/workloads/controllers/daemonset/)과 같은 다른 객체를 포함하는 파드 명세서이다. 컨트롤러는 파드 템플릿을 사용하여 실제 파드를 만든다.
아래 예시는 메시지를 출력하는 컨테이너를 포함하는 파드에 대한 간단한 매니페스트이다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
```

모든 레플리카의 현재 원하는 상태를 지정하는 대신, 파드 템플릿은 쿠키 틀과 같다. 쿠키가 한 번 잘리면, 그 쿠키는 쿠키 틀과 더이상 관련이 없다. 양자 얽힘이 없는 것이다. 그 이후 템플릿을 변경하거나 새로운 템플릿으로 바꿔도 이미 만들어진 파드에는 직접적인 영향이 없다. 마찬가지로, 레플리케이션 컨트롤러에 의해 만들어진 파드는 아마 그 이후 직접 업데이트될 수 있다. 이것은 모든 컨테이너가 속해있는 파드에서 현재 원하는 상태를 명시하는 것과 의도적으로 대비가 된다. 이러한 접근은 시스템의 의미를 철저히 단순화하고 유연성을 증가시킨다.

{{% /capture %}}

{{% capture whatsnext %}}
* [파드](/docs/concepts/workloads/pods/pod/)의 다른 동작들을 더 배워보자.
  * [파드 종료](/docs/concepts/workloads/pods/pod/#termination-of-pods)
  * [파드 라이프사이클](/ko/docs/concepts/workloads/pods/pod-lifecycle/)
{{% /capture %}}
