---
layout: post
title: '(pthon) pandas 처리 속도 개선에 대하여'
date: 2022-08-22 21:00:00 +0900
# categories: python # 별도 폴더가 생긴다 !!!
tags: [python, numpy, pandas]
---

지금 하고 있는 프로젝트에서 python pandas 와 numpy를 사용해서 데이터를 처리하는 부분이 있다.
그런데 기존에 있던 처리 속도가 느려서 이 부분을 검토하면서,
처리 속도를 높이기 위해 이것 저것 수정해서 테스트 하다 보니 알게 된 것들이 있어서 정리해 본다.

-   <span style="color:{{site.span_h4_color}}">최선의 방식은 numpy ndarray vectorization 처리다</span>
-   <span style="color:{{site.span_h4_color}}">for loop를 이용해서 모든 데이터를 순회하는 방식은 피해야 한다.
    (nested for loop 방식은 어떤 식 으로 든 개선의 여지가 있다)</span>

그럼 위 내용에 대해서 상세히 확인 해 보도록 하자.
우선 기존 소스 분석을 통해서 로직을 심플 하게 추려서 만들어 보면 다음과 같았다.

```python
import pandas as pd
import numpy as np
import sys
import time

loop_result =[]
df_big   = pd.DataFrame(np.random.random_sample((5000, 2)), columns=["X", "Y"])
df_small = pd.DataFrame(np.random.random_sample((  50, 3)), columns=["x_val", "y_val", "dummy"])

start = time.time()
for index_small in df_small.index:
    list_distance = []
    for index_big in df_big.index:
        distance = (df_small.loc[index_small, "x_val"] - df_big.loc[index_big, "X"]) ** 2 + (
            df_small.loc[index_small, "y_val"] - df_big.loc[index_big, "Y"]
        ) ** 2
        list_distance.append(distance)

    if len(list_distance) > 0:
        min_index = list_distance.index(min(list_distance))
        loop_result.append(min_index)

end = time.time()
print("elapsed (for loop)      = ", end - start)
```

형태(shape)가 서로 다른 두 개의 dataframe을 가지고 for loop를 돌면서,
서로 간의 거리를 모두 구해서 그중에 제일 작은 값을 구하는 처리를 수행하고 있었다.

보시다시피 df_big 과 df_small 은 shape 가 서로 다르다 (5000 x 2 vs 50 x 3).
df_small 의 모든 row 마다 df_big 의 모든 row 간 거리(squared euclidean distance)를 구해서 그중 min 값들을 구하고 있다.

이것을 수행 해보면 내 pc 에서는 대략 6초 정도 걸린다.
이 정도면 별거 아니라고 생각 할 수도 있겠지만 실제 코드에서 처리되는 데이터는
5000 x 50개가 아니고 수만 x 수천 단위이며 또 이런 식으로 처리되는 것이
반복해서 수행되기도 해서 결국 프로젝트에 있던 소스는 처리 시간이
수 분 이상 걸리고 있었다.

<!-- ### 개선 시도 1 (iterrows) -->
<h3> <span style="color:{{site.span_h3_color}}"> 개선 시도 1 (iterrows) </span> </h3>

처음 시도 했던 것은 for loop 부분에서 `iterrows()` 를 사용해보는 것이었다.

```python
for _, small_row in df_small.iterrows():
        list_distance = []
        for _, big_row in df_big.iterrows():
            distance = (small_row["x_val"] - big_row["X"]) ** 2 + (small_row["y_val"] - big_row["Y"]) ** 2
            list_distance.append(distance)
        #... 상동 ...
```

결과는 8초 정도로, 오히려 더 느려졌다. 인터넷 검색해보면 for 보다는 좀 빠르게 나온다고
하는데 이 경우엔 해당되지 않았다.
즉, 이걸 쓰면 무조건 빨라 진다는 건 없는 거 같다.
절대적이지 않고 테스트 해봐야만 알 수 있다.

<!-- ### 개선 시도 2 (at) -->
<h3> <span style="color:{{site.span_h3_color}}"> 개선 시도 2 (at) </span> </h3>

그럼 for loop 부분에서 `loc` 보다는 좀 더 빠르다는 `at` 으로 변경 해보자.

```python
for index_small in df_small.index:
        list_distance = []
        for index_big in df_big.index:
            distance = (df_small.at[index_small, "x_val"] - df_big.at[index_big, "X"]) ** 2 + (
                df_small.at[index_small, "y_val"] - df_big.at[index_big, "Y"]
            ) ** 2
            #... 상동 ...
```

결과는 3초 정도로 많이 빨라졌다.

<!-- ### 개선 시도 3 (vectorization) -->
<h3> <span style="color:{{site.span_h3_color}}"> 개선 시도 3 (vectorization) </span> </h3>

지금 이중 for loop 를 돌고 있는데 안쪽 for loop 대신 벡터 연산 으로 대체 해 보자.

```python
for index_small in df_small.index:
        list_distance = (df_small.at[index_small, "x_val"] - df_big["X"]) ** 2 + (
            df_small.at[index_small, "y_val"] - df_big["Y"]
        ) ** 2

        if len(list_distance) > 0:
            min_index = list_distance.argmin()
            loop_result.append(min_index)
```

df_big 에 대해서 개별 row 접근이 아닌 벡터 연산 처리로 수정을 했다.
실행 해보면 이 간단한 수정으로 0.03 초로 감소되어 무려 200배 정도 빨라졌다.

<!-- ### 개선 시도 4 (vectorization + numpy) -->
<h3> <span style="color:{{site.span_h3_color}}"> 개선 시도 4 (vectorization + numpy) </span> </h3>

0.03초 면 충분하다고 할 수도 있지만, 실제 데이터는 수십 만 건일 수도 있어서 안심할 수 없었다.
여기서 조금 더 나아가 보자.
이번엔 df_big 를 numpy array 로 변경해서 처리 해보자.

```python
for index_small in df_small.index:
        list_distance = (df_small.at[index_small, "x_val"] - df_big["X"].to_numpy()) ** 2 + (
            df_small.at[index_small, "y_val"] - df_big["Y"].to_numpy()
        ) ** 2
        #... 상동 ...
```

0.03 초에서 0.00502 초로 6배 정도 더 빨라졌다.
numpy 배열은 동일한 type으로 처리되기 때문에 불필요한 동적 type 체크 등을
수행하지 않아, pandas series 처리보다 훨씬 빠르다.

<!-- ### 개선 시도 5 (apply + vectorization + numpy) -->
<h3> <span style="color:{{site.span_h3_color}}"> 개선 시도 5 (apply + vectorization + numpy) </span> </h3>

코드엔 아직 for loop 가 하나 더 남아있는데, 인터넷에 검색해보면 나오는
수많은 pandas speed tricks 중에 이런 for loop 사용은 지양 해야 할 순위 1 번으로 되어 있다.
그렇다면 이것을 `apply` 처리로 한번 수정해 본다.

```python
apply_result =[]

start = time.time()
df_small.apply(get_distance, axis=1, args=[df_big])
end = time.time()
print("elapsed (apply) = ", end - start)
return end - start

def get_distance(small_row, arg_df_big):
    x_val = small_row["x_val"]
    y_val = small_row["y_val"]
    list_distance = (x_val - arg_df_big["X"].to_numpy()) ** 2 + (y_val - arg_df_big["Y"].to_numpy()) ** 2

    if len(list_distance) > 0:
        min_index = list_distance.argmin()
        apply_result.append(min_index)
```

수행 시 결과는 0.00273 초 가 나와서, 약간 더 빨라졌다.

위 테스트 내용을 바탕으로 기존 코드를 수정 해보니,
예제처럼 수백 수천 배 빨라졌으면 좋았겠지만 전체 코드를 모두 수정할 수 없는 상황이어서
그 정도의 개선을 이루진 못했다. 기존 코드 일부분만 수정을 했고, 그 정도의 작업으로도
전체 수행 시간을 1시간 20분 --> 10분으로 단축 시킬 수 있었다.

<!-- ### 전체 코드 -->
<h3> <span style="color:{{site.span_h3_color}}"> 전체 코드 </span> </h3>
{: .notice--accent}

[https://gist.github.com/jeremyko/b92399a94dd01db3cf5499a3a2b5afc4](https://gist.github.com/jeremyko/b92399a94dd01db3cf5499a3a2b5afc4)
