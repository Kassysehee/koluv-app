# 윈도우 빌드 지시서 — 한옥마을 앱

> 이 문서는 **윈도우 PC의 클로드 코드가 읽고 그대로 실행**하기 위한 것입니다.
> 사람이 할 일은 ①엔진·VS 설치 클릭과 ②GitHub 로그인뿐입니다.

## 대상 기기 (2026-09-15 실측)

```
HP Laptop 15-fc0xxx · Windows 11 Home 64bit
AMD Ryzen 5 7530U (6코어 12스레드, 2.0GHz)
RAM 16GB · AMD Radeon 내장 · DirectX 12 Ultimate 사용 안 함
```

**중요**: 이 기기는 **빌드는 되지만 결과물을 제대로 실행하지 못합니다.**
Nanite 가 SM6(DX12 Ultimate 급)을 요구하는데 이 GPU 가 지원하지 않습니다.
빌드 성공 여부는 「패키지가 만들어졌는가」로 판정하고, 화면 검증은 다른 기기에서 합니다.
(SM5 폴백을 넣어놨으므로 실행 자체는 될 수 있으나 화질이 낮습니다.)

---

## 0단계 — 환경 점검 (제일 먼저)

```powershell
# 저장공간
Get-PSDrive -PSProvider FileSystem | Select Name,@{n='여유GB';e={[math]::Round($_.Free/1GB,1)}}

# 엔진·VS 설치 여부
Test-Path "C:\Program Files\Epic Games\UE_5.8\Engine\Build\BatchFiles\RunUAT.bat"
Get-ChildItem "C:\Program Files*\Microsoft Visual Studio\2022" -ErrorAction SilentlyContinue | Select Name
```

필요 공간: **엔진 60~90GB + 프로젝트 9GB + 빌드 중간산출물 30GB = 최소 150GB**
외장 SSD 를 써도 됩니다. 단 **C드라이브에도 30GB 는 남겨야** 합니다(윈도우가 임시파일을 씀).

---

## 1단계 — 설치 (사람이 클릭)

### Epic Games Launcher → Unreal Engine 5.8
https://store.epicgames.com/ko/download
런처 → **언리얼 엔진** → **라이브러리** → `+` → **5.8**

설치 옵션에서 **「Windows 타깃 플랫폼」을 반드시 체크**하세요. 빠지면 빌드가 안 됩니다.

### Visual Studio 2022 Community (무료)
https://visualstudio.microsoft.com/ko/vs/community/

**워크로드** 두 개:
- C++를 사용한 데스크톱 개발
- C++를 사용한 게임 개발

**개별 구성 요소**:
- MSVC v143 - VS 2022 C++ x64/x86 빌드 도구
- Windows 11 SDK
- .NET 8 SDK

> 이 프로젝트에는 C++ 모듈(`Source/Koluv3D`, `Source/Koluv3DEditor`)이 있어
> Visual Studio 없이는 빌드가 진행되지 않습니다.

---

## 2단계 — 프로젝트 받기

경로는 **짧게** 잡으세요. 윈도우 260자 경로 제한에 걸립니다.

```powershell
gh auth login        # 아직 안 했으면
git clone https://github.com/Kassysehee/Koluv3D.git C:\UE\Koluv3D
cd C:\UE\Koluv3D
```

### 한옥 애셋 채워넣기 (필수)

`Content/KHS` 2.4GB 는 `.gitignore` 로 제외돼 있어 클론에 없습니다.
없으면 건물이 전부 사라진 채 빌드됩니다.

```powershell
cd C:\UE\Koluv3D
gh release download khs-assets --repo Kassysehee/Koluv3D --dir .
copy /b Content-KHS.zip.part-aa + Content-KHS.zip.part-ab Content-KHS.zip
tar -xf Content-KHS.zip
del Content-KHS.zip*
```

**확인** — 이게 존재해야 합니다:
```powershell
Test-Path "Content\KHS\Gonnyeonghap\SM_Gonnyeonghap.uasset"
```

---

## 3단계 — 빌드

```powershell
cd C:\UE\Koluv3D
& "C:\Program Files\Epic Games\UE_5.8\Engine\Build\BatchFiles\RunUAT.bat" BuildCookRun `
  -project="C:\UE\Koluv3D\Koluv3D.uproject" `
  -noP4 -platform=Win64 -clientconfig=Shipping `
  -cook -allmaps -build -stage -pak -nozenstore -utf8output
```

맥에서 쓴 명령과 `-platform` 만 다릅니다 (Mac → Win64).
**Shipping 인 이유**: Development 는 엔진 assert 로 3분 만에 죽는 크래시가 있었습니다.
Shipping 은 `check()` 가 컴파일아웃되어 그 경로가 사라집니다.

**첫 빌드는 4~8시간** 걸립니다. 셰이더를 전부 새로 컴파일하기 때문입니다.
이 기기는 저전력 CPU 라 더 걸릴 수 있습니다.

### RAM 16GB 대비 — 페이지파일을 먼저 늘리세요
쿠킹이 메모리를 크게 먹어 도중에 죽을 수 있습니다.

```
설정 → 시스템 → 정보 → 고급 시스템 설정 → 성능 설정 → 고급 → 가상 메모리 변경
→ "자동 관리" 해제 → 사용자 지정 크기 → 처음 32768 / 최대 65536 (MB)
```

### 결과물
```
C:\UE\Koluv3D\Saved\StagedBuilds\Windows\
```
이 폴더 통째가 앱입니다. 안의 `Koluv3D.exe` 를 실행합니다.

---

## 4단계 — 막히면

이걸 그대로 찍어서 맥 쪽 세션에 보내주세요.
- 명령 프롬프트 마지막 50줄
- `C:\UE\Koluv3D\Saved\Logs\` 의 최신 `.log`
- `%APPDATA%\Unreal Engine\AutomationTool\Logs\` 의 최신 로그

---

## 참고 — 이 프로젝트에서 이미 해결된 것들

윈도우에서 다시 부딪히지 않도록 적어둡니다.

| 증상 | 원인·해결 |
|---|---|
| `DefaultBuildSettings` 오류 | `BuildSettingsVersion.V7` 이어야 함. V5 는 경고레벨이 엔진과 달라 UBT 가 거부 |
| 앱이 3분 만에 종료 | `FSceneCullingBuilder::ProcessPostSceneUpdate` assert. Shipping 빌드 + `r.SceneCulling.Async.*=0` 으로 해결 |
| 내장그래픽에서 실행 불가 | 윈도우 타겟이 SM6 전용이었음. SM5 를 함께 쿠킹하도록 수정됨 |
