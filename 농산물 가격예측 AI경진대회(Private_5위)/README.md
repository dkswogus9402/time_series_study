### 대회이름 : 2021 농산물 가격예측 AI 경진대회

데이터 종류 : 시계열 데이터



Private 5위팀의 모델구성

프레임워크 : sklearn, tensorflow, statsmodels



## 데이터 전처리

### EDA 처리

![image-20220301135010248](README.assets/image-20220301135010248.png)

각 데이터들을 미리 시각화를 시켜본다. 

해당 데이터들의 특성을 파악해본다.

해당 그래프에서 얻을 수 있는 것 : 계절 패턴이 뚜렷하고 품목별로 다른 분포를 보이는 것을 알 수 있다.

따라서 해당 품목별로 다시 시각화를 진행해야 할 필요가 있음.

![image-20220301134951463](README.assets/image-20220301134951463.png)

해당 데이터를 통해 볼 수 있는 특성

대부분의 데이터가 분포하고 있는 box내부에서는 데이터들이 잘 몰려있는 것을 확인할 수 있으나,

특이값이 많이 포함되어 있는 것을 확인할 수 있습니다.

특이값은 해당 데이터만으로 파악할 수 없으므로 특이값 발생 원인에 대해서 파악하도록 합니다.

![image-20220301135227841](README.assets/image-20220301135227841.png)

건고추의 경우 장마 기간이 길어지면 건조가 되지 않아서 가격이 폭등하는 현상이 발생합니다.

특이값 예측하는 방안을 마련해야할 필요가 있음

---

### 모델링

첫번째 모델 : 랜덤포레스트

일주일 후 배추 가격을 예측

![image-20220301135534680](README.assets/image-20220301135534680.png)

주황색 선이 해당 모델링의 결과 값을 표현함

전반적인 추세는 잘 예측하지만 거래가 발생하지 않는 휴일 전후로 예측 성능이 떨어지는 모습을 보임

큰 폭으로 변동되는 가격은 잘 예측하지 못함





두번째 모델 : feature selection 수행 모델

![image-20220301135742991](README.assets/image-20220301135742991.png)



왼쪽은 배추 가격그래프이고, 오른쪽은 배추 거래량 그래프임

정상성을 띄는 시계열 데이터인 거래량과는 다르게 가격은 비정상 시계열 데이터임

![image-20220301141136649](README.assets/image-20220301141136649.png)

-> 두가지의 상관관계를 분석해봄 - 데이터는 서로 영향을 끼치지 않는 것을 확인할 수 있음

( 굉장히 서로 밀접한 관계가 있을 것이라고 판단했었는데 검증을 통한 결론만을 믿는 자세가 필요함 )



![image-20220301141332615](README.assets/image-20220301141332615.png)

배추 가격으로만 예측한 모델 : MAE가 늘었지만, 저자는 휴일을 예측하지 못해서 발생한 결과이며, 휴일 전후와 특이 값을 전보다 더

잘 예측하는 것을 확인했다고 알렸음

-> 하지만 거래가 발생하지 않는 휴일을 잘 예측하지 못한다는 한계가 존재했음



정상성을 나타내는 시계열은 시계열의 특징이 해당 시계열이 관측된 시간과 무관합니다.

따라서 추세나 계절성이 있는 시계열은 정상성을 나타내는 시계열이 아닙니다. 추세와 계절성은 서로 다른 시간에 시계열의 값에 영향을 줄 것이기 때문입니다. 



### feature engineering

휴일 가격을 예측하기 위해서 휴일 가격을 평균으로 대체했습니다.

거래가 발생하지 않은 휴일의 농산물 가격을 하루 전, 하루 후 평균 가격으로 데이터를 대체합니다.

![image-20220301141645126](README.assets/image-20220301141645126.png)

데이터를 대체하자, 휴일의 데이터를 더 잘 예측할 수 있는 것을 확인할 수 있음

데이터를 대체하는 방법과 데이터를 제외하는 방법 중 데이터를 대체하는 방법을 사용했음



### 시계열 분해 (STL 분해)

![image-20220301144045722](README.assets/image-20220301144045722.png)



STL은 다양한 상황에서 사용할 수 있는 강력한 시계열 분해 기법입니다.

Loess를 사용한 계절성과 추세 분해의 약자입니다. 여기세엇 Loess는 비선형 관계를 추정하기 위한 기법입니다. (1990년 개발)

trend, seasonal remainder에 해당되는 데이터들을 뽑아낼 수 있습니다.

해당 데이터를 추가적인 train feature로 활용하여 데이터를 예측합니다.

![image-20220301144106455](README.assets/image-20220301144106455.png)



최종 모델에서 보면 데이터를 예측할 때 좋은 결과를 관측하는 것을 확인 할 수 있습니다.



이후 랜덤포레스트 모델과 LSTM 모델의 성능을 비교하며 종료됩니다.

4주 후를 예측하는 모델은 너무 긴 기간이기 때문에, 예측이 어려워서 거의 따라가는 추세를 보임

window size는 21일을 사용함



사용했던 개념

pool = multiprocessing.Pool(processes=multiprocessing.cpu_count())

pool.map(preprocessing, tsalet_files)







EDA란?

EDA : 탐색적 데이터 분석

-> 해당 데이터에 대한 탐색과 이해를 기본으로 가져야 한다.

raw data를 접할 때 부터 데이터를 잘 이해라고 파악한 다음 어떤 결과를 만들어 낼지 

해당 feature로 필터링을 해보고, 다른 feature로 필터링을 해보고 데이터를 여러 측면으로 쪼개고,

출력해보면서 인사이트를 얻어내는 것이다.

즉 데이터 분석을 통한 결과 값을 출력하기 전에 어떤 결과 값을 낼지 가설을 가지고 기본적인 표나 그래프를 간단히

그려보며, '사전 검증' 을 하는 과정을 의미한다. -> 그렇게 할 때 진정 유의미한 데이터를 생산할 수 있다.



이를 어떻게 하면 잘할 수 있을까..

1. raw data의 묘사 및 사전을 통해 각 데이터들의 제대로 된 의미를 이해하는 기술

2. 결측치 처리 및 데이터 필터링 기술 : 데이터 분석을 본격적으로 들어가기 전, 데이터에 결측치가 없는지 확인하고 있다면 제거해줘야한다. 또한 분석할 때 필요한 데이터가 수치형 데이터라면 범주형 데이터로 변환해줘야 한다.
3. 어떤 데이터를 시각화할 때 필요한 기술, 그래프를 그릴 때 멋진 컬러, 디자인을 넣는다고 하더라도 그래프의 이해가 먼저되어야 한다.

왜 어려울 까..



누구에게나 인지 편향이 있다.

우리는 무의식적으로 다른 사람들의 이야기를 비롯한 외부의 정보를 접할 때 스스로의 경험으로 판단을 내리는 경향이 있다..

이는 기존에 경험했던 것으로 먼저 판단을 내리게 하여 에너지를 효율적으로 관리하기 때문에 생기는 편향이다.

인지적 편향을 조금 내려놓고 다른 사람들의 이야기와 정보에 대해 있는 그대로 이해할 수 있는 능력이 필요하다고 생각한다.

https://jalynne-kim.medium.com/

해당 사이트를 참고하여 글을 작성하였습니다.

