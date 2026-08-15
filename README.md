# Doctor Assist Agent Setup

`doctor_assist_project`를 OpenHands(개발·GitHub 작업)와 OpenManus(근거 수집)로 운영하기 위한 최소 명령 설치 저장소임.

## 목표

- Windows에서는 **Docker Desktop**과 **PowerShell**만 사용함.
- 매일 사용하는 실행 명령은 한 개임: `docker compose up -d`.
- 사용자가 직접 입력하는 민감 정보는 `.env`의 Gemini API 키뿐임.
- GitHub push/commit은 OpenHands 웹 UI가 수행함. PAT나 API 키를 프롬프트·소스·Git에 기록하지 않음.

## 1. 준비: 클릭으로만 설치

1. [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/) 페이지를 열고 **Download for Windows**를 클릭함.
2. 내려받은 설치 파일을 실행하고, 설정 화면에서 **Use WSL 2 instead of Hyper-V**를 선택한 뒤 설치·재시작함.
3. 시작 메뉴에서 **Docker Desktop**을 실행하고 엔진이 Running 상태가 될 때까지 기다림.
4. [Google AI Studio](https://aistudio.google.com/)에 로그인함.
5. 좌측 메뉴의 **Get API key** → **Create API key**를 클릭하고, 새 Google Cloud 프로젝트를 만들거나 기존 프로젝트를 선택한 뒤 **Create key**를 클릭함.
6. 표시된 키를 복사함. 이 키는 Gemini Flash와 Gemma 공용임. 복사한 원문을 GitHub, 채팅, 문서에 붙여넣지 않음.

## 2. 이 저장소 내려받기

PowerShell 하나만 열고 다음 **한 명령**을 실행함.

```powershell
git clone https://github.com/kimhs950627/doctor-assist-agent-setup.git; cd doctor-assist-agent-setup; Copy-Item .env.example .env
```

`.env` 파일을 메모장 또는 VS Code로 열어 `GEMINI_API_KEY=...`의 placeholder만 발급 키로 바꾸고 저장함. `.env`는 절대 커밋하지 않음.

## 3. 서비스 실행

같은 PowerShell에서 다음 **한 명령**을 실행함.

```powershell
docker compose up -d
```

첫 실행은 이미지 다운로드 때문에 시간이 걸림. 완료되면 브라우저에서 [http://localhost:3000](http://localhost:3000)을 열어 OpenHands에 접속함.

중지할 때도 명령 하나면 됨.

```powershell
docker compose down
```

상태를 볼 때는 다음을 사용함.

```powershell
docker compose ps
```

## 4. OpenHands에서 Gemini Flash 설정

1. `http://localhost:3000`의 최초 설정 창에서 **LLM Provider**를 `Gemini`로 선택함.
2. **LLM Model**에서 Flash 모델을 선택함. 목록에 없으면 **see advanced settings**를 클릭하고 **Advanced**를 켠 뒤 **Custom Model**에 `gemini/gemini-2.0-flash`를 입력함.
3. **API Key**에 `.env`에 넣은 동일한 Gemini 키를 입력함.
4. **Save Changes**를 클릭함.
5. Settings → LLM에서 두 번째 프로필을 만들고, Advanced → Custom Model에 `gemini/gemma-3-27b-it`를 입력하여 Gemma 프로필을 저장함.
6. 채팅 입력창의 프로필 선택 버튼으로 Flash/Gemma를 바꿔 사용함. 실시간 코드·임상 보조는 Flash, 반복 문서 초안은 Gemma를 우선 사용함.

## 5. doctor_assist_project 작업 시작

OpenHands에서 작업 저장소로 `kimhs950627/doctor_assist_project`를 선택/클론한 뒤, `prompts/openhands_build_doctor_assist.txt`의 내용을 채팅에 붙여넣음. OpenHands가 코드 수정·테스트·commit·push를 수행하되, 실제 외부 게시나 배포 전에 항상 결과를 검토해야 함.

## 6. OpenManus 역할

`docker-compose.yml`은 OpenManus 런타임을 별도 컨테이너로 둠. OpenHands가 doctor_assist_project에 워커를 구현할 때 이 서비스는 FastAPI/Telegram 요청 핸들러 내부가 아니라 백그라운드 job worker로만 호출해야 함. 임상 리서치 결과는 `EvidenceBundle`로 정규화하고, PHI는 어떤 외부 모델·로그·Git에도 전송하지 않음.

## 안전 원칙

- `.env`, 브라우저 세션, 로그, `data/`는 Git에 커밋하지 않음.
- 환자식별정보는 Gemini, OpenManus, Telegram, GitHub에 입력하지 않음.
- SNS·블로그 게시 기능은 `DRY_RUN=true`를 유지하고 의사 검토 후에만 수동 게시함.
- OpenHands에는 한 번에 한 단계만 요청하고, commit diff와 테스트 결과를 검토한 뒤 다음 단계를 진행함.

상세 설치 및 문제 해결은 `docs/OPENHANDS_OPENMANUS_TUTORIAL.md`를 참조함.
