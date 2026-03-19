# gws-korean-setup

Google Workspace CLI(gws) 설치 및 설정을 단계별로 안내하는 Claude Code 스킬.

npm 설치 → gcloud CLI 설치 → OAuth 인증 → 로그인 → Claude Code 스킬 설치 → 한국어 인코딩 안내까지 6단계를 커버합니다.

## 설치

```bash
npx skills add https://github.com/Canine89/gws-korean-setup-skills
```

## 사용

Claude Code에서:

```
/gws-korean-setup
```

### 옵션

```
/gws-korean-setup --skip-gcloud    # gcloud CLI 설치 건너뛰기
/gws-korean-setup --skip-skills    # Claude Code 스킬 설치 건너뛰기
```
