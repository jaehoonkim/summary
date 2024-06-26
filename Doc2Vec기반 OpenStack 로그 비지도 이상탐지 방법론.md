1. 서론
    1. 클라우드 컴퓨팅으로 인해 자원 관리가 용이해졌다
    2. 효율적인 관리를 위해 로그 이상탐지를 한다
    3. 이용자 행위 패턴으로 이상 패턴을 정확하게 판변하는 방법론을 개발하는게 목적이다.
2. 선행연구
    1. 로그 이상탐지 프레임워크
        1. 이상의 정의: 기록된 의심스러운 활동
    2. 로그 이상탐지
        1. 로그 수집 → 로그 파싱 → 특징 추출 → 분석(이상탐지)
    3. 비지도 머신러닝, 딥러닝 기반 로그 이상탐지  
    ![alt text](<Doc2Vec기반 OpenStack 로그 비지도 이상탐지 방법론/image-20240329-051342.png>)
        1. 로그 메세지의 이상탐지를 위한 방법론
            1. 1st AutoEncoder → Isolattion Forest → 2nd AutoEncoder
                1. 이상과 상관없는 Latent Vector 추출
                2. Vector를 이상과 정상으로 추론
                3. 정상 패턴 확보
        2. 레이블이 존재하지 않는 데이터셋(로그)를 정상 패턴들만을 학습 시킴으로써 성능을 높이기 위한 전략
    4. Doc2Vec
        1. 문장 또는 문서에 대한 임베딩 벡터 추출 알고리즘
        2. 본 논문에서는 PV-DM방식으로 추론
3. 방법론
    1. 연구 모형  
    ![alt text](<Doc2Vec기반 OpenStack 로그 비지도 이상탐지 방법론/image-20240329-051445.png>)
        1. 로그 데이터 그룹화(시간 기준 윈도우 슬라이딩 전처리 방식)
        2. 슬라이딩된 각의 윈도우 데이터를 하나의 document로 가정 후 Doc2Vec으로 추론
        3. MSE를 손실함수로 AutoEncoder 학습
            1. 학습 데이터 복원 오차의 90% 수준 신뢰 구간의 상계점을 정상과 이상 판별 임계점으로 정의함
    2. Drain Parser
        1. Drain(로그 파싱)은 로그 의 매개변수 부분을 마스킹 처리 후 텍스트 시퀀스 일치 여부를 확인
        2. 리프 노드까지 일치하는 로그는 EventID를 할당
        3. EventID를 활용하여 Doc2Vec 알고리즘을 학습
4. 실험 및 결과
    1. 분석 데이터  
    ![!\[alt text\](<Doc2Vec기반 OpenStack 로그 비지도 이상탐지 방법론/image-20240329-051342.png>)](<Doc2Vec기반 OpenStack 로그 비지도 이상탐지 방법론/image-20240329-052646.png>)
        1. 정상(207,524), 이상(112)개의 로그
        2. 5초 단위로 윈도우 슬라이딩 처리
            1. VM 인스턴스 주소를 포함하는 로그 메시지
        3. 정상 윈도우 데이터에 비해 이상 윈도우 데이터 크기가 상대적으로 크다.  
        ![alt text](<Doc2Vec기반 OpenStack 로그 비지도 이상탐지 방법론/image-20240329-052935.png>)
        4. 데이터셋은  
        ![!\[alt text\](image-1.png)](<Doc2Vec기반 OpenStack 로그 비지도 이상탐지 방법론/image-20240329-053135.png>)
            1. dataset1은 Doc2Vec 학습을 위해서 사용
            2. dataset2는 2번째 AutoEncoder 학습에 사용
            3. dataset3은 AutoEncoder 성능 검증에 사용
    2. 알고리즘 별 매개변수 설정
        1. 단어 앞, 뒤 단어의 개수는 10
        2. 임베딩 벡터 크기는 300
        3. 학습률은 0.025
        4. AutoEncoder 은닉층은 FC, 활성함수는 tanh
    3. 비교 알고리즘별 최종 성능  
    ![!\[alt text\](<Doc2Vec기반 OpenStack 로그 비지도 이상탐지 방법론/image-20240329-051342.png>)](<Doc2Vec기반 OpenStack 로그 비지도 이상탐지 방법론/image-20240329-053438.png>)
        1. Unsupervised Anomaly Detection vs LogBERT vs Doc2Vec+AutoEncoder
            1. Unsupervised Anomaly Detection은 Isolation Forest의 특성상 Precision 성능이 매우 떨어짐
            2. 나머지 두 방법은 판별 성능이 비교적 높음
    4. 비교 알고리즘별 리소스 사용량 및 추론시간 측정
        1. Colab(무료), Epoch을 10회로 측정
        2. Unsupervised Anomaly Detection
            1. 추론 시간: 평균 29.11초, 최대 32.64초, 최소 28.07초
            2. 메모리: 평균 15.51KB, 최대 71.83KB, 최소 1KB
        3. LogBERT
            1. 추론 시간: 평균 116.02초, 최대 118.26초, 최소 112.09초
            2. 메모리: 평균 114.30KB, 최대 977.953KB, 최소 4.3KB
        4. Doc2Vec+AutoEncoder
            1. 추론시간: 평균 35.2초, 최대34.87초, 최소 25.26초
            2. 메모리: 평균 17.29KB, 최대 54.41KB, 최소 2KB
        5. LogBERT와 제안된 방법론은 비슷한 성능을 보임
5. 결론  
![!\[alt text\](<Doc2Vec기반 OpenStack 로그 비지도 이상탐지 방법론/image-20240329-051342.png>)](<Doc2Vec기반 OpenStack 로그 비지도 이상탐지 방법론/image-20240329-050821.png>)
    1. PCA로 이상 분포를 시각화
    2. 일반적인 전처리 방식으로는 이상 패턴을 완벽하게 판별하기 어려움
    3. 방법
        1. 시간 기분 윈도우 슬라이딩으로 정상과 이상을 분리하는 전처리과정
        2. Doc2Vec 판별 알고리즘 사용
        3. 판별에는 AutoEncoder 사용
        4. 선행 연구에서 제안한 여러 방법론을 비교 실험으로 본 연구에서 제안하는 방법론의 효율성 입증
    4. 향후 로그 데이터 이상탐지 연구에서고민 해야할 부분과 방향성 제시의 의미가 있음
