# 광학계를 이용한 패턴 매칭 프로그램

## 프로젝트 개요

본 프로젝트는 반도체 공정에서 불량품 검사 및 부품 식별 등 품질 관리를 위한 비전 시스템의 중요성이 증가함에 따라, 기존 검사의 자동화 수준을 고도화할 수 있는 고속, 고정밀 패턴 매칭 기술을 구현하고자 합니다. 서버-클라이언트 환경에서 동작하는 실시간 패턴 매칭 시스템을 구축하여 스마트 팩토리 구축의 기반 기술 및 유지보수 효율화를 확보하는 것을 목표로 합니다.

이 시스템은 템플릿 이미지를 기준으로 입력 영상에서 유사한 패턴을 찾아내는 프로그램입니다. 서버와 클라이언트로 구성되며, TCP/IP 소켓 통신을 기반으로 데이터를 교환합니다.

* 개발 인원 : 4명
* 개발 기간 : 25.03.13 ~ 25.06.05 (약 3개월)
* ![Image](https://github.com/user-attachments/assets/24cd20ea-5106-46f9-9226-109428b262cf)

## 🧑‍💻 담당 업무

* **서버-클라이언트 통신 기능 구현**: TCP/IP 소켓 통신을 이용한 서버 및 클라이언트 프로그램 개발
* **데이터베이스 연동**: MySQL 서버를 이용한 매칭 결과(이미지, 로그) 저장 및 관리 기능 구현
* **GUI 개발**: MFC 기반의 사용자 인터페이스 설계 및 구현
* **전체 시스템 통합 및 테스트**: 개발된 모듈들의 통합 및 기능 검증

## 주요 기능 및 특징

* **서버-클라이언트 시스템**: 안정적인 TCP/IP 통신 기반의 서버-클라이언트 시스템 구축.
    * 서버는 클라이언트의 접속을 대기하고, 접속 시 통신을 위한 전용 소켓을 생성합니다.
    * 클라이언트는 서버에 연결 요청을 보내고, 연결이 수락되면 1:1 통신을 위한 소켓이 형성됩니다.
    * 데이터는 `Send` 및 `OnReceive` 함수를 통해 송수신됩니다.
    * 하나의 서버가 여러 클라이언트와 동시에 접속 및 통신이 가능합니다.
* **다양한 패턴 매칭 알고리즘**: OpenCV 라이브러리를 활용한 5가지 이상의 패턴 매칭 알고리즘 구현 및 선택 기능 제공.
    * **Edge_Simple**: Sobel 필터를 이용하여 엣지를 추출하고 `matchTemplate()` 함수로 유사 영역 매칭 (빠르고 직관적이나 회전 및 스케일 변화에 민감).
    * **Contours**: 입력 영상을 이진화하여 윤곽선을 검출하고 유사한 형태를 매칭 (형태 중심 비교로 정확도 높음).
    * **YOLO**: 사전에 학습된 YOLO 객체 탐지 모델을 이용한 실시간 객체 탐지 (빠르고 강건한 탐지 가능).
        * `YOLO/YOLO_Target`은 템플릿 이미지와 관계없이 첨부파일의 원본과 같은 이미지만 인식합니다.
        * YOLO 모델은 하나의 패턴만 탐지하는 방법과 주변 패턴도 탐지하는 방법 두 가지로 나뉘며, `.onnx` 파일을 따로 학습시켰습니다.
    * **ORB**: ORB 알고리즘을 이용한 특징점 추출 및 디스크립터 매칭 (회전 및 스케일 변화에 강인).
    * **회전 검사 기능**: MatchContours 알고리즘의 성능 향상을 위해 -30도에서 30도까지 각도를 반복문을 통해 변화시키며 가장 유사도가 높은 부분을 출력합니다. UI 상에서 회전 검사 기능의 활성화 여부를 설정할 수 있습니다. (Edge_Simple은 회전 검사를 추가해도 차이가 없어 제외되었습니다).
* **자동 밝기 조절 기능**: 카메라 영상의 히스토그램 분석을 통한 자동 밝기 조절 기능 구현.
    * 템플릿 이미지의 히스토그램과 카메라 영상의 히스토그램을 유사하게 하여 밝기를 조절합니다.
    * 수동 밝기 조절 (0~30으로 제한).
* **데이터 관리**: 매칭 결과(원본 이미지, 결과 이미지, 로그)의 실시간 서버 전송 및 데이터베이스(MySQL Server 8.0) 연동.
    * 이미지는 BLOB 형태로 DB에 저장됩니다.
    * DB에 이미지를 저장할 때 MySQL의 패킷 최댓값(기본 4MB)을 초과하는 경우 오류가 발생할 수 있어, DB 재연결 상호작용을 추가하였습니다.
* **GUI**: 사용자 편의성을 고려한 MFC 기반의 GUI 설계.

## 시스템 구성

### 1. 네트워크 구성도

![Image](https://github.com/user-attachments/assets/ac9e9dac-794b-43cd-98bc-0e67752422aa)

*서버 1대와 다수의 클라이언트가 연결될 수 있는 Csocket 기반의 클라이언트-서버 모델로 설계되었습니다.*

### 2. 개념 설계도

* ![Image](https://github.com/user-attachments/assets/98ddc562-9914-417e-b079-a9a2ba60cc56)
* ![Image](https://github.com/user-attachments/assets/70dc8a9a-6e8b-41ae-a58d-33d41385e556)

*서버는 시스템 제어 시작점을 담당하며, 클라이언트는 서버 명령에 따라 영상 처리 과정을 수행합니다.*
* **서버**: 클라이언트에 `Align` 명령을 전송하고, 클라이언트로부터 결과 데이터(매칭 여부, 위치, CD 값 등)를 수신하여 화면에 표시합니다.
* **클라이언트**: 영상 촬영 (카메라 및 라이트 박스 사용), 마크 탐지, `Align` 연산 및 CD 측정, 결과 전송을 수행합니다. 또한 카메라 및 라이트 박스와 직접 연결되어 영상의 밝기를 제어하거나 실시간 영상 데이터를 수집합니다.

## 개발 환경

* **H/W**:
    * 일반 PC (Windows 10 이상, 이더넷 포트 필요)
    * 카메라: Basler acA640-120gm (Interface: GigE)

    ![Image](https://github.com/user-attachments/assets/4faf3c83-ab5b-4d10-b370-f99ddd53f9e3)

    * Light Controller: 2CH, RS-232C
    
    ![Image](https://github.com/user-attachments/assets/ab97c50e-6749-47a4-b19b-66a9c1a4f226)

* **S/W**:
    * 개발 환경: Microsoft Visual Studio 2019 이상
    * 사용 언어: C++
    * 주요 라이브러리: MFC, CSocket, OpenCV 4.11.0, Pylon 5
    * 데이터베이스: MySQL Server 8.0

## 구현 및 테스트

### 1차 모델 (통신 및 DB 저장)

* **기능**: 기본적인 Csocket을 이용한 1:1 통신 기능 및 문자열, 이미지 전송 기능을 구현하였습니다. 클라이언트로부터 전송받은 이미지를 출력 박스에 출력하고, DB에 저장합니다.
* **테스트 항목**:
    * 클라이언트가 서버 IP/Port로 접속 시도 후, 클라이언트로부터 받은 데이터를 서버가 DB에 저장.
* **예상 출력**: 연결 성공 메시지 및 DB 기록 조회 가능.
  
  ![Image](https://github.com/user-attachments/assets/2ee2bfbc-196b-43ac-9999-4d284a0d2a27)
  
  ![Image](https://github.com/user-attachments/assets/ac101001-27f8-40d3-b179-8ab78775e8dd)
  
  ![Image](https://github.com/user-attachments/assets/b1a9d0e5-a30d-4ebf-bba5-64107f177b7e)
  
  ![Image](https://github.com/user-attachments/assets/908e86c6-fc52-4f85-b0d5-41e958c392a9)
  

### 2차 모델 (패턴 매칭 알고리즘)

* **기능**: OpenCV를 연동하여 서버로부터 받은 템플릿 이미지와 임의로 불러온 원본 이미지를 매칭 알고리즘을 이용하여 UI에 출력합니다.
* **테스트 항목**:
    * 특정 문자의 패턴과 원본 이미지 입력.
* **예상 출력**: 매칭이 완료된 이미지가 외곽선 검출 및 가운데 정렬 후 UI에 출력.
* **실행 시간 비교**:
    * Edge Matching execution time: 84.34 ms
    * ![Image](https://github.com/user-attachments/assets/e6b21b82-86d4-43ad-96ec-438e6a080479)
      
    * Contour Matching execution time: 14.68 ms
    * ![Image](https://github.com/user-attachments/assets/81d21267-fdab-49c2-be64-ca6238ef6932)
      
    * YOLO Matching execution time: 1414.26 ms
    * ![Image](https://github.com/user-attachments/assets/dc42e61b-6ba1-4324-b3a8-0728e89acc62)
      
    * ORB Matching execution time: 516.56 ms
    * ![Image](https://github.com/user-attachments/assets/f9734f1e-f304-47eb-aab1-297c26e8b9a6)
      
  

### 최종 모델

* **5가지 패턴 매칭 알고리즘 개선**:
    * `MatchContours` 알고리즘의 성능 향상을 위해 각도(-30도 ~ 30도)를 반복문으로 변화시키며 가장 유사도가 높은 부분을 출력합니다.
    * **회전 검사 기능 활성화 여부**:
        ![Image](https://github.com/user-attachments/assets/fe8c17ca-73fc-47e6-b82b-d44e07558b3d)

    * **회전 검사 기능 비교**:
        | 템플릿 이미지 | 원본 이미지 |
        | :-: | :-: |
        | ![Image](https://github.com/user-attachments/assets/ddb24ae5-0b84-4924-b72b-eef698b8211b) | ![Image](https://github.com/user-attachments/assets/58478d82-39fc-40ec-9fd8-4d83d3266a2a) |
        | **기존 함수 (회전 검사 X)** | **수정 함수 (회전 검사 O)** |
        | ![Image](https://github.com/user-attachments/assets/03c2a6c4-c7fc-4fe6-ba1c-bd7989876133) <br> (이미지 설명: 기존 함수 적용 결과) | ![Image](https://github.com/user-attachments/assets/dd81379e-29dc-4971-86dd-617ce1cfb3fe) <br> (이미지 설명: 수정 함수 적용 결과) |
        | ![Image](https://github.com/user-attachments/assets/8922afe1-7261-41bd-9a81-fa3138983e4b) <br> (이미지 설명: 기존 함수 적용 결과 2) | ![Image](https://github.com/user-attachments/assets/5f6b5ab3-a140-4b32-9c2e-fda44ed8b6a6) <br> (이미지 설명: 수정 함수 적용 결과 2) |

    * `YOLO` 매칭 알고리즘은 하나의 패턴만 탐지하는 방법과 주변 패턴도 탐지하는 방법 2가지로 나누어 학습된 `.onnx` 파일을 사용합니다.
    * **YOLO 알고리즘 종류**:
        | YOLO (주변 패턴 탐지) | YOLO_Target (단일 패턴 탐지) |
        | :-: | :-: |
        | ![Image](https://github.com/user-attachments/assets/f48fce10-f895-4720-939c-db1fff9a2c68) | ![Image](https://github.com/user-attachments/assets/d292f6ee-2314-4a80-96a1-b376dff236dc) |

### 최종 GUI
| 서버 | 클라이언트 |
|:---:|:---:|
| ![Image](https://github.com/user-attachments/assets/520205fb-8527-4d85-b576-a415fa74c019) | ![Image](https://github.com/user-attachments/assets/032c517b-3274-4e8e-b213-4aed0c7d6a0a) |
| ![Image](https://github.com/user-attachments/assets/5d0304a7-92a0-4e16-9b57-bae20bd27db3) | ![Image](https://github.com/user-attachments/assets/ad06f9ae-4e4a-4c5d-9e57-a1b582b066f5) |

### 매칭 알고리즘 성능 비교

| | 템플릿 이미지 | 원본 이미지 |
|---|:---:|:---:|
| **이미지** | ![Image](https://github.com/user-attachments/assets/37745e53-9b94-4815-87db-b00fbd1ac318) | ![Image](https://github.com/user-attachments/assets/e912461f-916b-4ee9-bac5-782e4d1788ea) |

| 각도 | 알고리즘 | 매칭율 / 실행 시간 |
|---|---|---|
| **0˚** | EdgeSimple | 99.54% / 20.33 ms | 
| | match_Contours | 99.96% / 1.76 ms |
| | match_Orb | 74.47% / 438.10 ms |
| | YOLO | 1162.63 ms |
| **30˚** | EdgeSimple | 10.19% |
| | match_Contours | 99.82% |
| | match_Orb | 61.70% |
| | YOLO | 1230.65 ms |
| **120˚** | EdgeSimple | 10.14% |
| | match_Contours | 99.82% |
| | match_Orb | 59.57% |
| | YOLO | 1379.94 ms |
    
* **카메라 연동을 통한 실시간 이미지 처리**:
    * `Pylon SDK` 연동 시 발생하는 흐릿한 현상은 Gain, Exposure Time, Frame Rate 파라미터 조절을 통해 해결하였습니다.
    * **Gain**: 센서가 감지한 빛의 증폭 정도를 결정.
    * **Exposure Time**: 센서가 빛을 감지하는 시간을 조정.
    * **Frame Rate**: 1초에 촬영하는 프레임 수.
* **라이트 박스 연동을 통한 히스토그램 기반 밝기 자동 조절 및 수동 조절**:
    * 밝기는 0~255까지 조절 가능하지만, 패턴 인식을 위해 0~30으로 제한하였습니다.
    * 자동 조절은 UI 메뉴 버튼을 통해 `Template_Image`와 `Basler_Camera`의 히스토그램을 유사하게 하여 밝기를 조절합니다.
    * 수동 조절은 Spin Control을 추가하여 라이트 박스의 밝기를 조절합니다.
        | 자동 조절 | 수동 조절 |
        | :-: | :-: |
        | ![Image](https://github.com/user-attachments/assets/2b552db7-491d-48e4-b8a3-7bccf877b086) | ![Image](https://github.com/user-attachments/assets/0dc2c817-979c-486b-899b-00c906be2be8) |

## 핵심 소스코드

### 서버 - 서버 소켓 (ServerSocket.cpp)

```cpp
void ServerSocket::OnAccept(int nErrorCode)
```
*새로운 클라이언트의 접속 요청을 처리하며, 최대 접속 인원을 초과한 경우 거절합니다.*

### 서버 - 클라이언트 소켓 (ClientSocket.cpp)

```cpp
void ClientSocket::SendImage(Mat& img, int selOption)
```
*Mat 형식의 이미지를 PNG로 인코딩 후, 서버로 전송하며 사용자가 선택한 매칭 알고리즘 정보를 함께 보냅니다.*

```cpp
void ClientSocket::OnReceive(int nErrorCode)
```
*클라이언트에서 전송한 before/after 이미지와 로그 문자열을 수신하고 디코딩하여 UI로 전달합니다.*

### 클라이언트 - 통신 소켓 (ClientSocket.cpp)

```cpp
SocketStatus ClientSocket::CreateSocket(CString ipAddr, int port)
```
*IP 주소와 포트를 기반으로 서버에 접속을 시도하고, 서버 가득참 여부를 확인합니다.*

```cpp
bool ClientSocket::CheckConnectReJection()
```
*서버가 최대 접속 수 초과 시, 특정 에러코드를 통해 접속 거절을 알려줍니다.*

```cpp
void ClientSocket::SendImage(Mat& beforeImg, Mat& afterImg, CString strLog)
```
*before/after 이미지를 PNG 형식으로 압축 후 전송하고, 로그 메시지도 함께 서버에 전달합니다.*

```cpp
void ClientSocket::OnReceive(int nErrorCode)
```
*수신한 이미지 데이터를 Mat 형식으로 디코딩하고, 옵션 문자열과 함께 메시지로 전송합니다.*

### MarkDetector.cpp - 공통 함수 (전처리)

```cpp
Mat CMarkDetector::PreprocessImage(const Mat& src)
```
*이미지 전처리 (그레이 스케일 변환 및 가우시안 블러)를 수행합니다.*

```cpp
Mat CMarkDetector::SobelEdgeBinary(Mat& img)
```
*소벨 연산 기반 바이너리 이미지로 변환합니다.*

### 매칭 알고리즘

#### Edge_Simple (MarkDetector.cpp)

```cpp
Mat CMarkDetector::matchEdgeSimple(Mat& image, Mat& temple)
```
*Sobel 연산 기반 바이너리 이미지로 변환 후 템플릿 매칭하여 가장 유사한 위치를 검출합니다.*

#### Contours (MarkDetector.cpp)

```cpp
Mat CMarkDetector::matchContours(Mat& image, Mat& temple)
```
*이미지와 템플릿의 외곽선을 추출하고 유사도 비교를 통해 가장 유사한 형태를 매칭합니다.*

#### YOLO (MarkDetector.cpp)

```cpp
Mat CMarkDetector::matchByYOLO(Mat& img)
```
*ONNX로 변환된 YOLOv8 모델을 통해 템플릿 객체를 탐지하고, 해당 영역의 윤곽선을 추출 및 시각화합니다.*

#### ORB (MarkDetector.cpp)

```cpp
Mat CMarkDetector::matchORB(Mat& image, Mat& temple)
```
*ORB (Oriented FAST and Rotated BRIEF) 기반 특징점 매칭 알고리즘입니다.*

### 카메라 함수 (CameraManager.cpp)

```cpp
bool CameraManager::Open()
```
*Pylon SDK를 이용하여 카메라를 열고 영상 스트리밍을 시작합니다.*
