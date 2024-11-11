# 제주도 관광지 추천 시스템

이 프로젝트는 **제주도 관광지 추천 시스템**으로, 하이브리드 필터링(회귀 기반 필터링과 콘텐츠 기반 필터링)을 통해 사용자 입력에 맞춘 관광지 추천을 제공합니다. 이 README 파일에서는 시스템의 구현 과정, 기술 스택, 핵심 로직, 데이터셋 등을 설명합니다.

## 목차
1. 프로젝트 개요
2. 설계 및 구현
3. 추천 시스템 구성
4. 접근성 필터링 구현
5. 기술 스택
6. 데이터셋
7. 모델 학습 및 저장

## 프로젝트 개요

이 프로젝트는 **사용자 맞춤형 관광지 추천 시스템**으로, 제주도의 관광지를 대상으로 합니다. 사용자가 성별, 연령대, 여행 스타일, 동반 인원수 등 다양한 정보를 입력하면, 해당 조건에 가장 적합한 관광지를 추천합니다. 장애인, 노약자 등 사회적 약자들이 편리하게 관광지에 접근할 수 있도록 돕기 위해 접근성 필터링 옵션을 제공하며, 추천 방식은 회귀 기반 필터링과 콘텐츠 기반 필터링을 결합한 하이브리드 필터링을 사용했습니다.

## 설계 및 구현

### 1. 설계 개요

- **하이브리드 필터링 모델**: 회귀 기반 필터링(CatBoost 사용)과 콘텐츠 기반 필터링(카테고리 간 유사도 기반)을 결합하여 각 필터링 방법의 장점을 살려 보다 정확한 추천을 제공합니다.
- **접근성 필터링 옵션**: 장애인 편의 시설 유무를 반영하여 필터링할 수 있도록 하여 사용자 경험을 향상시킵니다.

### 2. 핵심 로직 설계

1. **회귀 기반 필터링**: CatBoost 모델을 통해 사용자 입력에 따라 관광지에 대한 평점을 예측합니다.
2. **콘텐츠 기반 필터링**: 관광지 카테고리 간 유사도 계산을 통해 비슷한 특성을 가진 관광지를 추천합니다. 사용자가 선호하는 카테고리를 입력하기 용이하도록 대/중/소분류 중 중분류 컬럼을 이용하였습니다.
3. **하이브리드 추천 구성**: 회귀 기반 필터링의 예측 평점과 콘텐츠 기반 필터링의 유사도의 가중평균을 계산하여 최종 추천 리스트를 생성합니다.

## 추천 시스템 구성

- **사용자 입력 처리**: 성별, 연령대, 동반 인원수, 여행 스타일, 선호하는 카테고리 등 8종류의 사용자 정보를 입력받으며, 이 데이터는 필터링 모델을 통한 예측에 사용됩니다.

- **유사도 매트릭스**: 관광지 카테고리 간 유사도를 미리 계산하여, 콘텐츠 기반 필터링에서 활용합니다. 22x22 유사도 DataFrame으로 저장됩니다. 카테고리가 같아 유사도가 1인 경우 과대평가되는 것을 방지하기 위해 0.7로 계산합니다.
- **가중치 조절**: 회귀 기반 필터링과 콘텐츠 기반 필터링 결과를 조합하기 위해 가중치를 조정할 수 있도록 설계하여 추천 결과의 유연성을 높입니다. 현재 설정은 회귀 기반 평점 예측 점수(0.8) : 콘텐츠 기반 유사도(0.2)의 가중치를 부여하여 가중평균을 계산하고 있습니다.

## 접근성 필터링 구현

1. **필터링 항목**: 휠체어 사용 가능 여부, 장애인 주차장, 장애인 화장실, 안내견 출입 가능 여부, 점자 가이드, 한국어 오디오 가이드 등 여섯 가지 항목이 있습니다.
2. **Streamlit에서 체크박스 구현**: 각 필터 항목은 체크박스 형태로 제공되며, 사용자가 선택한 접근성 조건을 바탕으로 추천 결과에서 해당 조건을 반영하도록 구현했습니다.
3. **필터링 적용 방식**: 접근성 조건에 따라 관광지 데이터가 필터링되어 최종 추천 리스트에 반영됩니다.

## 기술 스택

- **프로그래밍 언어**: Python
- **프레임워크**: Streamlit (웹 애플리케이션 인터페이스)
- **머신러닝 모델**: CatBoost (회귀 기반 필터링에 사용)
- **데이터 분석**: Pandas, Numpy (데이터 처리 및 분석)

## 데이터셋

이 프로젝트에서는 두 가지 데이터셋을 사용하였습니다:

1. **국내 여행로그 데이터(제주도 및 도서지역) (2023)**: AI허브에서 제공하는 데이터셋으로, 제주도와 관련된 여행 로그 정보를 포함하고 있습니다. ([링크](https://www.aihub.or.kr/aihubdata/data/view.do?currMenu=&topMenu=&aihubDataSe=data&dataSetSn=71780))  
   - **주요 용도**: 콘텐츠 기반 필터링과 회귀 모델 학습에 활용되어, 제주도 내 주요 여행지 추천 시 기반 데이터로 사용됩니다.
   
2. **전국 배리어프리 문화예술관광지**: 빅데이터 문화포털에서 제공하는 데이터셋으로, 배리어프리 접근성을 갖춘 문화 및 예술 관광지 정보를 포함하고 있습니다. ([링크](https://www.bigdata-culture.kr/bigdata/user/data_market/detail.do?id=573acce0-0337-11ee-a67e-69239d37dfae))  
   - **주요 용도**: 접근성 필터링 옵션에 필요한 조건을 제공하여, 장애인 편의시설이 구비된 관광지로 필터링합니다.

## 모델 학습 및 저장

1. **CatBoost 모델 학습**: 성별, 연령대, 여행 스타일 등의 입력 데이터를 바탕으로 각 관광지에 대한 예상 평점을 예측하는 회귀 모델을 학습합니다.
2. **모델 저장 및 로드**: CatBoost를 통해 학습된 회귀 모델은 pickle 파일로 저장하였으며, 실시간 추론 시 모델을 재학습할 필요 없이 빠르게 예측을 수행할 수 있습니다.

## 요약

이 프로젝트는 사용자 맞춤형 관광지 추천을 위해 다양한 입력 조건과 접근성 옵션을 반영하여 최적의 관광지 리스트를 제공합니다. 하이브리드 필터링 방식을 통해 예측 평점과 유사도 기반 추천을 결합하여 보다 정확한 추천 결과를 얻도록 설계되었습니다.
