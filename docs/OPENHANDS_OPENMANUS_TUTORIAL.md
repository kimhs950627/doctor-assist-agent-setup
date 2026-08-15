# 최소 명령 튜토리얼: OpenHands + OpenManus + Gemini Flash/Gemma

## 설계 목표

이 튜토리얼은 Windows에서 **Docker Desktop**과 **PowerShell**만 쓰는 흐름으로 구성됨. 설치 뒤 일상 실행은 `docker compose up -d` 한 명령이며, 사용자가 직접 입력하는 값은 `.env`의 Gemini API 키뿐임.

OpenHands는 `doctor_assist_project`의 코드를 읽고 수정·테스트·GitHub commit/push를 수행하는 웹 기반 개발 에이전트임. OpenManus는 브라우저 기반 근거 수집을 맡는 별도 리서치 워커임. 두 역할을 분리하면 웹 대시보드 또는 Telegram 처리 요청이 장시간 리서치 때문에 멈추는 것을 피할 수 있음.

## 사전 준비

### Docker Desktop

1. 웹 브라우저에서 `https://www.docker.com/products/docker-desktop/`을 엶.
2. **Download Docker Desktop** 또는 **Download for Windows**를 클릭함.
3. 다운로드한 `Docker Desktop Installer.exe`를 실행함.
4. 설치 설정에서 **Use WSL 2 instead of Hyper-V**를 선택함.
5. 설치를 완료하고 필요 시 PC를 재시작함.
6. 시작 메뉴 → **Docker Desktop**을 클릭함.
7. Docker 아이콘이 실행 중이고 Docker Desktop의 상태가 Running인지 확인함.

### Gemini API 키

1. `https://aistudio.google.com/`을 엶.
2. Google 계정으로 로그인하고 최초 약관 화면이 나타나면 동의함.
3. 좌측 사이드바의 **Get API key**를 클릭함.
4. 화면 우측 상단의 **Create API key**를 클릭함.
5. 키 이름(예: `doctor-assist-local`)을 적음.
6. Google Cloud 프로젝트 선택 화면에서 기존 프로젝트를 선택하거나 **Create a new project**를 클릭해 새 프로젝트를 만듦.
7. **Create key**를 클릭한 뒤 **Copy key** 아이콘을 클릭함.
8. 키는 다음 단계의 로컬 `.env`에만 저장함. GitHub 웹사이트, 이 문서, OpenHands 채팅, 공개 화면 공유에 붙여넣지 않음.

## 한 번만 하는 로컬 준비

PowerShell을 열고 다음 한 줄을 실행함.

```powershell
git clone https://github.com/kimhs950627/doctor-assist-agent-setup.git; cd doctor-assist-agent-setup; Copy-Item .env.example .env
```

파일 탐색기에서 새 폴더를 열고 `.env`를 메모장 또는 VS Code로 엶. `GEMINI_API_KEY=` 뒤의 placeholder만 실제 Google AI Studio 키로 바꾸고 저장함. `GEMINI_FLASH_MODEL`, `GEMMA_MODEL`, `DRY_RUN=true`는 그대로 둠.

## 일상 실행: 한 명령

같은 폴더 PowerShell에서 다음을 실행함.

```powershell
docker compose up -d
```

첫 실행 시 컨테이너 이미지를 내려받는 동안 기다림. 다음으로 브라우저 주소창에 `http://localhost:3000`을 입력해 OpenHands를 엶.

중지 명령:

```powershell
docker compose down
```

상태 확인 명령:

```powershell
docker compose ps
```

## OpenHands 클릭 설정

### Gemini Flash 프로필

1. OpenHands의 최초 설정 팝업에서 **LLM Provider** 드롭다운을 클릭하고 **Gemini**를 선택함.
2. **LLM Model** 드롭다운에서 Flash 모델을 선택함.
3. 드롭다운 목록에 없으면 **see advanced settings**를 클릭하고 **Advanced** 토글을 켬.
4. **Custom Model** 입력란에 `gemini/gemini-2.0-flash`를 입력함.
5. **API Key**에 Google AI Studio 키를 입력함.
6. **Save Changes**를 클릭함.

### Gemma 프로필

1. 좌측 또는 우측 상단의 **Settings**를 클릭함.
2. **LLM** 탭을 클릭함.
3. **Advanced**를 켬.
4. **Custom Model**에 `gemini/gemma-3-27b-it`를 입력함.
5. 동일한 API Key를 입력하고 **Save Changes**를 클릭함.
6. 대화 화면으로 돌아가 입력창 주변의 현재 프로필 이름 버튼을 클릭함.
7. 속도가 중요한 개발·디버깅은 Flash 프로필을 선택하고, 긴 반복 초안·요약은 Gemma 프로필을 선택함.

## doctor_assist_project 연결과 개발

1. OpenHands의 Repository 선택 화면에서 `kimhs950627/doctor_assist_project`를 선택함.
2. GitHub 인증을 요청하는 창이 나오면 GitHub PAT를 해당 비밀 입력란에만 입력함. 토큰을 채팅에 입력하지 않음.
3. 저장소가 열리면 `prompts/openhands_build_doctor_assist.txt` 전체를 OpenHands 채팅에 붙여넣음.
4. OpenHands가 phase 1의 계획·코드·테스트·diff를 제시하면 테스트가 성공했는지와 변경 파일을 검토함.
5. push 완료 보고에서 commit SHA와 `main` 브랜치를 확인함.
6. phase 2 이상은 사용자가 별도 승인할 때만 진행함.

## OpenManus 워커 통합 원칙

현재 Compose의 `openmanus` 서비스는 의도적으로 placeholder 컨테이너임. OpenManus 이미지/의존성은 업데이트가 잦으므로 임상 운영 전에 pin된 버전으로 빌드하고 보안 검토해야 함. `doctor_assist_project`에는 `control_tower/workers/openmanus_worker.py`를 두고, SQLite job queue가 이 별도 워커를 호출하도록 구현함.

OpenManus 출력은 원문 보존용 `openmanus_raw.json`, 정규화한 `evidence_bundle.json`, 출처 메타데이터 `sources.json`, 사람이 읽을 `summary.md`로 나눠야 함. 이 파일들은 Git이 아닌 보호된 런타임 `data/`에만 보관함. Gemini가 근거 기반 문서를 작성할 때는 `EvidenceBundle`에 없는 의학적 사실을 덧붙이지 않도록 프롬프트 제약을 둠.

## 보안과 임상 안전

- PHI, EMR 원문, 환자 사진, 식별 가능한 날짜·번호를 어떤 모델 호출·텔레그램·GitHub·로그에도 전송하지 않음.
- API 키와 PAT는 `.env` 또는 비밀 관리 UI에만 보관하고 commit 전 `git diff --cached`를 확인함.
- 외부 게시 연동은 항상 `DRY_RUN=true`로 시작함.
- 의료 조언, 감별, 약물·환자 교육 초안은 의사 검토와 임상 판단을 거쳐야 함.
- 대시보드를 인터넷에 공개하기 전에는 인증·HTTPS·접근제어를 먼저 구성함.

## 문제 해결

- `docker compose up -d`가 실패하면 Docker Desktop이 실행 중인지 확인하고 Docker Desktop → Settings → General에서 WSL 2 backend가 선택됐는지 확인함.
- `localhost:3000`이 열리지 않으면 PowerShell에서 `docker compose ps`를 실행해 `openhands`가 Up 상태인지 확인함.
- LLM 오류가 나면 OpenHands Settings → LLM에서 API Key, Custom Model 접두사 `gemini/`, 모델명을 다시 확인함.
- push 오류가 나면 GitHub 토큰 권한과 저장소 선택 상태를 확인하고, 토큰 원문을 로그나 채팅에 공유하지 않음.
