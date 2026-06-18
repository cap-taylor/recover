# 🔧 재난 복구 부트스트랩 (포맷된 컴퓨터 → 전체 복구)

> **이 문서의 목적**: 컴퓨터를 완전히 포맷했을 때, **아무것도 없는 맨바닥에서 시작하는 단 하나의 진입점.**
> 진짜 복구 가이드(`SETUP.md`)·스크립트·데이터는 전부 Google Drive 백업 안에 있다.
> 이 README는 그 백업을 **손에 쥐기까지의 최초 몇 동작**만 알려준다. 그 다음은 전부 자동 + Claude Code가 처리한다.
>
> ⚠️ **민감정보 없음**: 이 repo는 public이다. 비밀번호·키·토큰은 절대 여기 없다.
> 실제 비밀(SSH 키·.env·DB 비번)은 Google Drive 백업 tar 안에 있고, 아래 절차가 그걸 꺼낸다.
> **Google Drive 계정 로그인 정보만큼은 사용자가 따로 기억하고 있어야 한다** (이건 어디에도 저장 안 함).

---

## 핵심: 왜 이 문서가 따로 필요한가 (닭-달걀)

복구에 필요한 모든 것(SETUP 가이드, 복구 스크립트, SSH 키, 전역 설정)이 **Google Drive 백업 안**에 있다.
그런데 그 백업을 받으려면 "rclone 설치 + Drive 인증 + 다운로드"를 먼저 해야 한다 —
**그 방법이 적힌 문서(SETUP.md)조차 백업 안에 있다.** 받기 전엔 못 본다.

GitHub의 3개 프로젝트(Mill/Harvest/Crawl)는 **private**이라, clone하려면 SSH 키가 필요한데 그 키도 백업 안에 있다.
→ 그래서 **백업 밖, 인증 없이 볼 수 있는 이 public 진입점**이 하나 필요하다. (폰으로도 볼 수 있게.)

---

## Step 0: 윈도우 측 사전 준비 (포맷 직후 자주 막히는 곳)

1. **BIOS에서 가상화 활성화** — Intel `VT-x` / AMD `SVM`(AMD-V). 포맷·CMOS 초기화로 꺼져 있으면 WSL이 안 깔린다.
2. **윈도우 업데이트 최신화** — 구버전은 `wsl --install` 한 줄 설치가 안 된다.
3. PowerShell(**관리자 권한**)에서:
   ```powershell
   wsl --install -d Ubuntu-22.04
   ```
4. **🔴 재부팅** — 첫 설치는 재부팅해야 Ubuntu가 처음 뜬다. 재부팅 후 Ubuntu가 사용자명/비번을 물으면 설정.

---

## Step 1: WSL 안에서 — 백업을 손에 쥐는 4동작

Ubuntu 터미널에서 (이 4개만 사람이 직접 친다):

```bash
# 1) rclone 설치
curl https://rclone.org/install.sh | sudo bash

# 2) Google Drive 연결 (브라우저 OAuth — 본인 Google 계정 로그인)
rclone config
#    → n (new) → name: gdrive → storage: drive → 나머지 기본값(Enter) → 브라우저 인증 → y

# 3) 백업 다운로드 (SETUP.md + restore_db.sh + DB덤프 + 설정tar 전부 들어 있음)
mkdir -p ~/backups
rclone copy gdrive:backups/db ~/backups --progress

# 4) 전자동 복구 시작
bash ~/backups/restore_db.sh --auto
```

`restore_db.sh --auto`가 알아서: 설정 tar 풀기(SSH 키·.env·전역 지침 복원) → DB 2개 복구 → 3프로젝트 git clone → pip 설치 → crontab 복원.

> 받은 폴더에 `SETUP.md`가 같이 있다. 자동 복구가 막히면 `~/backups/SETUP.md`를 열어 수동 단계(Step 0~12)를 따라가면 된다.

---

## Step 2: 여기서부터 Claude Code에게 맡긴다

자동 복구가 끝나면 `~/.claude/`(전역 지침)와 3프로젝트 코드+`CLAUDE.md`가 디스크에 복원된 상태다.
이제 Claude Code를 깔고 띄우면, Claude가 그 지침들을 읽고 나머지를 이어서 처리할 수 있다.

```bash
# Claude Code 설치 (네이티브 인스톨러)
curl -fsSL https://claude.ai/install.sh | bash

# uv/uvx 설치 (DB MCP 서버 실행에 필수 — 없으면 Claude가 DB를 못 봄)
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.bashrc

# Harvest 폴더에서 Claude 실행 → CLAUDE.md 자동 로드
cd ~/MyProjects/Harvest && claude
```

그 다음 Claude에게: **"`~/backups/SETUP.md`를 읽고 남은 복구 단계(systemd 파이프라인·검증)를 마저 진행해줘"** 라고 시키면 된다.

> ⚠️ **Claude Code 설치 ≠ Claude가 다 안다.** Claude는 위 자동 복구가 끝나 `~/.claude/`와 프로젝트 코드가 생긴 *뒤에야* 컨텍스트를 갖는다.
> 그 전(맨바닥)엔 Claude도 빈손이라, **Step 0~1의 첫 동작들은 반드시 사람이 직접** 해야 한다.

---

## 기억해야 할 단 하나

이 모든 게 굴러가려면 **Google Drive 계정 로그인**(이메일 + 비번 + 2FA)만큼은 사용자가 갖고 있어야 한다.
그것만 있으면 → 이 README → 4동작 → 전체 복구. 나머지는 백업이 들고 있다.

---

*복구 대상 시스템: WSL2 Ubuntu-22.04 / PostgreSQL(naver+wmaster) / 3프로젝트(Mill·Harvest·Crawl) / Harvest systemd 자동화 파이프라인*
*최신 SETUP 가이드 SSOT: Harvest repo `scripts/automation/setup_template.md` → 매일 자동 생성되어 `gdrive:backups/db/SETUP.md`로 sync*
