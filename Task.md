# PlantUML Builder — 작업 기록 (Task.md)

다른 컴퓨터에서 이어서 작업할 수 있도록 현재 상태를 정리한 문서.

## 개요
- 브라우저에서 동작하는 **단일 파일** PlantUML 다이어그램 빌더. 노드를 드래그·연결하며 시각적으로 그리면 PlantUML 코드가 실시간 생성된다.
- 전체 앱이 [`index.html`](index.html) 하나에 들어있다 (외부 JS/CSS 의존성 없음). 빌드 단계 없음.
- 지원 다이어그램: **클래스 / 시퀀스 / 로버스트니스 / 스테이트** (상단 탭으로 전환).

## 배포 (GitHub Pages)
- 저장소: https://github.com/puos/pumlBuilder (visibility: **public**)
- 공개 URL: **https://puos.github.io/pumlBuilder/**
- Pages 설정: `main` 브랜치 `/`(루트), `index.html`이 진입점.
- **배포 방법 = `main`에 push 하면 끝.** GitHub Pages가 1~2분 내 자동 재빌드한다. 별도 버튼/액션 없음.

## 로컬에서 미리보기
- **가장 간단: `index.html`을 브라우저로 직접 열기**(`file://`). 단일 정적 파일이라 서버가 필요 없다. (미리보기 버튼만 plantuml.com 호출 → 인터넷 필요.)
- Claude Code의 preview 도구로 띄우려면 [`.claude/launch.json`](.claude/launch.json)의 python 정적 서버(`python -m http.server 8123`) 설정을 사용한다. 이 파일은 **공용 설정이라 커밋한다**(개인 설정 `settings.local.json`만 .gitignore로 제외).

## 파일 구조
- `index.html` — 앱 전체 (HTML + CSS + JS)
- `README.md` — 소개
- `LICENSE` — MIT (Copyright (c) 2026 puos)
- `Task.md` — 이 문서
- `.claude/launch.json` — 로컬 프리뷰 서버 설정 (공용, 커밋함)

## index.html 코드 지도 (주요 위치)
- `DB` / `D()` — 다이어그램별 상태(nodes/edges 또는 parts/msgs)
- `REL` — 관계 종류별 화살표/장식 정의
- `addNode`, `drawNode`, `drawEdge`, `render`, `renderSequence` — 렌더링
- `fitCanvas()` — 내용에 맞춰 SVG 크기 자동 확장
- `genCode()` — 상태 → PlantUML 코드 생성 (+ 끝에 `'$pos` 좌표 주석)
- `posKey()` — 노드 → 좌표 주석 키
- **PUML import 영역**: `parsePuml`, `detectPumlType`, `parseClass/parseState/parseRobust/parseSequence`, `splitEdge`, `arrowToRel`, `extractPos`, `layoutNodes`
- `plantumlUrl()` / `pEnc`/`p3`/`p6` — 미리보기 URL 인코딩
- `persist()/restore()` — localStorage 자동 저장/복원 (키: `pumlbuilder`)

## 이번 작업에서 한 것 (완료)
1. **GitHub Pages 배포** — index.html을 루트에 두고 Pages 활성화.
2. **불러오기를 JSON → PUML로 변경**: `genCode`의 역방향 파서 추가. 파일 내용으로 다이어그램 종류 자동 감지 후 노드/엣지/메시지 복원, 해당 탭 자동 전환. (JSON 폴백은 사용자 요청으로 제거 — 불러오기/내보내기 모두 puml만.)
3. **노드 위치 보존**: 내보낼 때 각 노드 좌표를 `'$pos <alias> <x> <y>` 주석으로 `@enduml` 뒤에 저장. 불러올 때 복원 → 배치 유지. 좌표 주석은 PlantUML 렌더링에선 무시됨. 좌표 없는 외부 puml은 크기-인식 격자로 겹치지 않게 배치(`layoutNodes`).
4. **미리보기 400 에러 수정**: hex 인코딩(1바이트=2글자) → URL 과대 → "Request header is too large". PlantUML 표준 **deflate-raw + base64**(브라우저 내장 `CompressionStream`)로 교체하고, 전송 시 `'$pos` 주석 제거. `CompressionStream` 미지원 시 hex 폴백.
5. **캔버스 자동 확장**: SVG가 2200×1500 고정이라 가장자리 노드가 잘리던 문제 → `fitCanvas()`로 매 렌더마다 내용+여백에 맞춰 크기 조정(화면보다 작아지진 않음).

관련 커밋: 배포 / puml import / JSON 제거 / 위치 보존(`fd6cd88`) / 미리보기 인코딩(`ca9fdc7`) / 캔버스 자동확장(`c1acf99`).

## 알려진 제약 / 다음에 할 만한 것
- **기존에 내보낸 옛 .puml에는 좌표 주석이 없어** 불러오면 격자 배치됨 → 한 번 다시 내보내면 그 뒤부터 위치 보존.
- 시퀀스는 좌표 개념이 없어 참여자 **순서**만 보존(위치 주석 미사용).
- PUML 파서는 **이 앱이 생성하는 형식** 위주로 동작. 일반 PlantUML의 모든 문법(패키지, note, group/alt, 멀티 initial/final 등)은 미지원 → 필요 시 확장.
- 매우 큰 다이어그램은 deflate 후에도 URL이 길어질 수 있음 → 정 안 되면 plantuml POST 방식 고려.
- 아이디어: 다이어그램 4종을 한 파일에 함께 저장/불러오기, undo/redo, PNG 직접 내보내기, 좌표 주석 on/off 옵션.

## 다음에 이어서 할 것 (미결 — 결정 대기)
**미리보기 레이아웃이 캔버스와 다른 문제.**
- 원인: 미리보기 버튼은 텍스트만 plantuml.com에 보내고, PlantUML이 **자체 자동 레이아웃(Graphviz)** 으로 위치를 새로 계산한다. 우리가 정한 좌표(`'$pos` 주석)는 PlantUML이 무시 → 빌더 배치 ≠ 미리보기 배치. 관계·레이블 많으면 겹쳐 보임.
- 정해야 할 방향 (둘 중 택, 사용자와 상의 중):
  1. **(추천) WYSIWYG**: 미리보기/내보내기를 plantuml.com 대신 **빌더가 그리는 캔버스 SVG를 그대로 SVG/PNG로 내보내기**로 변경 → 화면과 100% 일치.
  2. PlantUML 렌더 유지 + 레이아웃 힌트로 완화: `left to right direction`, `skinparam nodesep/ranksep`, `hide empty members`, `skinparam linetype ortho` 등 (그래도 완전 일치는 안 됨).
- 아직 코드 변경 없음. 사용자가 방향 결정하면 진행.
