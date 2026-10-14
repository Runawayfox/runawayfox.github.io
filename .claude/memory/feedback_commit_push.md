---
name: feedback_commit_push
description: 작업 완료 후 커밋과 푸시를 별도 지시 없이 스스로 진행한다
metadata:
  node_type: memory
  type: feedback
  originSessionId: f44d4d1e-bf4c-42e6-b3ba-29d07400c5ff
  modified: 2026-10-12T04:50:26.029Z
---

작업이 완료되면 커밋과 `git push origin master`까지 자동으로 진행한다. 별도로 "커밋하세요" 또는 "푸시하세요" 지시를 기다리지 않는다.

**Why:** 사용자가 커밋과 푸시를 각각 별도로 지시하는 것이 번거롭다고 판단해 자동 진행을 요청함.

**How to apply:** 코드 변경 작업이 완료될 때마다 커밋 메시지를 작성해 커밋하고, 바로 push까지 실행한다.
