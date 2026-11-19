<img width="952" height="1348" alt="A4 - 1" src="https://github.com/user-attachments/assets/71bfc5dd-d838-4196-aa97-30a0c98f9038" />

# 다선일미 (茶禪一味)

> **고민은 풍등에 띄우고, 마음은 찻잔에 비운다.**

다선일미는 **휴대폰을 찻잔처럼 기울이며** 차를 따르고 마시는 경험을 통해, 마음속 번뇌를 잠시 내려놓도록 돕는 명상 앱입니다.
화면을 누르는 대신 **기울기 센서**로 차를 따르고 마시며, 고민의 종류에 맞는 차·수인(手印)·선어를 기기 안에서 골라 줍니다.

<br>

## 주요 흐름

| 단계 | 화면 | 사용자 경험 |
|---|---|---|
| 1 | **홈 · 풍등** | 오늘 다른 사람들이 날린 풍등 수와 실시간 피드 |
| 2 | **고민 적기** | 고민을 적으면 기기 안에서 8가지 번뇌 종류 중 하나로 자동 분류 → 풍등에 실어 날려 보냄 |
| 3 | **차 고르기** | 번뇌 종류에 어울리는 차 추천 (국화차, 오미자차, 연꽃차 등 19종) |
| 4 | **차 우리기** | 차가 우러나는 동안 고민에 맞는 수인을 따라 들숨·날숨·멈춤 3회 호흡. 휴대폰이 흔들리면 그 호흡이 조금 길어짐 |
| 5 | **차 따르기** | 휴대폰을 기울여 찻잔에 차를 따름 (기울기 센서) |
| 6 | **차 마시기** | 찻잔을 입가로 기울이듯 휴대폰을 기울여 세 모금 마심 (기울기 센서) |
| 7 | **선어 확인** | 다 마신 찻잔 바닥에서 오늘의 선어와 해설을 만남. 공유 카드 제공 |
| + | **찻잔 도감** | 지금까지 마신 차, 적은 고민, 받은 선어를 선반 위 찻잔으로 모아 보기 |

<br>

## 핵심 기능

### 🫖 기울기로 따르고 마시는 차
- `expo-sensors` DeviceMotion을 16ms 간격으로 구독하고 **로우패스 필터**로 손떨림을 걸러낸 자세 값을 사용합니다.
- 센서 값은 React state가 아닌 **Reanimated `SharedValue`** 로 UI 스레드에 바로 전달해 프레임마다 리렌더링하지 않습니다.
- 차 표면은 기기 기울기만큼 반대로 회전시켜 **항상 수평을 유지**하도록 Skia로 그립니다.
- 센서 구독은 `shared/sensors` 한 곳에서만 하고, 각 화면은 훅으로 값만 받아 씁니다.

### 🏮 번뇌 종류 자동 분류 (온디바이스)
고민 텍스트는 **기기 밖으로 나가지 않습니다.** 외부 LLM·API 없이 기기 안에서만 분류합니다.

1. **키워드 매칭** — 번뇌 종류마다 키워드 목록을 두고, 띄어쓰기를 무시한 채 일치 개수로 점수를 매깁니다.
2. **나이브 베이즈 fallback** — 키워드가 하나도 걸리지 않으면, 직접 작성한 예문으로 미리 학습한 **글자 n-gram 나이브 베이즈 점수표**로 고릅니다.
3. **확신이 없으면 `unsure`** — 1·2등 점수 차이가 기준보다 작으면 억지로 고르지 않습니다.

학습은 개발 PC의 스크립트(`quote/scripts/trainWorryModel.mjs`)로만 하고, 앱에는 결과 점수표(JSON)만 들어갑니다. 사용자는 분류 결과를 직접 바꿀 수 있습니다.

| 묶음 | 번뇌 종류 |
|---|---|
| 탐(貪) | 조급함·욕심, 흔들림·유혹 |
| 진(瞋) | 관계·분노, 자책·무기력 |
| 치(癡) | 불안·걱정, 생각 과다, 진로·방향 |
| — | 잘 모르겠어요 |

### 🙏 번뇌에 맞는 수인(手印)과 호흡
차가 우러나는 동안, 고민의 번뇌 종류에 맞춰 고른 **수인**(부처의 손 모양)을 따라 하며 들숨·날숨·멈춤을 세 번 반복합니다.

| 번뇌 종류 | 수인 | 멘트 |
|---|---|---|
| 조급함·욕심 | 여원인 | 그대의 바람이 이루어지기를 |
| 흔들림·유혹 | 항마촉지인 | 흔들림 속에서도 중심을 지키라 |
| 관계·분노 | 합장 | 그대 자신에게도 자비를 베풀라 |
| 자책·무기력 | 지권인 | 번뇌 속의 그대도 온전함을 잊지 말라 |
| 불안·걱정 | 시무외인 | 두려워하지 말라 |
| 생각 과다 | 선정인 | 고요히 마음에 머물라 |
| 진로·방향 | 전법륜인 | 깨달음의 빛을 나누라 |
| 잘 모르겠어요 | 찻잔 감싸기 | 잠시 걱정을 내려놓고 온기에 머물라 |

- 휴대폰의 **흔들림(충격량)을 센서로 감지**해, 수인을 유지하지 못하고 흔들리면 그 순간의 호흡 단계만 조금 길어집니다 (단계당 최대 +4초, 연속 감지 방지 쿨다운).
- 같은 단계에서 여러 번 흔들려도 시간이 끝없이 늘어나지 않도록 상한을 두어, 흐름이 멈추지 않게 했습니다.

### 📜 선어 추천 알고리즘
- 경전·고전에서 고른 **선어 50개**에 번뇌 종류(`themes`)·어조(`tone`) 태그를 달고, 사용자의 번뇌 종류로 후보를 고릅니다.
- 최근에 받은 선어는 제외하고, **seed 기반으로 결정적으로** 하나를 고릅니다 (같은 입력 → 같은 결과, 시연 재현성 확보).
- 위기 표현이 감지된 입력에는 무거운 어조의 선어를 고르지 않습니다.
- 후보가 비면 기본 선어로 fallback 하므로 **결과가 비는 경우가 없습니다.**
- 추천·분류 로직은 모두 **순수 함수**이며 jest 단위 테스트로 검증합니다.

### 💛 위기 안내
- 고민 텍스트에서 위기 표현이 감지되면 흐름을 끊지 않고, 부드러운 문구로 **자살예방상담전화 109** 연결을 안내합니다.
- 감지는 놓칠 수 있으므로, 감지와 별개로 언제든 볼 수 있는 고정 안내도 둡니다.

### 🌐 실시간 풍등 피드
- Supabase Realtime으로 `lanterns` 테이블의 INSERT를 구독해, 다른 사람이 풍등을 날리면 오늘의 풍등 수가 실시간으로 올라갑니다.
- 공유되는 정보는 **"언제, 어떤 번뇌 종류의 풍등을, 어떤 차와 함께 날렸는지"** 뿐입니다.

<br>

## 개인정보 설계

| 데이터 | 저장 위치 |
|---|---|
| 고민 내용, 받은 선어 | 기기 로컬 (`expo-sqlite`) 에만 저장 |
| 고른 차 | 로컬 기록 + Supabase `lanterns` |
| 공유 정보 | `lanterns(created_at, worry_kind, tea)` 만 — RLS로 INSERT·SELECT만 허용 |

고민 원문은 분류·위기 감지를 포함한 어떤 단계에서도 서버로 전송되지 않습니다.

<br>

## 기술 스택

**App**

<p>
  <img src="https://img.shields.io/badge/iOS-000000?style=for-the-badge&logo=apple&logoColor=white" alt="iOS" />
  <img src="https://img.shields.io/badge/Expo_SDK_58-000020?style=for-the-badge&logo=expo&logoColor=white" alt="Expo" />
  <img src="https://img.shields.io/badge/React_Native-20232A?style=for-the-badge&logo=react&logoColor=61DAFB" alt="React Native" />
  <img src="https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white" alt="TypeScript" />
  <img src="https://img.shields.io/badge/Expo_Router-000020?style=for-the-badge&logo=expo&logoColor=white" alt="Expo Router" />
</p>

**Sensor · Graphics · Animation**

<p>
  <img src="https://img.shields.io/badge/Expo_Sensors-000020?style=for-the-badge&logo=expo&logoColor=white" alt="Expo Sensors" />
  <img src="https://img.shields.io/badge/Skia-4B32C3?style=for-the-badge&logo=skia&logoColor=white" alt="React Native Skia" />
  <img src="https://img.shields.io/badge/Reanimated-6E44FF?style=for-the-badge&logo=react&logoColor=white" alt="Reanimated" />
</p>

**Data**

<p>
  <img src="https://img.shields.io/badge/Supabase-3FCF8E?style=for-the-badge&logo=supabase&logoColor=white" alt="Supabase" />
  <img src="https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white" alt="PostgreSQL" />
  <img src="https://img.shields.io/badge/SQLite-003B57?style=for-the-badge&logo=sqlite&logoColor=white" alt="SQLite" />
</p>

**Test · Tool**

<p>
  <img src="https://img.shields.io/badge/Jest-C21325?style=for-the-badge&logo=jest&logoColor=white" alt="Jest" />
  <img src="https://img.shields.io/badge/ESLint-4B32C3?style=for-the-badge&logo=eslint&logoColor=white" alt="ESLint" />
  <img src="https://img.shields.io/badge/Figma-F24E1E?style=for-the-badge&logo=figma&logoColor=white" alt="Figma" />
</p>

별도 백엔드 서버 없이 Supabase와 기기 로컬 DB만으로 구성했습니다.

<br>

## 아키텍처

**화면 하나 = 도메인 하나** 를 기준으로 나누고, 도메인끼리는 각 도메인의 `index.ts`(공개 API)로만 의존합니다. ESLint `no-restricted-imports` 규칙으로 다른 도메인 내부 파일 import를 막습니다.

```
src/
  app/              # expo-router 라우팅 (도메인 컴포넌트 조합만)
  features/
    home/           # 홈 · 오늘의 풍등 수 · 실시간 피드
    worry/          # 고민 적기 · 풍등 날리기 · 위기 안내 모달
    teaFlow/        # 차 고르기 · 우리기 · 따르기 · 마시기
    zen/            # 찻잔 바닥 선어 확인 · 공유 카드
    collection/     # 찻잔 도감 (내 기록)
    session/        # [화면 없음] 회차 데이터 보관 · 로컬 저장/조회
    safety/         # [화면 없음] 위기 표현 감지
    quote/          # [화면 없음] 선어 데이터 · 추천 · 번뇌 자동 분류
  shared/
    sensors/        # 기울기·흔들림 센서 훅 (유일한 센서 구독 지점)
    theme/          # 디자인 토큰
    lib/            # Supabase · 로컬 DB 클라이언트
    constants/      # 번뇌 종류 id · 묶음 · 표시 이름
supabase/
  migrations/       # 스키마 SQL
```

의존 방향은 `app → features → shared` 한 방향이며, 도메인 간 함수가 필요하면 GitHub issue로 시그니처를 합의한 뒤 담당자가 먼저 stub을 공개하는 방식으로 병렬 개발했습니다.

<br>

## 실행 방법

```bash
npm install
cp .env.example .env   # EXPO_PUBLIC_SUPABASE_URL, EXPO_PUBLIC_SUPABASE_ANON_KEY 입력
npx expo start         # 아이폰 Expo Go로 QR 스캔
```

> iOS 시뮬레이터에는 기울기 센서가 없어 차 따르기·마시기는 **실제 아이폰**에서 확인해야 합니다.

```bash
npx tsc --noEmit   # 타입 체크
npx expo lint      # lint
npx jest           # 단위 테스트 (분류 · 추천 · 위기 감지 · 센서 필터)
```

<br>

## 팀

| 이름 | 파트 | 담당 |
|---|---|---|
| 신연아 [@snyeona](https://github.com/snyeona) | FE | 차 흐름(고르기 · 따르기 · 마시기), 센서 |
| 노희윤 [@hyroh5](https://github.com/hyroh5) | FE | 홈 · 풍등, 고민 적기, 차 우리기, 선어 확인, 찻잔 도감 |
| 박세인 [@parksein0223](https://github.com/parksein0223) | BE · Data | Supabase(스키마 · RLS · Realtime), 로컬 DB, 회차 데이터, 위기 감지 |
| 유민아 [@yumina0616](https://github.com/yumina0616) | Algorithm | 선어 데이터, 추천 알고리즘, 번뇌 자동 분류 |
