# Chapter 5 – Grid/Block 구성과 대형 데이터 연산 튜토리얼

이 폴더는 책의 5장을 따라가며 그리드/블록 차원을 바꿔가며 벡터·행렬 합을 수행하는 예제들을 담고 있습니다. 각 예제는 GPU 커널이 어떻게 스레드 인덱스를 계산하고, 호스트-디바이스 간 데이터 이동과 동기화를 처리하는지 보여줍니다.

## 예제 개요

| 폴더 | 핵심 포인트 | 파일 |
| --- | --- | --- |
| `5_1_vecAdd_large` | 1차원 그리드/블록에서 대규모 벡터 합. 타이머로 Host/Device 구간 성능 비교. | `5_1.cu`, `DS_timer.*`, `DS_definitions.h` |
| `5_2_MatAdd_2D_index` | 2차원 블록에서 행·열 인덱스를 직접 계산하여 32×32 행렬 합. | `5_2.cu` |
| `5_3_LargeMatrix_Add` | 대형 행렬(8192×8192)을 다양한 스레드 레이아웃(1D/2D 그리드·블록 조합)으로 더하며 성능과 인덱싱 차이 비교. | `main.cpp`, `MatAdd_LargeMatrix.*`, `DS_timer.*`, `DS_definitions.h` |

## 사전 준비

- CUDA Toolkit이 설치되어 있고 `nvcc`가 PATH에 있어야 합니다.
- CUDA가 동작하는 GPU가 있어야 실제 커널 실행을 확인할 수 있습니다. (없을 경우 컴파일은 되지만 실행 시 실패할 수 있습니다.)

## 터미널에서 바로 빌드/실행하기

아래 명령은 `Book_BJ/Chap5` 디렉터리에서 실행합니다.

```bash
# 대용량 벡터 합
mkdir -p bin && nvcc 5_1_vecAdd_large/5_1.cu 5_1_vecAdd_large/DS_timer.cpp -o bin/vec_add_large -std=c++11
./bin/vec_add_large

# 2D 인덱스 기반 행렬 합 (32x32)
mkdir -p bin && nvcc 5_2_MatAdd_2D_index/5_2.cu -o bin/mat_add_2d_index -std=c++11
./bin/mat_add_2d_index

# 대형 행렬 합 (다양한 그리드/블록 레이아웃)
mkdir -p bin && nvcc 5_3_LargeMatrix_Add/main.cpp 5_3_LargeMatrix_Add/MatAdd_LargeMatrix.cu 5_3_LargeMatrix_Add/DS_timer.cpp -o bin/mat_add_large -Xcompiler -std=c++11
./bin/mat_add_large
```

각 실행 결과는 `GPU works well!` 같은 검증 메시지와 타이머 출력을 통해 호스트와 GPU 구간별 시간을 보여줍니다.

## VS Code 환경 설정

1. VS Code에서 `Book_BJ/Chap5` 폴더를 워크스페이스로 엽니다.
2. **터미널** → **구성된 작업 실행**(`Ctrl+Shift+B`)을 선택하고 원하는 빌드 작업을 고릅니다.
   - `build: vec_add_large`
   - `build: mat_add_2d_index`
   - `build: mat_add_large`
3. 기본 디버그 설정은 `bin/mat_add_large` 실행 파일을 대상으로 합니다. 다른 실행 파일을 디버깅하려면 `.vscode/launch.json`의 `program` 경로를 원하는 바이너리로 변경하거나, 새 구성을 추가하면 됩니다.

## 튜토리얼 포인트 체크리스트

- **스레드 인덱싱**: 1D(`blockIdx.x`, `threadIdx.x`)와 2D(`blockIdx.y`, `threadIdx.y`) 조합이 행·열을 어떻게 결정하는지 코드를 따라가세요.
- **그리드 크기 계산**: `ceil((float)N / blockDim.x)`처럼 나머지 데이터를 처리하기 위한 그리드 크기 계산을 확인하세요.
- **메모리 이동 구분**: 타이머 네임을 통해 Host↔Device 복사, 커널 실행, 전체 CUDA 구간 시간이 어떻게 나뉘는지 살펴보세요.
- **대형 데이터 처리 전략**: `5_3_LargeMatrix_Add`에서 1D/2D 블록·그리드 조합이 동일한 계산을 어떻게 다른 방식으로 스케줄링하는지 비교해 보세요.

이 튜토리얼을 따라가며 스레드 레이아웃과 인덱싱, 그리고 데이터 전송 비용이 GPU 커널 성능에 어떤 영향을 주는지 실험해 보세요.
