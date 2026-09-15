# 윈도우에서 시작하기 — 한옥마을 빌드

> 클로드 코드에 **이 문서 주소를 주면** 나머지는 알아서 합니다.
> 사람이 할 일은 **설치 클릭 두 번**과 **로그인**뿐입니다.

---

## 0. 클로드 코드에 줄 프롬프트

PowerShell 에서 `claude` 를 실행한 뒤 아래를 **그대로** 붙여넣으십시오.

```
https://raw.githubusercontent.com/Kassysehee/koluv-app/main/WINDOWS_START.md
이 문서를 읽고 그대로 진행해줘. 먼저 0단계 환경 점검부터 하고,
설치가 필요한 게 있으면 뭘 어디서 받아야 하는지 알려줘.
내가 설치를 마치면 말할 테니 그때 다음 단계로 가자.
```

---

## 1. 윈도우에 설치할 것 (사람이 클릭)

### ① 클로드 코드 — 이미 하셨으면 건너뜁니다
PowerShell 에서:
```powershell
irm https://claude.ai/install.ps1 | iex
```
창을 닫았다 다시 열고 `claude --version` 으로 확인.

### ② Git + GitHub CLI
```powershell
winget install Git.Git
winget install GitHub.cli
```
설치 후 창을 다시 열고:
```powershell
gh auth login
```
→ `GitHub.com` → `HTTPS` → `Login with a web browser` 선택, 화면의 코드를 브라우저에 입력.

### ③ Unreal Engine 5.8 — **집 와이파이에서 하십시오 (60~90GB)**
https://store.epicgames.com/ko/download 에서 **Epic Games Launcher** 설치
→ 로그인 → 상단 **언리얼 엔진** → **라이브러리** → `+` → **5.8** 선택

설치 옵션에서 **「Windows 타깃 플랫폼」을 반드시 체크**하십시오. 빠지면 빌드가 안 됩니다.

### ④ Visual Studio 2022 Community (무료, 약 10GB)
https://visualstudio.microsoft.com/ko/vs/community/

**워크로드** 탭에서 두 개 체크:
- C++를 사용한 데스크톱 개발
- C++를 사용한 게임 개발

**개별 구성 요소** 탭에서 추가로:
- MSVC v143 - VS 2022 C++ x64/x86 빌드 도구
- Windows 11 SDK
- .NET 8 SDK

> 이 프로젝트에는 C++ 모듈이 있어 Visual Studio 없이는 빌드가 진행되지 않습니다.

---

## 2. 필요한 공간

| 항목 | 용량 |
|---|---|
| 언리얼 엔진 5.8 | 60~90GB |
| Visual Studio 2022 | ~10GB |
| 프로젝트 소스 | ~600MB |
| 한옥 3D 애셋 | 2.4GB |
| 빌드 중간산출물 | ~30GB |
| **합계** | **최소 150GB** |

외장 SSD 에 둬도 됩니다. 단 **C드라이브에도 30GB 는 남겨야** 합니다(윈도우가 임시파일을 씀).

---

## 3. 프로젝트 받기 — 클로드 코드가 합니다

SSD 없이 인터넷으로 전부 받아집니다. 총 **3GB** 입니다.

```powershell
git clone https://github.com/Kassysehee/Koluv3D.git C:\UE\Koluv3D
cd C:\UE\Koluv3D

# 한옥 3D 애셋 (.gitignore 로 제외돼 있어 따로 받습니다)
gh release download khs-assets --repo Kassysehee/Koluv3D --dir .
copy /b Content-KHS.zip.part-aa + Content-KHS.zip.part-ab Content-KHS.zip
tar -xf Content-KHS.zip
del Content-KHS.zip*
```

**확인** — 이게 `True` 여야 합니다:
```powershell
Test-Path "Content\KHS\Gonnyeonghap\SM_Gonnyeonghap.uasset"
```

> 경로는 **짧게** 두십시오(`C:\UE\Koluv3D`). 길면 윈도우 260자 제한에 걸려 빌드가 실패합니다.

---

## 4. 빌드

```powershell
cd C:\UE\Koluv3D
& "C:\Program Files\Epic Games\UE_5.8\Engine\Build\BatchFiles\RunUAT.bat" BuildCookRun `
  -project="C:\UE\Koluv3D\Koluv3D.uproject" `
  -noP4 -platform=Win64 -clientconfig=Shipping `
  -cook -allmaps -build -stage -pak -nozenstore -utf8output
```

**첫 빌드는 4~8시간.** 걸어두고 주무시면 됩니다.

### RAM 16GB 라면 페이지파일을 먼저 늘리십시오
```
설정 → 시스템 → 정보 → 고급 시스템 설정 → 성능 설정 → 고급
→ 가상 메모리 변경 → "자동 관리" 해제 → 사용자 지정: 처음 32768 / 최대 65536 (MB)
```

### 결과물
```
C:\UE\Koluv3D\Saved\StagedBuilds\Windows\
```
이 폴더 통째가 앱입니다.

---

## 5. 이 기기의 한계 (미리 알아두실 것)

실측 사양: HP Laptop 15-fc0xxx · Ryzen 5 7530U · RAM 16GB · AMD 내장 GPU ·
**DirectX 12 Ultimate 사용 안 함**

**빌드는 됩니다. 그러나 결과물을 제대로 볼 수는 없습니다.**
한옥마을은 Nanite 로 돌아가는데 Nanite 는 SM6(DX12 Ultimate 급)을 요구하고,
이 GPU 는 그걸 지원하지 않습니다. 셰이더 컴파일은 CPU 가 하므로 빌드 자체는 됩니다.

SM5 폴백을 넣어놨으므로 실행은 되지만 화질이 낮게 나옵니다.
**성공 판정은 「패키지가 만들어졌는가」로 하고, 화면 검증은 다른 PC 에서 하십시오.**

---

## 6. 이미 해결된 함정 (다시 안 밟도록)

| 증상 | 원인·해결 |
|---|---|
| `DefaultBuildSettings` 오류 | `BuildSettingsVersion.V7` 이어야 함. V5 는 경고레벨이 엔진과 달라 UBT 가 거부 |
| 앱이 3분 만에 종료 | `FSceneCullingBuilder::ProcessPostSceneUpdate` assert. Shipping 빌드 + `r.SceneCulling.Async.*=0` 으로 해결 (설정에 이미 들어감) |
| 내장그래픽에서 실행 불가 | 윈도우 타겟이 SM6 전용이었음 → SM5 함께 쿠킹하도록 수정됨 |
| 카메라·마이크 안 됨 | 맥은 네이티브 WKWebView 로 해결. **윈도우는 별도 구현 필요** — 빌드 성공 후 과제 |

---

## 7. 막히면

이걸 그대로 찍어서 맥 쪽 세션에 보내주십시오.
- 명령 프롬프트 마지막 50줄
- `C:\UE\Koluv3D\Saved\Logs\` 의 최신 `.log`
- `%APPDATA%\Unreal Engine\AutomationTool\Logs\` 의 최신 로그
