# FASTCAMPUS EDA
- 프로젝트 기간
- 프로젝트 인원: 5명
  |이름|담당 역할|
  |---|---|
  |박태근|팀장, 데이터셋 전처리, 카테고리별 평균 매출 / 가겨분포 분석, 거래일자에 따른 거래내역 분석, 가격대별 강의 분류 및 분석, 쿠폰 사용에 따른 거래내역 분석
  |오익수|고객별 구매 행동 분석|
  |이민교|판매가격, 실거래금액, 쿠폰할인액, 거래금액, 카테고리별 할인율간 상관관계 분석|
  |이정훈|카테고리별 판매수량 & 총 매출 비율 분석, 거래일자에 따른 거래내역 분석 및 시각화, 쿠폰 종류별 분류 작업 빛 쿠폰 사용에 따른 거래내역 분석|
  |지주영|외부 데이터 수집 및 패스트 캠퍼스 연관 단어 빈도 분석|

# 데이터 전처리
  - 컬럼제거
    - 값이 하나 밖에 없는 `['사이트']` 컬럼 제거
      ```python
      # 컬럼 '사이트'는 제거하겠습니다.
      df.drop('사이트', axis=1, inplace=True)
      ```
  - 데이터타입 변경
    - `['판매가격, '쿠폰할인액', '거래금액', '환불금액']` 컬럼은 숫자 계산이 용이함으로 데이터타입을 `int64`로 변경
      ```python
      # 컬럼 ['판매가격', '쿠폰할인액', '거래금액', '환불금액'] 의 '-'값을 0으로 치환
      df[['판매가격', '쿠폰할인액', '거래금액', '환불금액']] = df[['판매가격', '쿠폰할인액', '거래금액', '환불금액']].replace('-', 0)

      # 컬럼 ['판매가격', '쿠폰할인액', '거래금액', '환불금액'] 의 데이터 타입을 int로 변경
      df[['판매가격', '쿠폰할인액', '거래금액', '환불금액']] = df[['판매가격', '쿠폰할인액', '거래금액', '환불금액']].astype(int)
      ```
    - `['거래일자']`는 날짜계산이 용이하도록 `datetime`으로 변경
      ```python
      # 컬럼 ['거래일자'] 를 datetime 타입으로 변경
      df['거래일자'] = pd.to_datetime(df['거래일자'].str.replace('오후', 'PM').str.replace('오전', 'AM'),
                                    format='%Y. %m. %d. %p %I:%M:%S')
      ```
  - 불필요한 row 제거
    - `['실거래금액']`이 `['판매가격']`보다 액수가 큰 이상거래, 점검용 데이터는 제거
      ```python
      # 판매가격 < 실거래금액 인 데이터 제거
      df = df[df['판매가격'] >= df['실거래금액']]
      
      # 시스템 점검등 정상적인 거래로 보이지 않는 데이터를 제거하겠습니다.
      keywords = ['직원', '검수', 'test', 'crm', 'CX', '테스트']
      filtered_data = df[df['쿠폰이름'].str.contains('|'.join(keywords))]
      df = df.drop(filtered_data.index)
      ```

# EDA
- 진행과정
  1. 거래금액 등 가격 정보간의 상관관계 분석
  2. 카테고리에 따른 판매량 조사 및 분석
  3. 가격대별 강의 분류 및 상관관계 분석
  4. 거래 일자에 따른 판매량 조사 및 상관관계 분석
  5. 고객에 따른 구매 행동 분석
  6. 쿠폰 종류별 분류 및 구매 내역간의 상관관계 분석

1. 거래금액 등 가격 정보간의 상관관계 분석
   - 판매가격, 쿠폰할인액, 거래금액 간 상관관계 분석
     ```python
     # 카테고리를 기준으로 판매가격, 거래금액, 쿠폰할인액 추출
  
     # '거래일자'의 월을 추출해서 '월' 컬럼 생성
     df['월'] = df['거래일자'].dt.month 
     df_category = pd.pivot_table(data = df,
                                   index = ['카테고리', '월'],
                                   values = ['거래금액', '판매가격','쿠폰할인액']).astype(int)
  
     # 거래금액, 쿠폰할인액 판매가격 간의 상관관계 분석
     df_category_corr = df_category.corr(numeric_only=True)
  
     # 관계 분석한 자료를 기준으로 heatmap 그래프를 통해 시각화
     sns.heatmap(data = df_category_corr, annot = True, cmap='Blues')
     ```
     <img width="250" alt="Screenshot 2024-04-11 at 6 48 55 PM" src="https://github.com/je0nh/yd-edapj/assets/145730125/48f352bb-3ea6-4024-84a8-ea5ca02db179">
     
     ![output](https://github.com/je0nh/yd-edapj/assets/145730125/9812cfb1-ef85-46ec-9612-0d91009fb888)

   - 판매량과 할인율의 상관관계 분석
     ```python
     # 카테고리별 월별 강의 판매수를 작성
     sell_to_category = df.groupby(['카테고리', '월']).size().reset_index(name = '판매량')
     df_category = pd.pivot_table(data = df,
                             index = ['카테고리','월'],
                             values = ['판매가격','쿠폰할인액'])
     df_category['할인율'] = (df_category['쿠폰할인액'] / df_category['판매가격']) * 100

     # 판매가격 대비 쿠폰 할인액 비율
     discount_to_category = pd.pivot_table(data = df_category, index = (['카테고리', '월']), values='할인율').reset_index()

     # 할인율 pivot과 판매량 pivot merge
     merge_to_sd = pd.merge(sell_to_category, discount_to_category)

     # 할인율과 판매량간의 관계 분석
     merge_to_sd_corr = merge_to_sd.corr(numeric_only = True)

     # heatmap을 통한 할인율, 판매량간 관계 시각화
     sns.heatmap(data = merge_to_sd_corr, annot = True, cmap = 'Blues')
     ```
     ![output](https://github.com/je0nh/yd-edapj/assets/145730125/6fa6d202-4889-417b-8d68-92b919e1da4b)

     


