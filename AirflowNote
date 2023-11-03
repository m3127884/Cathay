# Airflow 筆記
# 1. 介紹
## DAG & Task
*一個 DAG 是由多個 Tasks 組成，每個 Task 是分開執行的，Task 是 Airflow 執行基礎單位。*
## DAG
在 Graph View 頁面，可以看到每個 DAG 都是由多個 Tasks 組成，所以代表我們待會在寫 DAG 時，除了宣告 DAG 物件代表不同的 DAG，同時也會宣告很多 Task 物件來組成 DAG 要執行的內容。
![Alt text](Pics/image.png)
## Task
Task 的狀態有很多種，每個 Task 都是要先經過 queued 的狀態，才會進到 Running 的狀態，而每個 Task 之間又存在著相依的關係，像是上圖中的 run_after_loop 要等 runme_0, runme_1 及 runme_2 都成功執行後，才會開始執行，這代表每一個 Task 都是分開執行的，因為每個 Task 都是分開進入 queue 的。

![Alt text](Pics/image-1.png)
## SubDAG
如果有一組不斷重複的 Tasks 出現在 DAG 中，像是下面的 section tasks 1~5 重複了兩次，透過SubDAG就能達到模組化。   
![Alt text](Pics/image-2.png)
下面這張圖就是將 section tasks 包裝成 SubDAG。  
![Alt text](Pics/image-3.png)

---
# 2. 實作DAG
## 1) Default Arguments
```python
default_args = {
    'owner': 'someone', # DAG 擁有者的名稱
    'depends_on_past': False, # 每一次執行的 Task 是否會依賴於上次執行的 Task，如果是 False 的話，代表上次的 Task 如果執行失敗，這次的 Task 就不會繼續執行
    'start_date': datetime(2023, 11, 1), # Task 從哪個日期後開始可以被 Scheduler 排入排程(測試時通常設為昨天日期)
    'email': ['sean.hsu@cathaysec.com.tw'], # 如果 Task 執行失敗的話，要寄信給哪些人的 email
    'email_on_failure': False, # 如果 Task 執行失敗的話，是否寄信
    'email_on_retry': False, # 如果 Task 重試的話，是否寄信
    'retries': 1, # 最多重試的次數
    'retry_delay': timedelta(minutes=5), # 每次重試中間的間隔
    # 'end_date': datetime(2020, 2, 29), # Task 從哪個日期後，開始不被 Scheduler 放入排程
    # 'execution_timeout': timedelta(seconds=300), # Task 執行時間的上限
    # 'on_failure_callback': some_function, # Task 執行失敗時，呼叫的 function
    # 'on_success_callback': some_other_function, # Task 執行成功時，呼叫的 function
    # 'on_retry_callback': another_function, # Task 重試時，呼叫的 function
}
```
## 2) DAG Object
```python
dag = DAG(
    dag_id='my_dag',
    description='my dag',
    default_args=default_args,
    schedule_interval='@Once' # 設定DAG多久執行一次, @Once為執行一次
)
```
## 3) Operator
Operator 用來定義 Task，可能是執行 Bash 或 Python。

### BashOperator
第一個 Task 是取得時間戳 , 並把結果放入 XCom , 這個 Task 透過 BashOperator 執行 Bash 完成 , 在 ```BashOperator``` 設定 xcom_push=True , 可以將 bash執行的結果放入 Xcom。
```python
get_timestamp = BashOperator(
    task_id='get_timestamp',
    bash_command='date +%s',
    do_xcom_push=True,
    dag=dag
)
```
### BranchPythonOperator
第二個 Task 是從 XCom 取得時間戳 , 並判斷要執行寫入 , 還是什麼都不做 .
這邊要判斷時間戳是否為偶數 , 如果是偶數 , 則回傳 ```isEven``` 反之則執行 ```skip```。
```python
branching = BranchPythonOperator(
    task_id='branching',
    python_callable=lambda **context: 'isEven' if int(context['task_instance'].xcom_pull(task_ids='get_timestamp')) % 2 == 0 else 'skip',
    provide_context=True,
    dag=dag,
)
```
如果是要做判斷後選擇下一個要執行的 Task , 則選擇使用 ```BranchPythonOperator``` , 反之 , 使用 ```PythonOperator``` 即可。

### EmailOperator
寄信通知
```python
isEven = EmailOperator(
    task_id = 'isEven', 
    to = ['sean.hsu@cathaysec.com.tw'], # 寄給誰
    subject = "test_Continue", # 標題
    html_content = "test_Continue", # 內文
    dag = dag
)
```
### DummyOperator
什麼都不做的情況 , Airflow 提供了 ```DummyOperator``` , 讓我們可以將什麼事情都不做的情況 , 也設置成一個 Operator。
```python
skip = DummyOperator(
    task_id='skip',
    dag=dag
)
```
## 4) 串接 Task
```python
get_timestamp >> branching >> [isEven, skip]
```
## 5) Graph
![Alt text](Pics/arflow_graph-1.png)
## 6) 完整程式碼
```python
from airflow import DAG
from airflow.operators.python import PythonOperator, BranchPythonOperator
from airflow.operators.empty import EmptyOperator
from airflow.operators.email import EmailOperator
from airflow.operators.bash_operator import BashOperator
from airflow.utils.trigger_rule import TriggerRule
from datetime import datetime
from airflow.operators.dummy_operator import DummyOperator

default_args = {
    'owner': 'Sean',
    'start_date': datetime(2023,11,1),
    'schedule_interval': '@once',
    'retries': 2 #最多重試的次數
}

dag = DAG(
    dag_id = 'SeanTest3',
    default_args = default_args
)

get_timestamp = BashOperator(
    task_id='get_timestamp',
    bash_command='date +%s',
    do_xcom_push=True,
    dag=dag
)

branching = BranchPythonOperator(
    task_id='branching',
    python_callable=lambda **context: 'isEven' if int(context['task_instance'].xcom_pull(task_ids='get_timestamp')) % 2 == 0 else 'skip',
    provide_context=True,
    dag=dag
)

isEven = EmailOperator(
    task_id = 'isEven',
    to = ['sean.hsu@cathaysec.com.tw'],
    subject = "test_Continue",
    html_content = "test_Continue", 
    dag = dag
)

skip = DummyOperator(
    task_id='skip',
    dag=dag
)

get_timestamp >> branching >> [isEven, skip]
```
## 7) Trigger Rules
Airflow 可以幫每個 Task 設定 Trigger Rule，讓 Scheduler 判斷要不要將某個 Task 放入排程，所以我們就不需要在 Task 裡，再自己用 if-else 判斷了。在每個 Task 裡，我們都可以透過 ```trigger_rule``` 表示我們想設置的 Trigger Rule。
+ ```all_success``` : 預設的 Trigger Rule，代表某個 Task 的上游 Tasks 的狀態都要是成功，才會執行這個 Task。以第一張圖 run_after_loop 為例，代表 runeme_0, runme_1 及 runme_2 都要成功，才會執行。


+ ```all_failed``` : 與 ```all_success``` 相反，表示上游 Tasks 的狀態都是失敗時執行，這可以用於處理 exception 狀態。


+ ```all_done``` : 代表只要上游 Tasks 完成，不管它們的狀態是成功、失敗或是 ```skipped```，都會執行。


+ ```one_failed``` : 上游 Tasks 其中一個失敗就執行，這個 Trigger Rule 不會等上游 Tasks 都完成才執行，而是只要有失敗就立即執行。


+ ```one_success``` :  與 ```one_failed``` 相反，上游 Tasks 其中一個成功就立即執行。


+ ```none_failed``` : ```none_failed``` 與 ```all_success``` 的差異是，```all_success``` 要上游所有 Tasks 都成功，```none_failed``` 則是上游 Tasks 的狀態都是成功或是 skipped。


+ ```none_skipped``` : 上游 Tasks 的狀態是成功或是失敗時執行。
