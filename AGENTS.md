# iStack

여러 언어의 서버 · 클라이언트를 한 pnpm 워크스페이스에 두고 `contract/` 로만 잇는 템플릿.
클라이언트 ↔ 서버는 HTTP(서버 코드에서 만든 openapi), 서버 ↔ 서버는 gRPC(공통 proto). 문서는 [docs/](docs/README.md).

## 에이전트 지시

- 지시는 `AGENTS.md` 에 쓴다. 같은 폴더의 `CLAUDE.md` 는 `@AGENTS.md` 한 줄로 그 파일을 불러올 뿐이다. 새 `AGENTS.md` 를 만들면 옆에 같은 `CLAUDE.md` 를 둔다.
- **가까운 `AGENTS.md` 가 우선한다.** 이 파일은 레포 전체에서 꼭 지킬 것만, 서버 · 앱마다 하나씩 있는 것(`servers/<서버>`,
  `clients/apps/<앱>`)은 그 서버 · 앱의 구조 · 명령 · 도구 · 테스트 방법을 갖는다. 둘이 다르면 서버 · 앱 것을 따른다.
- 단 채택된 ADR(`docs/decisions/`)은 영역 규칙보다 앞선다. 영역에서 다르게 하려면 ADR 을 먼저 바꾼다.
- 설명 · 세부는 `docs/` 에 쓰고 여기 되풀이하지 않는다. 개인 지시는 `CLAUDE.local.md`(gitignore) 나 `~/.claude/CLAUDE.md`.

## 꼭 지킬 것

- **계약** — HTTP API 는 서버 코드가 기준이다(ADR-0001). `contract/openapi` · `contract/gen` 같은 생성물은 손으로 고치지 않고,
  API · proto 를 바꾸면 `pnpm gen` 을 돌려 함께 커밋한다. proto 필드 번호는 바꾸거나 다시 쓰지 않는다(ADR-0002). → [docs/contracts.md](docs/contracts.md)
- **테스트** — 실제 서버를 랜덤 포트로 띄워 부르고, 외부 클라이언트만 Fake 로 바꾼다. mock 은 쓰지 않는다(ADR-0003).
  새 엔드포인트 · RPC 에는 `{METHOD}_specs` 를 더한다. → [docs/testing.md](docs/testing.md)
- **사용자 환경** — 사용자가 띄운 개발 서버(8000~8003 · 9001 · 9002 · 5280~5283 · 1420)를 끄거나 거기에 요청하지 않는다.
  확인은 `verify-local` 처럼 +100 포트 · 테스트 DB 로 따로 띄우고, 끌 때는 그 포트의 PID · 띄운 컨테이너만 끈다. → [docs/development.md](docs/development.md)
- **UI** — 화면은 [DESIGN.md](DESIGN.md) 의 토큰 · 규칙을 따른다. 토큰에 없는 색 · 크기 · 간격이 필요하면 DESIGN.md 에 먼저 더한다.
- **ADR** — 기반 패턴을 바꾸기 전에 `docs/decisions/` 에서 관련 ADR 을 찾는다. 어긋나면 번호를 들어 알리고,
  어떤 ADR 도 다루지 않는 중요한 결정은 `docs/decisions/template.md` 로 새 ADR 을 제안한다. 바꾼 결정은 지우지 않고 "대체됨(→ 번호)".

## Skills · hook

`.agents/skills/` (`.claude/skills` 는 링크). Claude Code 는 `/이름`, Codex 는 `$이름`.

- `repo-review` — diff 를 이 저장소 기준(계약 · ADR · 레이어 · 테스트 · 재사용 · 군더더기)으로 보고 P0~P3 로 적는다. 규칙은 `docs/` · `AGENTS.md` 에서 읽는다.
- `verify-local` — 바꾼 서버 · 앱을 겹치지 않는 포트로 띄워 실제 응답 · 화면 · 콘솔로 확인한다.
- `write-specs` — 엔드포인트 · RPC 를 더하거나 바꿀 때 `{METHOD}_specs`(실제 서버 · Fake)를 쓰고 가장 좁은 테스트부터 돌린다.

`.claude/hooks/check-patterns.py` 가 편집 전에 막는다: 생성물 수정, main 에 들어간 Flyway 마이그레이션 수정, API 문자열을 HTML 로 넣기,
외부 클라이언트 mock. 규칙을 바꾸면 `.claude/hooks/check-patterns.test.sh`. hook 이 없는 에이전트도 같은 규칙을 지킨다.

## 명령

```bash
pnpm dev · test · gen · typecheck · lint · format    # 루트에서 전체 (typecheck 는 gen 을 먼저, dev · test 는 infra 를 먼저)
pnpm infra · stack                                   # docker 의 Postgres · 전체 스택(이미지 + smoke)
pnpm --filter @app/<패키지> <스크립트>               # 하나만 (DB 가 필요하면 pnpm infra 먼저)
```
