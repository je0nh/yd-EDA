# FASTCAMPUS EDA
- 프로젝트 목표
  - Pandas 라이브러리를 이용해 FASTCAMPUS 쿠폰 마케팅 전략 분석
  - Matplotlib, Seaborn을 이용해 분석한 자료 시각화
- 프로젝트 기간: 23.5.9 ~ 23.5.17 (1 주)
- 프로젝트 인원: 5명
  |이름|담당 역할|
  |---|---|
  |박태근|팀장, 데이터셋 전처리, 카테고리별 평균 매출 / 가겨분포 분석, <br> 거래일자에 따른 거래내역 분석, 가격대별 강의 분류 및 분석, 쿠폰 사용에 따른 거래내역 분석
  |오익수|고객별 구매 행동 분석|
  |이민교|판매가격, 실거래금액, 쿠폰할인액, 거래금액, 카테고리별 할인율간 상관관계 분석|
  |이정훈|카테고리별 판매수량 & 총 매출 비율 분석, 거래일자에 따른 거래내역 분석 및 시각화, <br> 쿠폰 종류별 분류 작업 빛 쿠폰 사용에 따른 거래내역 분석|
  |지주영 &nbsp; &nbsp;|외부 데이터 수집 및 패스트 캠퍼스 연관 단어 빈도 분석|

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

     ![output](https://github.com/je0nh/yd-edapj/assets/145730125/c822aa39-f817-4b18-9a70-3e688f86d7ed)

2. 카테고리별 판매량 조사 및 분석
   - 카테고리별 판매량과 매출분석을 위해 `REFUND` 값 제거
     ```python
     # '유형' 컬럼에서 'REFUND' 제거
     df = df[df['유형'] != 'REFUND']

     # 카테고리별 판매량
     df_category_sales = df.groupby('카테고리').size().reset_index(name='판매량').sort_values(by='판매량', ascending=False).set_index('카테고리')

     # 카테고리별 총매출
     df_category_amount = df.groupby('카테고리')['실거래금액'].sum().reset_index(name='총매출').set_index('카테고리').sort_values('총매출', ascending=False)

     # 카테고리별 평균 실거래금액
     df_category_avg_amount = df.groupby('카테고리')['실거래금액'].mean().reset_index(name='평균실거래금액').set_index('카테고리').sort_values('평균실거래금액', ascending=False).astype(int)

     # 카테고리별 상품수 대비 평균 실거래금액
     df_category_amount_sum = df.groupby('카테고리')['실거래금액'].sum()
     df_category_course_amount = df.groupby('카테고리')['코스ID'].nunique()

     df_category_course_avg_amount = (df_category_amount_sum / df_category_course_amount).reset_index(name='상품수 대비 평균실거래금액').set_index('카테고리').sort_values('상품수 대비 평균실거래금액', ascending=False).astype(int)

    ![output](https://github.com/je0nh/yd-edapj/assets/145730125/7904ba24-e212-4572-af76-dc0532632c93)

3. 거래 일자에 따른 판매량 조사 및 상관관계 분석
   - 월별, 요일별, 시간별 판매수량에 대해 분석
     ```python
     # 요일별 컬럼 생성
     df['요일'] = df['거래일자'].dt.day_name()

     # 시간별 컬럼 생성
     df['시'] = df['거래일자'].dt.hour

     # 월별 판매수량
     df_month = df.groupby('월').size().reset_index(name='판매 수량').set_index('월')

     # 요일별 판매수량
     df_week = df.groupby('요일').size().reset_index(name='판매 수량').set_index('요일').sort_values('판매 수량', ascending=False)

     # 시간별 판매수량
     df_time = df.groupby('시').size().reset_index(name='판매 수량').set_index('시')
     ```
     ![output](https://github.com/je0nh/yd-edapj/assets/145730125/7c43cc60-d44b-47aa-bf83-50c9d665bd0f)

   - 쿠폰 사용 여부에 따른 판매량 차이 분석
     ```python
     # '쿠폰 유무' 컬럼 생성에서 '쿠폰이름'에 '-'값이 없으면 1, 있으면 0을 넣어줌
     df['쿠폰유무'] = df['쿠폰이름'].apply(lambda x: 1 if '-' not in x else 0)

     # 쿠폰 유무에 따른 월별 판매량을 보여주는 피벗 테이블
     coup_monsale = df.groupby(['쿠폰유무','월']).size().reset_index(name='판매량')
     coup_monsale_pivot = coup_monsale.pivot(index='월', columns='쿠폰유무', values='판매량')
     coup_monsale_pivot['total'] = coup_monsale_pivot[0] + coup_monsale_pivot[1]

     # 쿠폰 유무에 따른 요일별 판매량을 보여주는 피벗 테이블
     weekday_order = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday']

     coup_daysale = df.groupby(['쿠폰유무','요일']).size().reset_index(name='판매량')
     coup_daysale_pivot = coup_daysale.pivot(index='요일', columns='쿠폰유무', values='판매량')
     coup_daysale_pivot = coup_daysale_pivot.reindex(weekday_order)
     coup_daysale_pivot['total'] = coup_daysale_pivot[0] + coup_daysale_pivot[1]
     ```
     ![output](https://github.com/je0nh/yd-edapj/assets/145730125/9b773a5b-74b0-4f8a-ae91-dd6cc696896d)

   - 월별 제공되는 쿠폰수
     - 판매량이 많은 달에 쿠폰을 얼만큼 제공했는지 알기위해 분석
       ```python
       # 월별로 쿠폰의 종류수 확인
       df_coupon = df.groupby(['월', '쿠폰이름']).size().reset_index(name='쿠폰수')
       df_coupon = df_coupon[df_coupon['쿠폰이름'] != '-'][['월','쿠폰이름']].groupby('월').count()

       # 월별 실거래금액
       df_month = df.groupby('월')['실거래금액'].sum().reset_index(name='실거래금액')
       df_month = df_month.reset_index(drop=True)
       
       combined_df = pd.merge(df_coupon, df_month, on='월')
       combined_df.columns = ['월', '월별 쿠폰종류수', '월별 실거래금액 총합계']
       ```
       ![output](https://github.com/je0nh/yd-edapj/assets/145730125/23365918-9ed8-4c22-8ff9-5f0966e6f94f)

    - 시간에 따른 판매 가격 변화
      ```python
      # 가장 판매가 많은 상품
      df.groupby('코스(상품) 이름').size().reset_index(name='판매량').sort_values(by='판매량', ascending=False).head(10)

      # 첫번째 가격과 마지막 가격의 차이를 보여주는 표
      df_grouped = df.groupby(['코스(상품) 이름']).agg({'거래일자': ['first', 'last'], '판매가격': ['first', 'last']}).reset_index()
      df_grouped.columns = ['코스(상품) 이름', '마지막 거래일자', '첫번째 거래일자', '마지막 판매가격', '첫번째 판매가격']

      # 가장 큰 판매가격 차이를 보이는 상품
      max_price_difference_row = df_grouped.loc[df_grouped['마지막 판매가격'] - df_grouped['첫번째 판매가격'] == (df_grouped['마지막 판매가격'] - df_grouped['첫번째 판매가격']).max()].reset_index()

      df_allinone = df[df['코스(상품) 이름'].str.contains('초격차 패키지 : 일잘러 필수 스킬 모음.zip')]
      df_allinone = df_allinone[['거래일자', '판매가격']]
      df_allinone = df_allinone.groupby('거래일자').mean().reset_index()
      df_allinone['거래일자'] = pd.to_datetime(df_allinone['거래일자'])
      df_allinone = df_allinone.sort_values(by='거래일자', ascending=True)
      ```
      ![output](https://github.com/je0nh/yd-edapj/assets/145730125/64e47a92-b810-4791-9b72-9b2fd4a2d070)

4. 매출 상위 20%에 해당하는 고객군과 나머지 고객군 간의 비교분석
   - 매출 상위 20%에 해당하는 고객군 분류
     ```python
     df_modi = df[['고객id', '카테고리', '코스(상품) 이름', '판매가격', '실거래금액', '쿠폰할인액', '거래금액', '쿠폰유무']]

     df_modi['row_count'] = 1
     df_modi = df_modi.sort_values(by='실거래금액', ascending=False)

     # 고객id 그룹 및 실거래금액순 정렬
     top_count_id = df_modi.groupby('고객id').sum('실거래금액').reset_index().sort_values(by='실거래금액', ascending=False)
     top_20_df = top_count_id.head(len(top_count_id)//5)

     print('상위 20% 고객 정보')
     # 상위 20% 고객의 수
     print(' - 고객 수:', len(top_20_df))
      
     # 상위 20% 총매출
     print(' - 총매출:', '{:,.0f} 원'.format(top_20_df['실거래금액'].sum()))
      
     # 상위 20% 고객의 총매출이 전체 매출에서 차지하는 비율
     print(' - 총매출이 전체 매출에서 차지하는 비율:', round(top_20_df['실거래금액'].sum() / df['실거래금액'].sum(),4) * 100, '%')
      
     # 상위 20% 고객의 평균 거래횟수
     print(' - 평균 거래횟수:', round(top_20_df['row_count'].mean(),2), '회')
      
     # 상위 20% 고객의 평균 실거래금액, 원으로 표현
     print(' - 평균 실거래금액:', '{:,.0f} 원'.format(round(top_20_df['실거래금액'].mean())))
      
     # 상위 20% 고객의 한 사람당 평균 쿠폰 이용 건수
     print(' - 한사람당 평균 쿠폰 이용 건수:', round(top_20_df['쿠폰유무'].mean(),2), '건')
      
     # 상위 20% 고객이 쿠폰으로 할인되는 금액의 비율
     print(' - 쿠폰으로 할인되는 금액의 비율:', round(top_20_df['쿠폰할인액'].sum() / top_20_df['판매가격'].sum(),4) * 100, '%')
      
     # 상위 20% 고객의 평균 쿠폰 할인액, 원으로 표현
     print(' - 평균 쿠폰 할인액:', '{:,.0f} 원'.format(round(top_20_df['쿠폰할인액'].mean())))

     # 그 외 80% 고객 정보
     print('상위 20% 고객을 제외한 나머지 80% 고객 정보')

     # 나머지 80% 고객의 수
     print(' - 고객 수:', len(df) - len(top_20_df))
      
     # 나머지 80% 고객의 총매출
     print(' - 총매출:', '{:,.0f} 원'.format(df['실거래금액'].sum() - top_20_df['실거래금액'].sum()))
      
     # 나머지 80% 고객의 총매출이 전체 매출에서 차지하는 비율
     print(' - 총매출이 전체 매출에서 차지하는 비율:', round((df['실거래금액'].sum() - top_20_df['실거래금액'].sum()) / df['실거래금액'].sum(),4) * 100, '%')
      
     # 나머지 80% 고객의 평균 거래횟수
     print(' - 평균 거래횟수:', round((len(df) - top_20_df['row_count'].sum()) / (len(df.groupby('고객id')) - len(top_20_df)),2), '회')
      
     # 나머지 80% 고객의 평균 실거래금액, 원으로 표현
     print(' - 평균 실거래금액:', '{:,.0f} 원'.format(round((df['실거래금액'].sum() - top_20_df['실거래금액'].sum()) / (len(df.groupby('고객id')) - len(top_20_df)))))
      
     # 80% 고객의 한사람당 평균 쿠폰 이용 건수
     print(' - 한사람당 평균 쿠폰 이용 건수:', round((df['쿠폰유무'].sum() - top_20_df['쿠폰유무'].sum()) / (len(df.groupby('고객id')) - len(top_20_df))), '건')
      
     # 80% 고객이 쿠폰으로 할인되는 금액의 비율
     print(' - 쿠폰으로 할인되는 금액의 비율:', round((df['쿠폰할인액'].sum() - top_20_df['쿠폰할인액'].sum()) / (df['판매가격'].sum() - top_20_df['판매가격'].sum()), 4) * 100, '%')
      
     # 80% 고객의 평균 쿠폰 할인액, 원으로 표현
     print(' - 평균 쿠폰 할인액:', '{:,.0f} 원'.format(round((df['쿠폰할인액'].sum() - top_20_df['쿠폰할인액'].sum()) / (len(df.groupby('고객id')) - len(top_20_df)))))
     
     # 실거래금액 상위 20퍼센트 고객 구하기
     top_20 = df_modi.groupby('고객id')['실거래금액'].sum().sort_values(ascending=False).head(int(len(df_modi.groupby('고객id'))*0.2))
     top_20_customer = list(top_20.index)
     top_20_df = df[df['고객id'].isin(top_20_customer)]
     top_20_group = top_20_df.groupby(['고객id', '카테고리'])['실거래금액'].sum().reset_index(name='거래금액합')
     top_20_category_sum = top_20_group.groupby('카테고리')['거래금액합'].sum()

     # 실거래금액 상위 80퍼센트 고객 구하기
     top_80 = df_modi.groupby('고객id')['실거래금액'].sum().sort_values(ascending=False).tail(int(len(df_modi) - int(len(df_modi.groupby('고객id'))*0.2)))
     top_80_customer = list(top_80.index)
     top_80_df = df[df['고객id'].isin(top_80_customer)]
     top_80_group = top_80_df.groupby(['고객id', '카테고리'])['실거래금액'].sum().reset_index(name='거래금액합')
     top_80_category_sum = top_80_group.groupby('카테고리')['거래금액합'].sum()
     ```
     ![output](https://github.com/je0nh/yd-edapj/assets/145730125/f4fdc853-48c6-4c61-9281-89ff0e6800cd)

5. 쿠폰 종류별 분류 및 구매 내역간의 상관관계 분석
- 쿠폰 종류별 효과 분석
  - `PROMOTION`으로 할인을 많이 받은 고객은 `무료수강권`으로, 나머지는 `기타`로 쿠폰 종류를 분류
  - 재구매 고객 중 종류에 따라 어떤 차이가 있는지 확인
    ```python
    import re
    
    # 쿠폰 이름만 가져오기 (.unique로 중복제거)
    coup_list = list(df['쿠폰이름'].unique())
    category_coup = []
    category_coups = []
    
    for i in range(len(coup_list)):
        try:
            result = re.findall('\[(.*?)\]', coup_list[i]) # 특수문자 제거
            if result:  # Check if the result is not empty
                category_coup.append('[' + result[0] + ']')
                category_coups.append(result[0])
        except TypeError: # NaN 값을 pass
            pass
                
    coup_list = sorted(list(set(category_coup)))
    coup_lists = sorted(list(set(category_coups)))
    
    # coup_dic으로 딕셔너리 생성
    coup_dic = {name:value for name, value in zip(coup_list, coup_lists)}
    
    # 쿠폰이름에 coup_dic의 해당하는 name이 있을 경우 value를 df['쿠폰 종류']이라는 column에 저장
    def check_coup(row):
        name = row['쿠폰이름']
        for pattern, value in coup_dic.items():
            if pattern in str(name):
                return value
        return ''
    
    df['쿠폰 종류'] = df.apply(check_coup, axis=1)

    df.reset_index(drop=True, inplace=True)

    # 결제 수단이 'PROMOTION'인 경우 '쿠폰 분류' 컬럼에 를 '무료수강권'으로 저장
    
    coupon_sort = []
    
    for i in range(len(df)):
        if df.loc[i,'쿠폰이름'] != '-' and df.loc[i,'결제수단'] == 'PROMOTION':
            coupon_sort.append('무료수강권')
        elif df.loc[i,'쿠폰이름'] != '-' and df.loc[i,'결제수단'] != 'PROMOTION':
            coupon_sort.append('기타')
        else:
            coupon_sort.append('-')
    
    df['쿠폰 분류'] = coupon_sort

    # '쿠폰 종류' 컬럼 제거
    df.drop('쿠폰 종류', axis=1, inplace=True)
    ```

  - 고객별 첫 구매 리스트
    - 재구매 고객군과 첫 구매 고객군 구분
      ```python
      df_purchase_counts = df.groupby('고객id').size().reset_index(name='거래횟수')
      ```

    - 재구매 고객들의 **첫 구매** 리스트 추출
      ```python
      # 재구매 고객들의 첫구매 리스트
      repurchase = df_purchase_counts[df_purchase_counts['거래횟수'] > 1]
      repurchase_customer = list(repurchase['고객id'])
      repurchase_df = df[df['고객id'].isin(repurchase_customer)]
      repurchase_df = repurchase_df.sort_values(by=['고객id', '거래일자'])
      repurchase_df = repurchase_df.reset_index(drop=True)

      first_df = repurchase_df.drop_duplicates(subset=['고객id'], keep='first')
      first_df = first_df[['고객id', '코스(상품) 이름', '거래일자','쿠폰이름', '쿠폰 분류', '판매가격','쿠폰할인액', '실거래금액']]
      ```

    - 2회 이상 구매한 고객 중 첫 구매로 `무료수강권` 쿠폰을 사용한 고객
      ```python
      first_free_df = first_df[first_df['쿠폰 분류'] == '무료수강권']
      ```

    - 2회 이상 구매한 고객 중 첫구매로 `기타 쿠폰`을 사용한 고객
      ```python
      first_etc_df = first_df[first_df['쿠폰 분류'] == '기타']
      ```

    - 2회 이상 구매한 고객들 중 첫 구매에 **쿠폰을 사용하지 않은** 사람들
      ```python
      first_no_coupon_df = first_df[first_df['쿠폰 분류'] == '-']
      ```

    ![output](https://github.com/je0nh/yd-edapj/assets/145730125/b2ddd53a-e63d-463b-8e80-b08f93540763)

    ![output](https://github.com/je0nh/yd-edapj/assets/145730125/800a6c25-9e36-473f-8d75-ff41f2d37a50)

    ![output](https://github.com/je0nh/yd-edapj/assets/145730125/68d28ca0-f456-4f58-93fc-77e1eb93e0e3)

  - 재구매 고객 중 `첫 구매에서 쿠폰을 사용한 사람`과 `사용하지 않은 사람`들이 받은 __총 할인금액 대비 총 실거래 금액 비율__ 확인
    - 고객들이 첫 구매에 쿠폰을 사용한 경우와 사용하지 않은 경우의 `최초의 구매경험`은 `재구매에도 영향`이 있지 않을까?
      ```python
      first_df = pd.read_csv('./data/고객 id별 첫번째 구매.csv')
      
      df['판매금액'] = df['실거래금액'] + df['쿠폰할인액']
      
      used_coup = first_df[first_df['쿠폰유무'] == 1]
      nused_coup = first_df[first_df['쿠폰유무'] == 0]
      
      used_id = used_coup['고객id'].tolist()
      nused_id = nused_coup['고객id'].tolist()

      # 첫 구매에 쿠폰을 사용한 사람
      total_list = []
      
      for i in used_id:
          if df[df['고객id'] == i]['쿠폰할인액'].sum() != 0:
              a = df[df['고객id'] == i]['판매금액'].sum() / df[df['고객id'] == i]['쿠폰할인액'].sum()
              total_list.append(a)
          else:
              total_list.append(0)

      format(sum(total_list) / len(total_list), '.2%')

      # 첫 구매에 쿠폰을 사용하지 않은 사람
      total_list = []
      
      for i in nused_id:
          if df[df['고객id'] == i]['쿠폰할인액'].sum() != 0:
              a = df[df['고객id'] == i]['판매금액'].sum() / df[df['고객id'] == i]['쿠폰할인액'].sum()
              total_list.append(a)
          else:
              total_list.append(0)

      format(sum(total_list) / len(total_list), '.2%')
      ```


