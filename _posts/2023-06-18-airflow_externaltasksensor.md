---
layout: post
title: 'airflow ExternalTaskSensor 사용 시 고려해야 할것들'
date: 2023-06-18 16:30:01 +0900
# categories: python # 별도 폴더로 관리된다 !!!
tags: [airflow,python]
---

지금 하고 있는 프로젝트에서 `airflow` `ExternalTaskSensor` 를 사용해서 다른 dag 에서 호출한것을 감지하는 부분이 있다. 
이것을 사용할 때 몇가지 고려할 사항이 있었는데 한번 정리해 본다.

예를 들어, `dag_a` 가 `dag_b` 를 호출해야 되서 다음처럼 코드를 작성했다고 해보자.
`dag_a` 에서 `TriggerDagRunOperator`, `dag_b` 에서는 `ExternalTaskSensor`를 사용 했다.

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

---

    위 예제는 단지 설명을 위해 억지로 만든 예제이다.
    `dag_b` 의 schedule 이 None 이기 때문에 `ExternalTaskSensor` 가 사용될 일은 없을 것이다. 
    `dag_b` 는 스스로의 schedule이 없고, `dag_a` 가 호출해 줘야만 실행이 되고 있다. 
    이런 경우라면 굳이 `ExternalTaskSensor`를 사용할 필요가 없을 것이다.
    `ExternalTaskSensor`가 필요한 경우는 `dag_b` 도 자체 schedule 이 존재해서, 
    실행 되는 시점에 `dag_a` 가 수행 됬는지를 감지해야 하는 경우일 것이다.
    즉 그냥 `dag_a` 가 `dag_b` 를 호출하면 되는 경우인데, 설명을 위해서 
    `ExternalTaskSensor` 를 굳이 추가했다.
    (실제 프로젝트 코드에서는 둘다 schedule 이 별도로 존재했다)

---


이제 `dag_a` 를 UI 에서 Trigger Dag 로 실행해 본다. 

그리고 `dag_a` 의 Task Instance Details 항목에서 execution_date 을 확인한다.

`dag_a` 를 실행한 시간이 execution_date 로 설정된다.

    dag_a 의 execution_date	==> 2023-06-18, 17:24:32

그리고 이제 `dag_b` 의 execution_date 도 확인해 본다

    dag_b 의 execution_date	==> 2023-06-18, 17:24:34


`dag_b` 의 execution_date 도 자신이 실행된 시간으로 설정된다. `dag_a` 의 실행 시간과는 연관 없으며, execution_date 는 동일하지 않다.

**그런데 airflow 에서는 `ExternalTaskSensor` 가 사용된 경우에는 execution_date 가 동일한 경우만 실행을 한다.**

결국 `dag_b` 는 계속 `dag_a` 를 기다리게 되고, `timeout=3600` 이 지나면 그제서야 에러로 처리가 될것이다.


**이를 해결하기 위한 방법은 `dag_a` 에서 execution_date 인자를 추가하는 것이다.**

명시적으로 `dag_a` 의 execution_date 를 `dag_b` 로 전달해서 둘다 동일한 execution_date 를 갖도록 하면 된다.

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


이렇게 수정하고 `dag_a` 를 실행하면 `dag_b` 가 정상 실행 될 것이다.

**결국 핵심은 ExternalTaskSensor 를 사용하는 경우에는 동일한 execution_date가 필요하다는 것이 되겠다.**

`ExternalTaskSensor` 를 사용하지 않는다면 execution_date 가 달라도 개별 dag, task 는 실행은 될 것이다.

만약 위 예제의 `dag_a`, `dag_b` 가 schedule 이 None 이 아닌 경우라면 어떨까?

이 경우에는 2가지 방안이 있겠는데 

- 위 예제와 동일하게 `dag_b`로 execution_date 인자를 넘겨 주거나

- `ExternalTaskSensor` 인자 중에서 `execution_delta` 나 `execution_date_fn` 중에서 하나를 선택해서 execution_date 를 일치시키게 만들면 된다. 

결국 이것도 마찬가지로 동일한 execution_date를 만들어 주는 것이다.  

`ExternalTaskSensor` 가 제대로 동작하기 위해서는 그것을 호출한 dag, task 가 어떤 것인지를 구별하게 만들어 주면 된다. 
그리고 airflow 에서는 이것을 execution_date 가 일치하는 것으로 하고 있다.









