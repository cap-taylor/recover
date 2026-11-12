# 🖥 새 컴퓨터 세우기 — Claude Code 순서표 (메타프롬프트)

> **누가 읽나**: 새 컴퓨터(또는 포맷한 컴퓨터)에 막 깔린 **Claude Code**. 사람은 아래 「0. 사람 몫」만 하고 이렇게 말한다:
> **「https://github.com/cap-taylor/recover/blob/master/NEW_PC.md 를 읽고 그대로 진행해」**
>
> **목표 상태 = 옛 기기의 기기 명세서**(매일 백업에 담긴다 — 부품·BIOS 판·드라이버 판·모니터 배치·Windows 판·앱 목록·시작 프로그램·winget 목록).
> 손으로 목록을 만들지 않는다. 옛 명세서와 새 기기 실측의 **차이**가 할 일이다.
>
> ⚠️ 공개 문서다 — 비밀번호·키·계정·기기 고유값은 여기 없다. 전부 백업 안에 있다.
> WSL·데이터 전체 복구는 이 순서표의 11단계에서 [README.md](README.md) 로 넘어간다.

---

## 0. 사람 몫 (Claude 가 아직 없을 때)

1. **부품 자료를 폰으로 남긴다** — 구입 견적서, 본체 사진(옆판을 열고 메인보드·그래픽카드·**파워 라벨**·쿨러가 보이게).
   파워(전원 공급기)는 소프트웨어로 못 읽는 유일한 부품이라 이 사진·견적서가 출처다.
2. **BIOS** (부팅 때 Del/F2): 가상화 켜기(Intel `VT-x` / AMD `SVM`) · 램 XMP/EXPO(선택) ·
   그래픽카드 슬롯 PCIe 세대(옛 기기에서 Gen4 로 강제 종료가 반복돼 Gen3 로 묶은 전례가 있다 — 옛 명세서·부품이 같으면 같은 값부터).
   메뉴 이름은 보드마다 다르다 — 옛 데스크탑(ASUS 보드) 실제 자리:
   `Advanced → System Agent (SA) Configuration → PCI Express Configuration → PCIEX16(G4) Link Speed` (사진으로 본 값 = Auto · Gen3 · Gen4).
   바꾼 화면은 사진으로 남긴다.
3. **Windows 11 설치** — 로컬 사용자 이름 **`me`**. 인터넷 연결.
4. **Claude Code 설치** — PowerShell(관리자 아님):
   ```powershell
   winget install -e --id Git.Git
   irm https://claude.ai/install.ps1 | iex
   claude
   ```
   `/login` 으로 로그인한 뒤 위 한 줄을 말한다.

---

## Claude 가 지키는 규칙 (시작 전에 읽는다)

- **진행 파일** `C:\Users\me\newpc\progress.md` — 단계마다 「끝 · 시각 · 실측 한 줄」. 시작할 때 이 파일부터 읽고 끝난 단계는 건너뛴다.
- **재부팅**: 하기 전에 진행 파일에 다음 단계를 적고, 사람에게 한 줄 — *「재부팅 → `claude --continue` → 이어서」*.
- **사람에게 묻는 것은 소프트웨어로 못 읽는 것과 결정뿐**: 파워·케이스·쿨러(견적서·사진 판독) · BIOS 설정값(사진) ·
  로그인·비밀번호 · 되돌릴 수 없는 결정(BIOS 올리기·디스크 지우기) · 지울지 말지.
  나머지(CPU·램·그래픽·디스크·모니터·드라이버 판·Windows 판)는 **재서** 안다. 메인보드는 재되, 맞는지 꼭 묻는다.
- **관리자 일은 모아서** — 관리자 확인 창은 단계마다 한 번. 실행 기록은 파일로 남기고 다시 읽어 확인한다.
- **질문은 선택지로** — 추천을 맨 앞에, 고르면 무엇이 바뀌는지 한 줄씩.
- **매 단계 직전 스스로 묻는 다섯 가지** (메타프롬프트 — 답을 진행 파일에 적고 시작):
  1. 이 단계의 목표 상태는 옛 명세서의 어느 파일·줄인가?
  2. 지금 새 기기에서 잰 값은? — 차이 표를 만든다(같음 · 다름 · 새 기기에만 · 옛 기기에만).
  3. 사람만 할 수 있는 일이 남았나? — 있으면 묶어서 한 번에 부탁한다.
  4. 되돌릴 수 없는 일인가? — 그렇다면 하기 전에 사람 결정.
  5. 재부팅이 필요한가? — 진행 파일을 먼저 쓴다.

---

## 1. 옛 기기 명세서 손에 쥐기

- 외장하드(E: 등)가 있으면 `<드라이브>:\backups\configs_*.tar.gz` 중 가장 최신. 없으면 사람에게 부탁:
  Google Drive 웹 → `backups/db/` → 최신 `configs_*.tar.gz` 를 `Downloads` 로.
- Windows 기본 `tar` 로 필요한 것만 푼다:
  ```powershell
  mkdir C:\Users\me\newpc\old
  tar -xzf <configs 파일> -C C:\Users\me\newpc\old .config/windows-machine .config/laptop-windows/machine scripts/export_windows_machine.ps1 scripts/windows_cleanup.ps1 scripts/register_windows_tasks.ps1 .config/windows-tasks .config/wsl
  ```
  `.config/windows-machine` = 데스크탑 명세서 · `.config/laptop-windows/machine` = 노트북 명세서. 이 기기가 어느 쪽을 대신하는지 사람에게 묻는다(보기 둘).

## 2. 새 기기 재기 + 부품 확인

- `powershell -ExecutionPolicy Bypass -File old\scripts\export_windows_machine.ps1 -Dest C:\Users\me\newpc\now`
- 🔴 **메인보드와 파워는 꼭 사람에게 묻는다**(사장님 지시) — 잰 메인보드 모델을 보여주고 「맞나요?」, 파워는 「제품명·용량을 견적서나 라벨 사진으로 주세요」.
  견적서·본체 사진을 통째로 받으면 그것부터 판독한다.
- 사람에게 견적서·사진을 받아 판독 → `C:\Users\me\newpc\parts.md` (메인보드 · CPU · 램 · 그래픽 · 저장장치 · **파워 제품명·용량·등급** · 케이스 · 쿨러 · 모니터).
  잰 값과 견적서가 다르면 묻는다. 파워 용량이 그래픽카드 권장 용량을 넘는지 확인한다.
  (이 파일은 매일 기기 명세서에 같이 실려 백업된다.)
- 옛 기기 ↔ 새 기기 차이 표를 사람에게 보여준다(부품 · 모니터 수 · Windows 판).

## 3. Windows 업데이트 1차

- `start ms-settings:windowsupdate` 로 화면을 열고 「업데이트 확인」 — 또는 관리자 PowerShell 에서 `PSWindowsUpdate` 모듈로 설치까지.
- 재부팅이 필요하면 진행 파일 → 재부팅.

## 4. 드라이버 — 칩셋 → 그래픽 → 소리 → 랜·무선 → 그 밖

- 메인보드 모델은 2단계에서 잰 값. **보드 제조사 지원 페이지**에서 최신 칩셋·소리·랜 드라이버. 그래픽은 제조사 앱(NVIDIA 앱 / AMD Software).
- 기준: 옛 `drivers.csv` 의 판보다 낮지 않다 · 장치 오류 0 (`Get-PnpDevice -Status Error`).
- 옛 기기에만 있던 장치(옛 프린터·안 쓰는 동글)의 드라이버는 깔지 않는다 — 차이 표에서 사람에게 확인.

## 5. BIOS 판 확인

- 잰 판 ↔ 제조사 최신(지원 페이지의 BIOS 목록 · 판 · 날짜 · 바뀐 내용). 보안 수리·안정성 판이 있으면 표로 보여준다.
  바뀐 내용에 순서 조건(「먼저 X 판」·「ME 펌웨어 먼저」)이 있으면 그 순서를 표에 같이 적는다.
- **올리기는 사람 결정**(새 판은 옛 판으로 못 내려가는 경우가 많다). 올리면 BIOS 설정이 초기화되니 0단계 2번(가상화·PCIe 세대)을 다시.
- 🔴 **「위험하다」만 말하지 말고 안심 순서를 먼저 준비한다** (사장님 지시 — 옛 데스크탑에서 쓴 방식):
  1. 보드의 **되살리기 기능**을 제조사 사양표에서 확인한다 — ASUS `CrashFree BIOS 3`·`BIOS FlashBack` / MSI `Flash BIOS Button` /
     기가바이트 `Q-Flash Plus`·`DualBIOS` / ASRock `BIOS Flashback`. 있으면 「도중에 끊겨도 USB 로 되살린다」가 근거고, 없으면 없다고 말한다.
  2. 제조사 공식 파일만 받고 **SHA-256 을 제조사 표기와 대조**한다. FAT32 USB 에 올릴 파일과, 되살리기 기능이 찾는 이름의 사본을 같이 둔다
     (이름은 보드마다 다르다 — 설명서와 파일 안 문자열로 확인. 내려받은 이름 바꾸기 도구 같은 실행 파일은 돌리지 않는다).
  3. 사람 몫을 한 장으로: 지금 설정 화면 사진 · 올리기 도구에서 파일을 고르고 마지막 확인에 「아니오」로 끝내는 연습 한 번 ·
     정전 대비(UPS 가 없으면 날씨·전기 공사 없는 때) · 경우별 처방(화면 안 뜸 · 같은 판에서 멈춤 · 설정 초기화 · 되살리기 USB). 폰에서 볼 수 있게 보낸다.
  4. 올린 뒤 Claude 가 잰다: BIOS 판 · 가상화(WSL 이 뜨나) · 그래픽 슬롯 세대 · 장치 오류 0.
     램 속도는 BIOS 첫 화면 숫자(옛 데스크탑은 2133 으로 보였다) 말고 Windows `Win32_PhysicalMemory.ConfiguredClockSpeed` 로 잰다(같은 날 3200).
- 실제로 겪은 화면 (옛 데스크탑 ASUS · 0601 → 4004 · 2026-11-13) — 미리 알려 두면 사람이 놀라지 않는다:
  - 올리기 도구가 끝 무렵 `System will restart to update ME Firmware, please choose the same BIOS file to do BIOS update again`
    → 재부팅 뒤 **같은 파일을 한 번 더** 고른다. ME(관리 엔진) 펌웨어를 먼저 올리는 2단계라 정상이다(따로 ME 도구를 받을 일 없음).
  - BitLocker 안내 화면 — 쓰고 있으면 올리기 전에 일시 중지, 안 쓰면(`manage-bde -status` 보호 꺼짐) 그냥 넘긴다.
  - 올린 뒤 첫 부팅 `Before making any adjustments in the BIOS, please press F5 to load the factory default settings … press F1 to enter BIOS setup`
    → F1 로 들어가 F5(기본값) → 0단계 2번(가상화 · PCIe 세대 · XMP)을 다시 맞춘다. 기본값으로 돌아가며 꺼 둔 것이 되살아난다.
  - 그 되살아나는 것 하나 = `Tool → ASUS Armoury Crate → Download & Install ARMOURY CRATE app` (Windows 에 제조사 프로그램을 자동 설치).
    옛 데스크탑은 사람이 「안 쓴다」로 정해 끄고 지웠다 — 새 기기에서도 사람에게 묻는다(조명·팬 조절이 필요하면 켠다).

## 6. Windows 업데이트 2차

- 드라이버 뒤에 생기는 선택 업데이트까지. 「대기 중인 업데이트 0」이 될 때까지. 재부팅은 진행 파일 먼저.

## 7. 정리 — 옛 기기와 같게

- 새 기기의 Store 앱 중 옛 `appx.csv` 에 없고 프레임워크·제거 불가가 아닌 것 = 지울 후보 → 표로 확인받고
  `old\scripts\windows_cleanup.ps1 -RemoveAppx ... -Log ...` (관리자 창 1번).
- 시작 프로그램: 옛 `startup.csv` 와 대조해 옛 기기에서 끄던 것은 끈다(`-DisableStartup` — 작업 관리자와 같은 방식, 되돌릴 수 있다).

## 8. 프로그램 설치

- `winget import -i old\.config\windows-machine\winget.json --accept-package-agreements --accept-source-agreements`
- winget 밖 프로그램 = 옛 `programs.csv` 와의 차이 → 표로 보여주고 사람이 고른 것만.
  은행·관공서 보안 모듈은 깔지 않는다(사이트가 필요할 때 스스로 깐다). 유료 프로그램은 사람이 로그인.

## 9. 모니터

- 옛 `displays.csv`(위치·해상도·주 모니터)와 `monitors.csv`(모델·연결 단자)를 배치 그림으로 보여준다.
- `start ms-settings:display` → 사람이 화면을 같은 배치로 끌어 맞춘다 → Claude 가 다시 재서 옛 배치와 대조. 주 모니터 · 해상도 · 주사율까지.

## 10. 설정 맞추기

- 옛 `machine.txt` 대조: 전원 계획 · 빠른 시작 · 시간대·지역(ko-KR) · 시계 동기(`w32tm /resync /force`, 안 되면 SETUP.md Step -1).

## 11. WSL 과 전체 복구

- [README.md](README.md) Step 0 의 4~6번(WSL 설치 · 재부팅 · Ubuntu 사용자 `dino` · 터미널 한 번 열기) → Step 1 의 길 A(외장하드) 또는 길 B(Drive).
- 그다음은 `restore_db.sh` 가 한다. Windows 예약 작업은 복구 화면이 보여 주는 **관리자 PowerShell 한 줄**.

## 12. 끝 점검

- 다시 재기(`export_windows_machine.ps1 -Dest ...\after`) → 옛 명세서와 차이 표: 부품이 바뀐 줄은 정상 · 나머지 차이 0 이어야 한다.
- 장치 오류 0 · 대기 중인 업데이트 0 · 진행 파일 맨 끝에 「끝」.
- 사람에게 남은 것 한 장: 다시 로그인할 곳(`SETUP.md` 맨 아래 스냅샷 「다시 로그인할 곳」) · 올리지 않기로 한 BIOS 판.
