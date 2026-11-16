# test-app Helm 통합 차트

이 저장소는 MSA 애플리케이션, Kubernetes 정책, SUSE Virtualization(KubeVirt)의
VirtualMachineInstance(VMI)를 배포하기 위한 Helm 통합 차트 예제를 제공합니다.

## 차트 구성

```text
platform/
└── argocd/                       # 플랫폼 Argo CD 차트와 설정값
test-app/
├── Chart.yaml
├── values.yaml
└── charts/
    ├── msa-application/          # 배포, 서비스, 설정 맵, 인그레스, 수평 파드 자동 확장
    ├── policy/                   # MSA 및 가상화 네트워크 정책, PSA 레이블
    └── vm/
        └── templates/            # SUSE Virtualization용 VMI
```

## 사전 요구 사항

- Kubernetes 및 Helm 3
- VMI를 활성화하기 전에 SUSE Virtualization/KubeVirt 설치
- NetworkPolicy를 활성화하는 경우 Kubernetes NetworkPolicy를 적용하는 CNI
- PSA 레이블을 활성화하는 경우 Pod Security Admission(PSA)이 설정된 네임스페이스

## 설치

```sh
helm dependency update ./test-app
helm upgrade --install test-app ./test-app --namespace test-app --create-namespace
```

애플리케이션 및 정책 차트는 기본적으로 활성화됩니다. NetworkPolicy,
네임스페이스 생성 및 PSA 레이블, VMI는 선택적으로 활성화할 수 있습니다.
차트가 PSA 레이블을 포함한 네임스페이스를 생성하도록 하려면 릴리스를 기존
관리 네임스페이스에 설치하고 별도의 대상 네임스페이스를 지정합니다.

```sh
helm upgrade --install test-app ./test-app \
  --namespace helm-system \
  --set global.targetNamespace=test-app \
  --set policy.namespace.create=true
```

Argo CD는 플랫폼 구성 요소로 별도 관리합니다.

```sh
helm dependency update ./platform/argocd
helm upgrade --install argocd ./platform/argocd \
  --namespace argocd --create-namespace
```

Argo CD 차트 설정은 `platform/argocd/values.yaml`에서 변경합니다. 애플리케이션
차트와 플랫폼 차트는 서로 독립적으로 설치하고 업그레이드할 수 있습니다.

차트가 네임스페이스를 소유하도록 설정할 때 대상 네임스페이스가 이미 존재하면
안 됩니다. `global.targetNamespace`가 비어 있으면 워크로드는 Helm 릴리스
네임스페이스를 사용합니다. 이 경우 네임스페이스는 차트 외부에서 생성하고
관리해야 합니다.

VMI를 미리 렌더링해 확인하려면 다음 명령을 실행합니다. 실제 적용 시에는
KubeVirt가 설치되어 있어야 합니다.

```sh
helm template test-app ./test-app \
  --namespace test-app \
  --set vm.virtualizationVmi.enabled=true
```

워크로드 이미지, 리소스, 정책, VM 사양은 `test-app/values.yaml`에서 설정합니다.
운영 환경에 적용하기 전에 예시 이미지와 설정값을 실제 값으로 바꾸세요.
NetworkPolicy의 선택자와 인그레스/이그레스 규칙은 클러스터 및 애플리케이션에
따라 다르므로, 이 차트는 트래픽 허용 여부를 임의로 가정하지 않습니다.

`ingress` 또는 `egress` 규칙이 비어 있는 NetworkPolicy를 활성화하면 해당
방향의 트래픽이 선택된 Pod에서 차단됩니다. 정책을 활성화하기 전에 규칙을
정의하고 검증하세요.
