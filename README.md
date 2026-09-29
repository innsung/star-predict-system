# ORION - 밤하늘 사진 기반 별자리 인식 서비스

스마트폰으로 촬영한 밤하늘 사진에서 천체 후보를 찾고, 천문 좌표와 비교해 별자리 연결선을 표시하는 팀 프로젝트입니다. React 화면과 FastAPI 분석 서버를 연결하고, YOLO11n의 천체 검출 결과와 Astrometry.net의 좌표 계산 결과를 함께 활용합니다.

## 프로젝트 정보

- 저장소: `star-predict-system` / 서비스명: ORION
- 기간: 2026.08.24 - 2026.09.18
- 형태: 4인 팀 프로젝트
- 담당: YOLO11n 학습 및 평가, WCS 검증, AWS EC2·Docker 배포, 화면 세부 수정
- 주요 구현 영역: 사진 업로드·결과 화면, YOLO 천체 검출, 천문 좌표 검증, 별자리 연결선 표시, Docker 배포 구성
- 주 사용 언어: Python, JavaScript, SQL, HTML, CSS

## 시연영상

[▶ ORION 주요 기능 시연영상 보기 (2분 46초)](https://youtu.be/SqdMXclDtbU)

밤하늘 사진 업로드부터 분석 진행, 천체·별자리 결과와 연결선 확인까지의 서비스 흐름을 소개합니다.

## 주요 기능

- JPG·PNG 사진 선택, 드래그 앤 드롭과 업로드 미리보기
- FastAPI 분석 요청과 분석 진행·오류·재시도 화면
- YOLO11n 기반 주요 별·천체 후보 검출과 위치·신뢰도 반환
- Plate Solving으로 사진의 하늘 방향·화각·회전 정보 계산
- WCS를 이용한 별 위치 확인과 별자리 연결선 표시
- 기존 사진의 WCS 결과 재사용과 새로운 사진의 좌표 계산
- 좌표 검증 결과와 AI 후보 결과를 구분해 표시
- 회원 인증, 별자리 정보·등록, 운세·결제 관련 API 연동

## 서비스 흐름

```text
밤하늘 사진 업로드
  → React에서 FastAPI 분석 API 호출
  → YOLO11n으로 학습된 천체 후보와 위치 검출
  → 기존 WCS 캐시 확인 / 없으면 Plate Solving 수행
  → WCS와 기준 별 자료로 사진 속 별자리 위치 확인
  → 별자리 연결선·천체 후보·검증 상태를 화면에 표시
```

좌표 검증이 어려운 사진은 별 배치 비교 결과나 AI 후보를 활용하되, WCS로 검증된 결과와 구분합니다. 사진에 별이 적거나 흔들림·노이즈가 심하면 좌표 계산이 실패할 수 있습니다.

## AI 모델과 천문 좌표 처리

### YOLO11n: 사진에서 천체 후보 찾기

YOLO11n을 전이학습해 사진 속 천체의 종류와 위치를 검출합니다. 학습 대상은 `Pleiades`, `Jupiter`, `Betelgeuse`, `Aldebaran`, `Zeta Tauri`, `Elnath`, `Hassaleh`, `Bellatrix`의 8개 천체입니다.

YOLO가 88개 별자리 전체를 직접 분류하는 구조는 아닙니다. 천체 후보 검출은 YOLO가 담당하고, 별자리 위치와 연결선 확인에는 천문 좌표와 기준 자료를 함께 사용합니다. 화면의 신뢰도는 모델의 예측 점수이며 실제 정답률을 뜻하지 않습니다.

### Plate Solving·WCS: 사진과 실제 하늘 연결하기

1. **Plate Solving:** 사진 속 별 배열을 기준 자료와 비교해 촬영 방향, 화각과 사진의 회전 각도를 계산합니다.
2. **WCS 좌표 변환:** 계산 결과를 바탕으로 사진의 픽셀 위치와 하늘의 좌표를 서로 변환합니다.
3. **위치 확인·연결선 표시:** 기준 별 위치를 사진 위로 옮겨 관측된 별과 비교하고, 별자리 연결선을 겹쳐 표시합니다.

Plate Solving과 WCS 처리는 별 배열과 좌표 계산을 사용하는 과정으로, YOLO 딥러닝 추론과는 역할이 다릅니다.

### 데이터·학습 관리

- 외부 사진의 출처와 라이선스를 기록하고 중복·유사 이미지를 점검합니다.
- 같은 촬영 세션의 사진이 학습용과 평가용에 섞이지 않도록 분리합니다.
- YOLO 학습·평가와 천체별 오검출·누락 분석 스크립트를 관리합니다.
- 별 카탈로그와 별자리 연결선 자료를 좌표 검증·시각화에 활용합니다.

데이터 출처, 단계별 스크립트와 날짜별 학습·평가 기록은 [Constellation 상세 README](Constellation/README.md)에 정리되어 있습니다.

## 어려운 용어 간단 설명

| 용어 | 쉬운 설명 |
|---|---|
| Plate Solving | 사진 속 별 배열로 사진이 향한 하늘 방향, 화각, 회전 각도를 계산하는 과정입니다. |
| WCS | 사진 픽셀과 하늘 좌표를 연결하는 변환 정보입니다. 이를 이용해 별 위치를 검증하고 연결선을 표시합니다. |
| 천구 좌표·RA/DEC | 하늘에서 별의 위치를 나타내는 주소입니다. 적경(RA)과 적위(DEC)를 사용합니다. |
| 화각·픽셀 스케일 | 화각은 사진에 담긴 하늘의 넓이, 픽셀 스케일은 한 픽셀이 차지하는 하늘의 각도입니다. |
| Astrometry.net | 별 배열을 대조해 Plate Solving을 수행하고 WCS 정보를 만드는 도구입니다. |
| HYG·Gaia 카탈로그 | 별의 위치와 밝기 등을 모아 둔 기준 목록입니다. 사진 속 별과 대조할 때 사용합니다. |
| Stellarium 연결선 | 어떤 별들을 이어 별자리를 그릴지 정의한 자료입니다. |
| 재투영·오버레이 | 재투영은 하늘 좌표를 사진 위치로 옮기는 계산, 오버레이는 그 위치에 점·선을 겹쳐 보여주는 것입니다. |
| 그래프 매칭·Delaunay 삼각분할 | 별들을 점과 선·삼각형으로 묶고, 거리 비율과 각도를 비교해 비슷한 별 배치를 찾는 방법입니다. |
| YOLO11n·객체 검출 | 사진에서 대상의 종류와 위치를 찾는 딥러닝 모델입니다. `n`은 작은 경량 모델을 뜻합니다. |
| 전이학습 | 이미 학습된 모델을 출발점으로, 프로젝트의 천체 사진을 추가 학습하는 방법입니다. |
| 바운딩 박스·신뢰도 | 검출한 대상을 둘러싼 사각형과 모델의 예측 점수입니다. 신뢰도가 실제 정답률인 것은 아닙니다. |
| 세션 분할·데이터 누수 | 같은 촬영 묶음을 학습·평가에 나누어 넣지 않는 방식입니다. 비슷한 사진이 양쪽에 섞여 평가가 부풀려지는 것을 줄입니다. |
| Precision·Recall·mAP | 각각 검출 결과 중 정답 비율, 실제 대상 중 찾아낸 비율, 종류·위치 검출 성능을 종합한 평가 지표입니다. |

좌표 처리 용어 참고: [Astrometry.net 문서](https://astrometry.net/doc/readme.html), [Astropy WCS 문서](https://docs.astropy.org/en/stable/wcs/wcsapi.html). 더 자세한 설명은 [용어설명](Constellation/용어설명.md)을 참고하세요.

## 기술 스택

| 구분 | 기술 |
|---|---|
| Frontend | JavaScript, React 18, Vite, React Router, styled-components |
| Backend | Python, FastAPI, Uvicorn, Pydantic |
| AI·영상처리 | YOLO11n, Ultralytics, OpenCV, Pillow |
| 천문 좌표·데이터 | Astrometry.net, Astropy, Pandas, HYG·Gaia, Stellarium 자료 |
| Database·인증 | MySQL, SQLAlchemy, PyMySQL, JWT, bcrypt |
| 배포 | Docker Compose, Nginx, AWS EC2 배포 구성 |

## 프로젝트 구조

```text
front/                  사진 업로드·분석·결과 및 서비스 화면
server/routes/          회원·별자리·인식·운세·결제 API
server/services/        YOLO 추론, WCS 연결선과 인식 결과 처리
server/models/          데이터베이스 모델
Constellation/scripts/  데이터 수집·전처리·좌표 계산·학습·평가
Constellation/data/     기준 자료와 로컬 데이터·분석 결과
deploy/                 배포용 모델·Astrometry 인덱스·WCS 캐시
docker-compose.yml      Frontend·Backend·MySQL 실행 구성
```

## 실행 방법

### 로컬 실행 (Windows PowerShell)

Python, Node.js, MySQL을 준비하고 Frontend와 Backend를 **각각 별도 터미널**에서 실행합니다. 아래 명령은 저장소 루트를 시작 위치로 합니다.

#### 1. MySQL·환경변수 준비

MySQL에서 사용할 데이터베이스를 먼저 생성합니다.

```sql
CREATE DATABASE IF NOT EXISTS astra CHARACTER SET utf8mb4;
```

`server/.env`에 아래 항목을 설정합니다. 기존 파일이 있다면 필요한 값만 수정합니다.

```env
DB_HOST=127.0.0.1
DB_PORT=3306
DB_NAME=astra
DB_USER=your_db_user
DB_PASSWORD=your_db_password
ACCESS_SECRET=replace_with_a_unique_access_secret_at_least_32_chars
REFRESH_SECRET=replace_with_a_different_refresh_secret_at_least_32_chars
FRONT_ORIGINS=http://localhost:5173,http://127.0.0.1:5173
YOLO_MODEL_PATH=D:/dev/star-predict-system/deploy/models/best.pt
KNOWN_WCS_DIR=D:/dev/star-predict-system/deploy/wcs-cache
```

모델·캐시 경로는 실제 저장 위치로 변경합니다. 두 인증 비밀값은 서로 다른 32자 이상의 값으로 설정합니다. 서버 시작 시 모델에 정의된 테이블을 생성하지만, 별자리·별 등 기준 데이터는 별도로 준비해야 합니다. 운세·결제 기능을 사용할 때는 `.env.docker.example`의 관련 API 설정도 참고해 `server/.env`에 추가하고, 결제 복귀 주소는 로컬 Frontend 주소에 맞춥니다.

#### 2. Backend 실행

```powershell
cd server
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m pip install -r ../Constellation/requirements.txt
.\.venv\Scripts\python.exe -m uvicorn main:app --reload --port 8000
```

- 서버 확인: `http://localhost:8000`
- API 문서: `http://localhost:8000/docs`
- 가상환경의 Python을 직접 실행하므로 별도 활성화 명령은 필요하지 않습니다.

#### 3. Frontend 실행

새 터미널을 열고 저장소 루트에서 실행합니다.

```powershell
cd front
npm install
npm run dev
```

접속 주소는 `http://localhost:5173`입니다. Vite가 `/api` 요청을 `http://127.0.0.1:8000`의 Backend로 전달합니다.

#### 4. 사진 분석에 필요한 추가 준비

- YOLO 추론에는 학습된 `best.pt` 파일이 필요합니다.
- 새로운 사진의 Plate Solving에는 Windows의 WSL Ubuntu 환경에 Astrometry.net과 해당 화각의 인덱스가 필요합니다. Python 패키지 설치만으로 이 도구가 설치되지는 않습니다.
- 별 카탈로그·연결선과 WCS 캐시 등 분석 자료도 준비해야 합니다. 설치·데이터 준비는 [Constellation 상세 README](Constellation/README.md)를 참고합니다.

### Docker 실행

Docker Compose 구성을 기준으로 합니다. 저장소 코드 외에 학습 모델과 천문 기준 자료를 준비해야 사진 분석을 실행할 수 있습니다.

1. `.env.docker.example`을 `.env.docker`로 복사하고 DB·인증·외부 API 설정을 입력합니다. 기존 설정 파일이 있다면 필요한 값만 수정합니다.
2. 배포용 모델 `deploy/models/best.pt`와 사진 화각에 맞는 Astrometry 인덱스를 `deploy/astrometry/`에 준비합니다.
3. WCS 캐시와 별 카탈로그·연결선 등 분석에 필요한 자료를 준비합니다. 데이터 준비 과정은 [상세 README](Constellation/README.md)를 참고합니다.
4. 저장소 루트에서 실행합니다.

```powershell
docker compose --env-file .env.docker up -d --build
```

기본 접속 주소는 `http://localhost`이며 `.env.docker`의 `HTTP_PORT` 설정에 따라 달라집니다. Frontend 개발 화면과 연동 설명은 [Frontend README](front/README.md)에 있습니다.

원본 사진·대용량 학습 데이터·모델 가중치와 API 키는 별도로 관리합니다. 외부 사진과 천문 자료를 사용할 때는 각 출처의 라이선스를 확인합니다.
