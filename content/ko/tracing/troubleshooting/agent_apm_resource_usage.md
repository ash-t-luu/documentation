---
title: APM의 에이전트 리소스 사용
---


에이전트는 CPU를 사용하며, CPU 사용량은 초당 스팬 수신량과 상호 관련이 있습니다.

에이전트는 처리되지 않은 페이로드를 메모리에서 퍼버링하기 때문에 CPU가 충분하지 않으면 에이전트 프로세스 제한으로 인해 메모리 부족 문제가 발생할 수 있습니다.

## CPU 부족 문제 감지하기

CPU 사용량을 모니터링하고 CPU 부족 문제를 미리 감지하려면 에이전트용으로 구성된 [최대 CPU 백분율][1]을 `datadog.trace_agent.cpu_percent` 메트릭과 비교하세요. `datadog.trace_agent.cpu_percent` 메트릭은 코어 대비 CPU 사용량을 나타냅니다. 예를 들어, 값 `50`은 코어 절반이며, `200`은 코어 두 개와 같습니다.

[에이전트 APM 메트릭][2] 전체 목록을 참고하세요.



## 리소스 필수 요구 사항

에이전트에서 적절한 리소스 제한량을 계산하려면 `datadog.trace_agent.receiver.spans_received` 메트릭에 보고되는 초당 수신 스팬 개수를 살펴보는 것입니다.
이 메트릭 값을 기반으로 아래 테이블에서 적절한 CPU와 메모리 제한을 선택하세요.

| 초당 스팬 개수  | CPU(코어)   | 메모리(MB)  |
|----------|--------------|--------------|
| 2000       | 0.05         | 35           |
| 11 000      | 0.2          | 40           |
| 32 000      | 0.6          | 60           |
| 58 000      | 1            | 70           |
| 130 000     | 2            | 130          |

**참고**:
- 안내된 값은 에이전트 `7.39.0` 벤치마크 기반입니다.
- 벤치마크는 AWS `c5.2xlarge` 인스턴스(8 VCPU/ 16GiB RAM)에서 실행되었습니다.

[1]: /ko/tracing/troubleshooting/agent_rate_limits/#maximum-cpu-percentage
[2]: /ko/tracing/send_traces/agent-apm-metrics/