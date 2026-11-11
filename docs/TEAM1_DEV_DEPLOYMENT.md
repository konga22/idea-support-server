# team1 dev 배포 실행 안내

이 저장소는 [`MinQDU014/idea-support-server`](https://github.com/MinQDU014/idea-support-server)의 학생 포크다. 2026-11-11에 수업용 `dev-registry.yaml`을 가져와 `team1`로만 게시·배포되도록 대상 검사를 강화했다. 교수님 원본 저장소에 쓰기 권한이 없으므로 이 포크의 `dev` 브랜치에서 실습한다.

## 배포 경로

`dev` 대상 PR은 `Dev Lint and Test`만 실행한다. `dev`에 반영된 push는 테스트 성공 후 Jib으로 `docker.minq.live/team1/idea-support-server`에 `dev-<커밋 SHA>`와 `latest`를 게시하고, `https://daelim.minq.live/_cd/team1`에 배포를 요청한다. 마지막으로 `https://daelim.minq.live/team1/common/health`가 HTTP 200, `status: ok`, `database: up`인지 검사한다. PR 생성만으로 앱이 바뀌지는 않는다.

## GitHub Environment 설정

저장소 Settings → Environments → `dev`에서 다음 값을 설정한다. Environment는 `dev` 브랜치만 사용하도록 제한한다.

| 종류 | 이름 | 값 또는 출처 |
| --- | --- | --- |
| Variable | `DEV_REGISTRY_HOST` | `docker.minq.live` |
| Variable | `DEV_REGISTRY_NAMESPACE` | `team1` |
| Variable | `DEV_DEPLOY_URL` | `https://daelim.minq.live/_cd/team1` |
| Secret | `DEV_REGISTRY_USER` | 교수님이 team1에 전달한 레지스트리 계정 |
| Secret | `DEV_REGISTRY_PASSWORD` | 해당 계정의 비밀번호 |
| Secret | `DEV_DEPLOY_TOKEN` | team1 배포 토큰 |

2026-11-11 기준 포크의 Variable 3개, Secret 3개, `dev` 브랜치 제한을 설정했다. 실제 값은 이 문서나 커밋에 넣지 않았다. 기존 AWS 키는 ECR용이므로 이 세 Secret을 대신하지 않는다.

## 실행 및 확인

1. Secret 3개의 이름이 위 표와 정확히 일치하는지 확인한다.
2. 이 포크의 team1 변경 PR에서 테스트가 통과하는지 확인한다. PR의 `publish` job은 건너뛰어야 한다.
3. PR을 이 포크의 `dev`에 반영한다. 이 push가 실제 team1 배포를 시작한다.
4. Actions의 `Dev Registry`에서 `Dev Lint and Test`와 `Push dev image to Registry`가 모두 성공했는지 확인한다. 요약에 `team1` 이미지 주소와 digest가 있어야 한다.
5. 위 헬스 URL의 JSON에서 `status`, `database`, `aiMode`, `seedReady`를 확인한다. 헬스가 성공해도 Gemini의 실제 생성 요청은 별도 검증이다.

값 검사에서 실패하면 이미지 게시 전 멈춘다. `401`은 배포 토큰, 레지스트리 로그인 실패는 레지스트리 계정·비밀번호, `database != up`은 팀 DB 설정을 확인한다. DB와 VM은 8팀 공유 자원이므로 볼륨 삭제나 서버 전체 재설치는 하지 않는다.

수업 자료의 절차와 인프라 설명은 원본 저장소의 [`docs/DEV_REGISTRY.md`](https://github.com/MinQDU014/idea-support-server/blob/dev/docs/DEV_REGISTRY.md) 및 [`deploy/classroom-cd/README.md`](https://github.com/MinQDU014/idea-support-server/blob/dev/deploy/classroom-cd/README.md)를 참고한다.

## 2026-11-11 실습 결과

- [team1 변경 PR #1](https://github.com/konga22/idea-support-server/pull/1)의 `Dev Lint and Test`가 성공했고 `publish`는 예상대로 건너뛰었다.
- PR을 이 포크의 `dev`에 합친 커밋은 `b2856f8f15822f051df3145a523fa81313f0b587`이다.
- [dev push 실행 42954365300](https://github.com/konga22/idea-support-server/actions/runs/42954365300)에서 테스트, 대상 값 검사, team1 이미지 게시, 앱 교체, 외부 헬스체크가 모두 성공했다. 이미지 이름은 `docker.minq.live/team1/idea-support-server`이며 버전 태그는 `dev-b2856f8f15822f051df3145a523fa81313f0b587`와 `latest`다.
- 2026-11-11 12:46 KST에 외부에서 `GET https://daelim.minq.live/team1/common/health`를 다시 호출했다. HTTP 200, `{"status":"ok","database":"up","aiMode":"mock","seedReady":true}`를 받았다. 이는 서버와 DB 준비 상태를 확인한 결과다. Gemini 실제 생성 요청은 이번 실습에서 시험하지 않았다.
