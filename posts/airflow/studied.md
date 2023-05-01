---
layout: post
title:  "찾아본 내용들"
date:   2021-10-23 01:02:59
author: Inhyuk
category: Airflow
tags:	airflow
cover:  "/assets/instacode.png"
name: studied.md
---

Context
======

- - -

실행 날짜
=======


- Airflow에서 헷갈리는 것은 **execution_date**(실행 날짜)에 관한 것이다
- airflow에서는 start_date, interval, end_date의 3개로 DAG 실행 시점을 제어할 수 있다. 시작 날짜에 시작하여 종료날짜(optional)까지 실행된다. 이를 **간격 기반(interval based)**라고 한다.

```
<아래는 2022-01-02 00:00 전에 시작날짜를 2022-01-02 00:00, 간격을 1시간으로 설정한 DAG 잡의 실행모습이다. 첫 간격 실행(첫 run 수행이 아님)은 2022-01-02 00:00에 시작된다.

--->|<----interval---->|<----interval---->|<----interval---->|
    |==================|==================|==================|==================>
2022-01-02 00:00  2022-01-02 01:00  2022-01-02 02:00  2022-01-02 03:00
    L===> 첫 간격 시작    L===> 첫 run 수행
```

- 간격 기반ㅇ의 장점은 시간 간격이 명확하기 때문에 increment(증분)처리에 적합하다는 것이다. 시점기반(point based. 예를 들어, cron)은 현재 시점의 시간만 알기 때문에 이전 작업이 실행되었다고 가정하고 실행이 끝난 위치를 계산해내야한다.
- 실행시간(execution date): 해당 간격의 시작 시간. dagrun은 간격이 끝나는 지점에 시작되기 때문에 보통 **실행 시간 = 현재시각 - 간격**이 된다. 간격의 실행시간이라고 생각하면 편하다. next_execution은 다음 간격 시작 시간이기 때문에 현재시각과 같다


```
<아래는 2022-01-02 00:00 전에 시작날짜를 2022-01-02 00:00, 간격을 1시간으로 설정한 DAG 잡의 실행모습이다. 처음 실행은 2022-01-02 00:00에 시작된다.

--->|<----interval---->|<----interval---->|<----interval---->|
    |==================|==================|==================|==================>
2022-01-02 00:00  2022-01-02 01:00  2022-01-02 02:00  2022-01-02 03:00
                       L===> execution_date = 2022-01-02 00:00 / next_execution = 2022-01-02 01:00
```

- - -

backfilling
================

- airflow에서는 과거의 시작 날짜부터 과거의 간격도 정의할 수 있다. 이 속성을 이용하여 과거의 task도 수행가능하다. 이 기능을 **백필(backfilling)**이라고 함.
- 백필은 default로 *True*로 설정 되어있다. 이 기능을 끄고 싶으면 *catchup=False*로 설정하면 된다.
- 백필은 코드를 변경한 후에 데이터를 다시 처리할 때도 사용됨. 과거의 task 결과들을 지우면 새 코드를 이용하여 백필 기능이 수행된다.

- - -

Task 의존성
=============

Linear
-------

- 기본적으로 많이 사용하는 유형이다. 하나의 task 이후에 다음 task, 그리고 그 후에 또 다른 task로 연결하는 체인이다
- 이 체인에서는 이전 task(upstream이라고 부른다)가 성공해야 그 다음 task가 실행될 수 있다

```py
task1 >> task2 >> task3
```

Fan-in, Fan-out
-----------------

- linear하게가 아니라 여러 task로 병렬적으로 분기하는 의존성이다
  - fan-in(N:1): 여러 task들이 모여서 하나의 다음 task에 연결된 구조
  - fan-ou(1:N): 하나의 task가 다음 다수의 task의 의존성이 되는 구조

```
//fan in
[task1, task2] >> next_task

//fan out
first_task >> [task1, task2]
```

Branching
---------------

- 이전 task의 결과를 이용하여 다음 task를 브랜칭(선택)하기 위해서는 서비스(또는 수행) 코드내에서 브랜칭하는 것 보다는 DAG에서 브랜치하는 것이 좋다. 이 때 이용되는 것이 **BracnhPythonOperator**이다.

```py
def branch_fun(**context):
    if context["execution_date"] < TIME:
        return "task_id_1"
    else:
        return "task_id_2"

branch_task = BranchPythonOperator(task_id="branch_task",python_callable=branch_fun)

branch_task >> [task_id_1, task_id_2]
```

- 만약 이 이후에 어떤 task를 연결한다면 그 task는 수행이 안된다. 기본 *trigger_rule*이 *all_succeed*이기 때문이다. 이를 해결하기 위해 해당 task를 *none_failed*로 변경해도 되지만, 만약에 여기에 연결된 다른 task가 있으면 꼬이기 쉽다.

```py
branch_task >> [task_id_1, task_id_2] >> join_task
task_3 >> join_task

#여기서는 task_3는 꼭 성공해야 하는데 join_task의 trigger_rule을 none_failed로 하면 task_3가 이유없기 건너뛰는 것을 방지할 수 없다.
```

- 다음 task의 trigger_rule을 직접 변경하는 것 보다는 브랜치 이후에 **DummyOperator**를 하나 두는 것이 나은 선택이다

```py
branch_task = BranchPythonOperator(task_id="branch_task",python_callable=branch_fun)
join_branching_task = DummyOperator(task_id="join_branching_task", trigger_rule="none_failed")


branch_task >> [task_id_1, task_id_2] >> join_branching_task >> join_task
task_3 >> join_task
```

trigger 규칙
-----------

- all_success(default): 모든 상위 태스크의 성공이 완료되면 실행됨
- all_failed: 모든 상위 테스크가 실패했을 경우 실행
- all_done: 모든 상위 테스크가 완료되면(성공, 실패 여부 상관 없음) 실행됨
- one_failed: 상위 테스크 중 하나라도 실패하면 실행됨. 다른 upstream task들의 결과를 기다리지 않음.
- one_success: 상위 테스크 중 하나라도 성공하면 실행됨. 다른 upstream task들의 결과를 기다리지 않음.
- none_failed: 실패(건너뛰는 경우는 포함하지 않음)한 상위 테스크가 없는 경우 실행.
- none_skipped: 건너뛴 상위 테스크가 없는 경우에 실행됨. 성공, 실패 여부는 상관 없음.
- dummy: upstream과 상관 없이 실행됨. 보통 test할 때 사용된다


XCom
-----

- airflow에서 task간의 데이터 공유는 **XCom**을 통해서 이루어짐. Taskflow API의 경우에도 내부적으로 XCom을 사용하는 decorator 형식이다.
- XCom 데이터는 metastore DB에 저장된다. 그렇기 때문에 Serializable 데이터만 가능하며 데이터 크기에 제한이 있다.

```py
#push(저장)
def task_1(**context):
    context["task_instance"].xcom_push(key="key_name", value="abc")

#pull(사용)
def task_2(**context):
    value = context["task_instance"].xcom_pull(task_ids="task_1", key="key_name")

########### 데이터가 하나이면 간편하게 return할 수 도 있다
def task_1(**context):
    return "abc"

#pull(사용)
def task_2(**context):
    value = context["task_instance"].xcom_pull(task_ids="task_1", key="return_value") #return_value이면 key를 안넣어도 된다. default로 들어간다


########### 다른 dag나 다른 execution_date의 값도 가져올 수 있다. 그러나 웬만하면 쓰지 말자
def task_2(**context):
    value = context["task_instance"].xcom_pull(dag_id = "other_dag", task_ids="task_1", key="return_value") #return_value이면 key를 안넣어도 된다. default로 들어간다
```
