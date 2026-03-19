# gws 설치 문제 해결 가이드

## gws CLI 관련

### `gws: command not found`

npm 글로벌 설치 경로가 PATH에 없는 경우.

```bash
# 글로벌 경로 확인
npm config get prefix
# 출력 예: /usr/local 또는 /Users/username/.npm-global

# PATH에 추가 (zsh 기준)
echo 'export PATH="$(npm config get prefix)/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

Windows의 경우:
```powershell
# 글로벌 경로 확인
npm config get prefix
# 출력 예: C:\Users\username\AppData\Roaming\npm

# 시스템 PATH에 위 경로를 추가한다 (시스템 설정 > 환경 변수)
# 추가 후 PowerShell을 새로 열기
```

### npm이 없는 경우

Node.js를 먼저 설치해야 한다.

- https://nodejs.org/ 에서 **LTS** 버전 다운로드
- macOS: `.pkg` 인스톨러 또는 `brew install node`
- Linux: 패키지 매니저 또는 nvm 사용
- Windows: `.msi` 인스톨러 실행

### EACCES 권한 오류 (macOS/Linux)

```bash
# 방법 1: sudo 사용
sudo npm install -g @googleworkspace/cli

# 방법 2: npm 글로벌 경로 변경 (권장)
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH="$HOME/.npm-global/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
npm install -g @googleworkspace/cli
```

---

## gcloud CLI 관련

### `gcloud: command not found`

install.sh 실행 후 PATH가 반영되지 않은 경우.

```bash
# 방법 1: 터미널을 새로 열기

# 방법 2: 쉘 프로파일 수동 반영
source ~/.zshrc     # zsh
source ~/.bashrc    # bash
source ~/.profile   # sh

# 방법 3: 직접 PATH 추가
echo 'source ~/google-cloud-sdk/path.zsh.inc' >> ~/.zshrc
source ~/.zshrc
```

Windows: 새 터미널(PowerShell/CMD)을 열면 대부분 해결된다. 안 되면 시스템 PATH에 `C:\Users\{사용자}\AppData\Local\Google\Cloud SDK\google-cloud-sdk\bin`을 추가한다.

### gcloud 초기화 문제

```bash
# 프로젝트 목록 확인
gcloud projects list

# 활성 프로젝트 설정
gcloud config set project {PROJECT_ID}

# 인증 상태 확인
gcloud auth list
```

---

## OAuth / 인증 관련

### `restricted_client` 경고

정상 동작. 개인용 OAuth 앱은 Google 심사를 받지 않았으므로 이 경고가 항상 표시된다.

- External 타입일 경우 **테스트 사용자**에 본인 이메일이 등록되어 있어야 로그인 가능
- 등록 방법: Google Cloud Console > API 및 서비스 > OAuth 동의 화면 > 테스트 사용자 추가

### `invalid_grant` 오류

토큰이 만료되었거나 손상된 경우.

```bash
# 재로그인
gws auth login -s gmail,drive,sheets,calendar,docs

# 그래도 안 되면 토큰 파일 삭제 후 재시도
rm ~/.config/gws/tokens.json
gws auth login -s gmail,drive,sheets,calendar,docs
```

### `PERMISSION_DENIED`

API가 활성화되지 않았거나 스코프가 부족한 경우.

```bash
# API 재활성화
gws auth setup --project {PROJECT_ID}

# 더 넓은 스코프로 재로그인
gws auth login --full
```

### `client_secret.json`이 없음

auth setup이 완료되지 않았거나 다른 경로에 저장된 경우.

```bash
# 기본 경로 확인
ls ~/.config/gws/

# 재설정
gws auth setup --login
```

---

## 한국어 인코딩 관련

### 한글이 깨져서 출력됨

```bash
# 1. locale 확인
locale
# LANG이 UTF-8이어야 한다 (예: ko_KR.UTF-8 또는 en_US.UTF-8)

# 2. locale이 UTF-8이 아닌 경우 (macOS/Linux)
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
# 영구 적용하려면 ~/.zshrc에 추가
```

### 이메일 수신자 이름이 깨짐

주소 필드(to/cc/bcc)의 한글 이름은 RFC 2047 인코딩이 필요하다:

```bash
# 한글 이름을 Base64로 인코딩
echo -n "홍길동" | base64
# 출력: 7ZmN6ri464+Z

# 결과를 RFC 2047 형식으로 조합
# =?UTF-8?B?7ZmN6ri464+Z?= <hong@example.com>
```

### JSON에서 한글이 이스케이프됨

JSON 파일 내 `\uD55C\uAE00` 같은 이스케이프는 원문 한글로 교체한다:

```json
// ❌ 잘못된 예
{"subject": "\uD68C\uC758 \uC548\uB0B4"}

// ✅ 올바른 예
{"subject": "회의 안내"}
```

자세한 내용은 `/gws-korean-encoding` 스킬을 참조한다.
