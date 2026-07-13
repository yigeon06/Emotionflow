# 🧠 Emotionflow: 멀티모달 AI 기반 실시간 감정 인식 시스템
> **생체 신호(EEG, rPPG)와 비디오 데이터(얼굴, 눈, 입)의 게이트 어텐션 융합을 통한 강인한 감정 인식**

본 프로젝트는 인간의 외적 표현(얼굴 표정, 미세 변화, 시선)과 내적 생리 반응(EEG, 비접촉식 rPPG)을 통합하여, 사용자의 감정을 정확하고 강인하게 실시간으로 분류하는 **멀티모달 딥러닝 시스템**입니다. 

---

## ✨ 주요 특징 (Key Features)

* **5중 멀티모달리티(5-Modal Fusion):** 얼굴 전체(`Face`), 눈(`Eye`), 입(`Mouth`), 비접촉식 심박(`rPPG`), 뇌파(`EEG`)의 다층적 데이터 결합
* **DiffuSSM (Diffusion + SSM) 강인화 모듈:** 특징 공간(Feature-level)에서 확산 모델 기반 데노이징과 상태공간모델(SSM)을 접목해, 생체 신호의 낮은 SNR 및 노이즈 문제를 획기적으로 개선 (감정 인식 분야 최초 도입)
* **Softmax 게이트 기반 어텐션 융합:** 단순 연결(Concatenate)이 아닌 동적 가중치 조절을 통해 모달리티 불균형(Modality Imbalance) 해소
* **듀얼 스케일(Dual-Scale) 아키텍처:** 실시간성 단기 예측(1초)과 풍부한 문맥적 표정 변화 예측(15초) 파이프라인 병렬 운용
* **해석 가능한 AI (XAI):** VOH(그룹 퍼뮤테이션 중요도) 및 SHAP 분석을 통한 각 모달리티별 기여도 정량적 검증 완료

---

## 🏗️ 시스템 아키텍처 (System Architecture)

본 시스템은 입력 데이터 전처리, 모달리티별 특징 추출, 게이트 융합, 그리고 최종 감정 분류의 단계를 거칩니다.

1. **데이터 전처리 (Pre-processing)**
   * **라벨 정합:** DEAP(Valence-Arousal 연속값) 데이터를 SEED-IV 체계에 맞춰 4개 감정(`Neutral`, `Sad`, `Fear`, `Happy`)으로 재범주화 및 매핑
   * **rPPG 추출:** MediaPipe FaceMesh를 활용해 이마 및 양측 볼(3 ROI)에서 G 채널 밝기 신호 추출 후 CSP 적용
   * **Emotion CSP (EEG):** 감정 클래스별 공간 필터링을 통해 주파수/공간 패턴 극대화
2. **특징 추출 & 강인화 (Feature Extraction & Denoising)**
   * 각 비디오 피처(Eye 30개, Face 30개, Mouth 20개) 및 APEX 클립(3D CNN) 임베딩 생성
   * CNN 특징 추출기 이후 **DiffuSSM** 모듈을 거쳐 분류 손실($L_{ce}$)을 통해 분류에 유리한 방향으로 노이즈 정제
3. **모달리티 융합 (Gated Fusion)**
   * Softmax 게이트 메커니즘을 적용하여 조건별 신뢰도가 높은 모달리티에 적응적 가중치 할당

---

## 📊 실험 및 성능 평가 (Results)

### 1. EEG 단일 모달리티 성능 비교 (SEED Dataset)
동일 환경 테스트 시, 감정 라벨 기반 CSP 필터링과 DiffuSSM을 결합한 모델이 가장 우수한 성능을 보여주었습니다.
* **CSP + CNN + DiffuSSM 적용 시:** 기존 DE 특징 대비 **약 25% 가량의 성능 향상** 달성

### 2. 4-Modal 비디오 기반 실시간 시연 성능 (1s Model)
* **테스트 정확도:** `72.24%` (Happy 감정 분류 성능이 가장 탁월)
* **모달리티 기여도 분석 (Ablation Study vs Permutation):**

| Modality | 퍼뮤테이션 상대 중요도 (VOH) | 제거 실험 상대 중요도 (Ablation) | 제거 후 최종 정확도 |
| :--- | :---: | :---: | :---: |
| **Face** | **54.4%** | **66.8%** | 46.16% |
| **Mouth** | 31.1% | 25.9% | 62.12% |
| **Eye** | 12.1% | 8.4% | 68.95% |
| **rPPG** | 2.4% | -1.1% | 72.67% |
| **전체 (All)** | 100% | 100% | **72.24%** |

> 💡 **인사이트:** SHAP 분석 결과 순간·국소 단서로는 `Mouth`가 강하게 부각되며, VOH 분석 결과 10초 창으로 축적되는 집단·누적 정보량 관점에서는 `rPPG`가 감정 구분의 거친 바닥면(Baseline)을 형성함을 확인했습니다.

### 3. 실시간 카메라 전이 학습 (Transfer Learning) 결과
DEAP 데이터셋의 사전 학습(Pre-trained) 가중치를 적용하여 실시간 환경에서 캘리브레이션을 진행한 결과, 성능이 대폭 향상되었습니다.
* **기존 방식 (Random Init):** Accuracy `88%`
* **전이 학습 방식 (Transfer Learning):** **Accuracy `91%`**

---

## 🚀 향후 활용 방안 (Applications)

1. **초개인화 비대면 서비스 (CX):** 감성 지능형 키오스크/ATM, 화상 상담 솔루션 내 실시간 고객 불만 포착 및 상담원 코칭
2. **디지털 헬스케어:** 비접촉식 통증 정량화 스케일링 모니터링, 직장인 번아웃 및 만성 스트레스 조기 경보 디지털 바이오마커
3. **지능형 모빌리티 (In-Cabin):** 운전자의 인지적 피로(멍함) 감지, 로드 레이지(분노) 예방 및 이완 유도 시스템
4. **에듀테크 (E-Learning):** 학습자의 인지 과부하 및 '아하! 모먼트' 감지를 통한 맞춤형 콘텐츠 속도 조절

---

## 🛠️ 개발 환경 및 파라미터 (Environment & Parameters)

* **주요 모델 파이프라인:** CNN, Transformer, DiffuSSM, SSM, 3D-CNN Encoder
* **학습 하이퍼파라미터:**
  * Segment Length: `1s` / FPS: `30`
  * Batch Size: `16` / Epochs: `50` / Test Ratio: `0.2`
  * EEG Window Length: `128`
  * Calibration LR: `1e-4` (1s model) / `1e-5` (15s model)

---

## 👥 팀원 및 역할 분담 (Team Emotionflow)

* **조헌 (팀장):** EEG 모달리티 모델 개발, 실시간 모델 개발 총괄
* **김민지:** Micro Face 모델 개발, EEG 모달리티 모델 개발
* **김희주:** rPPG 모델 개발 및 신호 구역 최적화
* **이건:** Mouth 모델 개발, rPPG 모델 개발
* **진세현:** Face 모델 개발 및 피처 추출 파이프라인 구축
