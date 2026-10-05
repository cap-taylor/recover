# 🔧 재난 복구 부트스트랩 (포맷된 컴퓨터 → 전체 복구)

> **이 문서의 목적**: 컴퓨터를 완전히 포맷했을 때, **아무것도 없는 맨바닥에서 시작하는 단 하나의 진입점.**
> 진짜 복구 가이드(`SETUP.md`)·스크립트·데이터는 전부 백업 안에 있다(Google Drive · 내장 D: · 외장 E:).
> 이 README는 그 백업을 **손에 쥐기까지의 최초 몇 동작**만 알려준다. 그 다음은 `restore_db.sh --auto` 가 한다.
>
> ⚠️ **민감정보 없음**: 이 repo는 public이다. 비밀번호·키·토큰·계정 이메일은 절대 여기 없다.
> 실제 비밀(SSH 키·.env·DB 비번·Claude 로그인)은 백업 tar 안에 있고, 아래 절차가 그걸 꺼낸다.
> **Google 계정 로그인 정보(Drive 경로를 쓸 때)만큼은 사용자가 따로 기억하고 있어야 한다.**

---

## 핵심: 왜 이 문서가 따로 필요한가 (닭-달걀)

복구에 필요한 모든 것(SETUP 가이드, 복구 스크립트, SSH 키, 전역 설정)이 **백업 안**에 있다.
그 방법이 적힌 문서(SETUP.md)조차 백업 안이라 받기 전엔 못 본다. GitHub의 프로젝트들은 전부 **private** 이라
clone 하려면 SSH 키가 필요한데 그 키도 백업 안에 있다 → 그래서 **백업 밖, 인증 없이 볼 수 있는 이 진입점**이 필요하다.

---

## Step 0: 윈도우 측 사전 준비 (사람 손 — 포맷 직후 자주 막히는 곳)

1. **BIOS에서 가상화 활성화** — Intel `VT-x` / AMD `SVM`(AMD-V). 꺼져 있으면 WSL이 안 깔린다.
2. **윈도우 업데이트 최신화** — 구버전은 `wsl --install` 한 줄 설치가 안 된다.
3. **🔴 Windows 사용자 이름 = `me`** 로 만든다. 스크립트·터미널 프로필이 `C:\Users\me` 를 전제한다.
4. PowerShell(**관리자 권한**):
   ```powershell
   wsl --install -d Ubuntu-22.04
   ```
5. **🔴 재부팅** → Ubuntu 가 처음 뜨면 **사용자 이름 = `dino`** 로 만든다 (cron·프로필·심링크가 `/home/dino` 를 전제 — 다르면 복구가 첫 단계에서 이유를 말하고 멈춘다).
6. **Windows Terminal 을 한 번 열었다 닫는다** — 설정 폴더가 그때 생겨야 프로필 복원이 된다.
7. 데스크탑이면 Windows 앱 (PowerShell, 관리자 아님):
   ```powershell
   winget install -e --id Google.Chrome
   winget install -e --id Tailscale.Tailscale    # 로그인 후 "Run unattended" 켜기
   ```
   그리고 **NVIDIA 드라이버**(GPU 작업용) — 제조사 사이트 또는 NVIDIA 앱.
8. 시계가 틀어져 있으면(포맷 직후 흔함) — PowerShell(관리자): `w32tm /resync /force`. 안 되면 SETUP.md Step -1.

---

## Step 1: 백업을 손에 쥐고 자동 복구 시작 — 길은 둘

### 길 A (빠름): 외장하드 E: 키트
외장하드를 꽂고 **드라이브 루트의 `복구시작.bat` 을 더블클릭**. 하드 안의 `backups\restore_db.sh` 를 외장하드에서 바로 읽어
돌린다(10GB 를 다시 받지 않는다). **개인 파일 묶음(private)은 이 길에서만 복구된다** — Drive 엔 일부러 안 올린다.

### 길 B: Google Drive (외장하드가 없을 때)
Ubuntu 터미널에서 (이것만 사람이 친다):

```bash
# 1) rclone 설치
curl https://rclone.org/install.sh | sudo bash

# 2) Google Drive 연결 (브라우저 OAuth — 본인 Google 계정 로그인). 기본값으로 충분하다 —
#    받은 설정 tar 가 전용 client_id 가 든 rclone.conf 로 덮어쓴다.
rclone config
#    → n (new) → name: gdrive → storage: drive → 나머지 기본값(Enter) → 브라우저 인증 → y

# 3) 백업 다운로드 (SETUP.md + restore_db.sh + DB 덤프 + 설정 tar)
mkdir -p ~/backups
rclone copy gdrive:backups/db ~/backups --progress

# 4) 전자동 복구 시작 (노트북이면 --auto 대신 --laptop)
bash ~/backups/restore_db.sh --auto
```

`restore_db.sh --auto` 가 하는 일(목록 SSOT = 그 스크립트): 설정 tar 풀기(SSH 키·.env·전역 지침·두 Claude 계정 프로필) →
PostgreSQL 설정 + 백업에 있는 DB 전부 복구 → **프로젝트 전부 git clone**(목록 = 백업 안 `~/.claude/projects.list`) →
프로젝트별 의존성(pip · npm · `mise run setup`) → Claude Code · uv · rclone · mise · Chrome · Playwright · 글꼴 →
터미널 프로필 · 바탕화면 바로가기 → 프로젝트별 systemd 타이머 · cron → 점검.

> 받은 폴더에 `SETUP.md` 가 같이 있다. 자동 복구가 막히면 `~/backups/SETUP.md`(외장하드 길이면 `E:\backups\SETUP.md`)의
> 수동 단계를 따라가면 된다. 그 문서 아래쪽 「복구 공백」 절이 **백업 시점에 다시 못 세울 것으로 알려진 것**을 적어 둔다.

---

## Step 2: 여기서부터 Claude Code에게 맡긴다

자동 복구가 끝나면 Claude Code 가 이미 깔려 있다. 계정은 둘이라 각각 한 번씩 로그인한다:

```bash
source ~/.bashrc
claude                                   # → /login (본 계정)
CLAUDE_CONFIG_DIR=$HOME/.claude-b claude # → /login (예비 계정 — 있을 때만)
```

그 다음 아무 프로젝트 폴더에서 Claude 에게: **"`~/backups/SETUP.md` 를 읽고 복구 점검(`restore_db.sh --check`)과 남은 단계를 마저 진행해줘"**.

> ⚠️ **Claude Code 설치 ≠ Claude가 다 안다.** Claude 는 자동 복구가 `~/.claude/` 와 프로젝트 코드를 되살린 *뒤에야*
> 컨텍스트를 갖는다. 그 전(맨바닥)엔 Claude 도 빈손이라 **Step 0~1 은 반드시 사람이 직접** 한다.

---

## 기억해야 할 단 하나

**외장하드 E:** 또는 **Google 계정 로그인**(이메일 + 비번 + 2FA) — 둘 중 하나만 있으면 → 이 README → 전체 복구.
나머지는 백업이 들고 있다. 외장하드는 매일 새벽 갱신된다(꽂혀 있을 때).

---

*복구 대상: WSL2 Ubuntu-22.04 · PostgreSQL · MyProjects 의 프로젝트 전부(목록은 백업이 말한다) · Claude Code(계정 2) · 프로젝트별 systemd 자동화*
*SETUP 가이드 SSOT: Harvest repo `scripts/automation/setup_template.md` + `generate_setup.py` → 매일 `SETUP.md` 로 Drive·D:·E: 에*
