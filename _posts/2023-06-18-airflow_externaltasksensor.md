---
layout: post
title: 'airflow ExternalTaskSensor 사용 시 고려해야 할것들'
date: 2023-06-18 16:30:01 +0900
# categories: python # 별도 폴더로 관리된다 !!!
tags: [airflow,python]
---

지금 하고 있는 프로젝트에서 `airflow` `ExternalTaskSensor` 를 사용해서 다른 dag 에서 호출한것을 감지하는 부분이 있다. 
이것을 사용할 때 몇가지 고려할 사항이 있었는데 한번 정리해 본다.

## ExternalTaskSensor 사용 시 계속 대기 상태로 빠지는 에러
`ExternalTaskSensor` 를 사용할때 더 이상 진행이 안되고 hang 발생되는 예 를 하나 만들어 보자. 
예를 들어, `dag_a` 가 `dag_b` 를 호출하고 다음처럼 코드를 작성 했다고 가정해 보자.
둘다 schedule=None 이고, `dag_a` 에서 `TriggerDagRunOperator`, `dag_b` 에서는 `ExternalTaskSensor`를 사용 했다.


### dag_a

```python
import pendulum
from airflow import DAG
from airflow.operators.trigger_dagrun import TriggerDagRunOperator

with DAG(
    dag_id="dag_a",
    start_date=pendulum.datetime(2023, 6, 1, tz="UTC"),
    catchup=False,
    schedule=None,
    tags=["kojh"],
) as dag:
    trigger = TriggerDagRunOperator(
        task_id="trigger_dag",
        trigger_dag_id="dag_b",
        conf={"message": "Hello World"},        
    )
```

### dag_b

```python
import pendulum
from airflow import DAG
from airflow.sensors.external_task import ExternalTaskSensor
from airflow.operators.bash import BashOperator

with DAG(
    dag_id="dag_b",
    start_date=pendulum.datetime(2023, 6, 1, tz="UTC"),
    catchup=False,
    schedule=None,
    tags=["kojh"],
) as dag:
    sensor = ExternalTaskSensor(
          task_id='wait_for_task',
          external_dag_id='dag_a',
          external_task_id='trigger_dag',
          timeout=3600,
      )

    bash_task = BashOperator(
        task_id="bash_task",
        bash_command='echo "Here is the message: $message"',        
        env={"message": '{{ '{{' }} dag_run.conf.get("message")  }}'  },
        
    )

    sensor >> bash_task

```


이제 `dag_a` 를 UI 에서 Trigger Dag 로 실행해 본다. 
그리고 `dag_a`, `dag_b` 의 Task Instance Details 항목에서 `execution_date` 을 각각 확인한다.
**이처럼 schedule 이 없는 경우에는 `dag_a`, `dag_b` 는 자신이 실행된 시간이 `execution_date` 로 설정된다.**

    dag_a 의 execution_date ==> 2023-06-18, 17:24:32
    dag_b 의 execution_date ==> 2023-06-18, 17:24:34

`dag_b`는 `dag_a` 의 실행 시간과 연관 없으며, execution_date 는 서로 동일하지 않다.

**그런데 airflow 에서는 `ExternalTaskSensor` 가 사용된 경우에는 `dag_b` 에서 `dag_a` 를 찾을때 execution_date 가 둘다 일치하는 지 확인한다.**

이유는 `dag_a`가 여러 번 실행 됬을 수 도 있기 때문이다. `dag_b` 입장에서는 언제 실행된 `dag_a`를 감지해야 할지 기준이 필요할것이다.
그래서 결국 `dag_b` 는 계속 `dag_a` 를 계속 기다리게 되고, `timeout=3600` 이 지나면 그제서야 에러로 처리가 될것이다.

## 이 에러를 해결 하기
이를 해결하기 위한 방법은 `dag_a` 에서 `TriggerDagRunOperator`의 `execution_date` 인자를 추가하는 것이다.
명시적으로 `dag_a` 의 execution_date 를 `dag_b` 로 전달해서 둘다 동일한 `execution_date` 를 갖도록 하면 된다.


### dag_a

```python
import pendulum
from airflow import DAG
from airflow.operators.trigger_dagrun import TriggerDagRunOperator

with DAG(
    dag_id="dag_a",
    start_date=pendulum.datetime(2023, 6, 1, tz="UTC"),
    catchup=False,
    schedule=None,
    tags=["kojh"],
) as dag:
    trigger = TriggerDagRunOperator(
        task_id="trigger_dag",
        trigger_dag_id="dag_b",
        conf={"message": "Hello World"},        
        execution_date= '{{ '{{' }} logical_date  }}'
    )
```

airflow 에서 legacy 용어 혼란이 있는데, `execution_date` 는 현재 버전에서는 `logical_date` 이다. 
둘다 같은 의미이다. 
`execution_date`는 의미를 굳이 알 필요도 없다 (오히려 혼란이 가중되므로). 
그냥 `execution_date` 는 `logical_date` 라고 생각하면 이해가 쉽다.

airflow 에서 사용되는 시간 개념이 좀 혼란스러운데, 다음의 관계가 성립된다.

    execution_date = data_interval_start = logical_date

그러므로, execution_date 는 처리할 데이터의 시작 범위를 의미한다.
        
이렇게 수정하고 `dag_a` 를 실행하면 `dag_b` 가 정상 실행 될 것이다. 
만약 `ExternalTaskSensor` 를 사용하지 않았다면 `execution_date` 가 달라도 정상 실행 될 것이다.
이게 무슨 의미냐면, 그냥 각자의 로직을 수행할것이란 뜻이다. `dag_a`가 `dag_b` 를 호출하는 관계만 정의될 뿐.

## 참고할 것

위 예제는 단지 설명을 위해 억지로 만든 예제이다. 
이런식으로 사용할 경우는 없을 것이다. 
**`execution_date` 를 `dag_b` 로 전달하는 방법은 일반적인 해결 방안이 아니다.**

다시 생각 해보면, `dag_b` 는 스스로의 schedule이 없고, `dag_a` 가 호출 해줘야 만 실행이 되고 있다.
그렇다면 `dag_a`가 `dag_b`를 호출하고 `dag_b`가 `dag_a` 를 다시 감지할 필요가 있을까 ? 
이런 경우라면 굳이 `ExternalTaskSensor`를 사용할 필요가 없을 것이고, 정말 필요한 경우는 `dag_b` 의 schedule 이 존재해서, 스케쥴러에 의해 실행 되는 시점에 (`dag_a` 에 의해서가 아닌) `dag_a` 혹은 다른 dag 가 
수행 됬는지를 감지해야 하는 경우일 것이다.




## `ExternalTaskSensor` 를 제대로 사용하려면  

`ExternalTaskSensor` 인자 중에서 `execution_delta` 나 `execution_date_fn` 중에서 하나를 선택해서 감지할 dag 의 execution_date 를 특정해서 찾을 수 있게 만들어야 한다.
       
   

## 결론 
- `ExternalTaskSensor` 가 제대로 동작하기 위해서는 감지할 dag, task 가 어떤 것인지를 구별하게 만들어 줘야 하고 airflow 에서는 이것을 찾을때 id 뿐만 아니라 execution_date 이 동일한지도 같이 참고 한다.
- 그래서 `ExternalTaskSensor` 인자 중에서 `execution_delta` 나 `execution_date_fn` 중에서 하나를 선택해서 감지할 dag의 execution_date 를 찾을 수 있게 만들어 줘야 한다.









