## Apache Airflow Practice (Hands-On)

Apache Airflow is used to **schedule, orchestrate, and monitor workflows** using DAGs (Directed Acyclic Graphs). Below are **basic practice exercises with examples** you can try locally or in a lab.

---

# 1️⃣ Install Apache Airflow (Practice Setup)

### Linux / Ubuntu / WSL / VM

```bash
sudo apt update -y
sudo apt install python3-pip -y

pip install apache-airflow
```

Initialize database

```bash
airflow db init
```

Create admin user

```bash
airflow users create \
--username admin \
--firstname atul \
--lastname kamble \
--role Admin \
--email admin@example.com
```

Start Airflow

```bash
airflow webserver --port 8080
```

Open browser

```
http://localhost:8080
```

Start scheduler

```bash
airflow scheduler
```

---

# 2️⃣ Airflow Folder Structure

```
airflow/
 ├── dags/
 │    └── example_dag.py
 ├── logs/
 ├── plugins/
 └── airflow.cfg
```

Most important folder:

```
dags/
```

All workflow code goes here.

---

# 3️⃣ Practice DAG 1 — Hello World

Create file

```
dags/hello_airflow.py
```

Code:

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime

def hello():
    print("Hello from Apache Airflow!")

with DAG(
    dag_id="hello_airflow",
    start_date=datetime(2024,1,1),
    schedule_interval="@daily",
    catchup=False
) as dag:

    task1 = PythonOperator(
        task_id="hello_task",
        python_callable=hello
    )
```

Run DAG from UI.

---

# 4️⃣ Practice DAG 2 — Task Dependency

Example workflow

```
Task1 → Task2 → Task3
```

Code

```python
from airflow import DAG
from airflow.operators.bash import BashOperator
from datetime import datetime

with DAG(
    dag_id="dependency_example",
    start_date=datetime(2024,1,1),
    schedule_interval="@daily",
    catchup=False
) as dag:

    task1 = BashOperator(
        task_id="print_date",
        bash_command="date"
    )

    task2 = BashOperator(
        task_id="sleep_task",
        bash_command="sleep 5"
    )

    task3 = BashOperator(
        task_id="final_task",
        bash_command="echo Workflow Complete"
    )

task1 >> task2 >> task3
```

---

# 5️⃣ Practice DAG 3 — ETL Pipeline

Simulating **Extract → Transform → Load**

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime

def extract():
    print("Extracting data")

def transform():
    print("Transforming data")

def load():
    print("Loading data")

with DAG(
    dag_id="etl_pipeline",
    start_date=datetime(2024,1,1),
    schedule_interval="@daily",
    catchup=False
) as dag:

    t1 = PythonOperator(
        task_id="extract_data",
        python_callable=extract
    )

    t2 = PythonOperator(
        task_id="transform_data",
        python_callable=transform
    )

    t3 = PythonOperator(
        task_id="load_data",
        python_callable=load
    )

t1 >> t2 >> t3
```

---

# 6️⃣ Practice DAG 4 — Run Shell Commands

```python
from airflow import DAG
from airflow.operators.bash import BashOperator
from datetime import datetime

with DAG(
    dag_id="system_monitoring",
    start_date=datetime(2024,1,1),
    schedule_interval="@hourly",
    catchup=False
) as dag:

    cpu = BashOperator(
        task_id="check_cpu",
        bash_command="top -b -n1 | head -n5"
    )

    disk = BashOperator(
        task_id="check_disk",
        bash_command="df -h"
    )

cpu >> disk
```

---

# 7️⃣ Practice DAG 5 — Run Python Script

Example:

```python
python script.py
```

Airflow DAG

```python
run_script = BashOperator(
    task_id="run_python_script",
    bash_command="python3 /home/ubuntu/script.py"
)
```

---

# 8️⃣ Practice Project (Mini Project)

### Use Case

Automate **website monitoring**

Workflow

```
Check Website → Save Log → Send Alert
```

DAG Example

```python
import requests
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime

def check_website():
    url = "https://google.com"
    response = requests.get(url)

    if response.status_code == 200:
        print("Website is UP")
    else:
        print("Website DOWN")

with DAG(
    dag_id="website_monitor",
    start_date=datetime(2024,1,1),
    schedule_interval="*/10 * * * *",
    catchup=False
) as dag:

    monitor = PythonOperator(
        task_id="check_site",
        python_callable=check_website
    )
```

---

# 9️⃣ Important Airflow Commands

| Command                     | Purpose         |
| --------------------------- | --------------- |
| airflow db init             | Initialize DB   |
| airflow webserver           | Start UI        |
| airflow scheduler           | Start scheduler |
| airflow dags list           | List DAGs       |
| airflow tasks list dag_id   | List tasks      |
| airflow dags trigger dag_id | Run DAG         |

---

# 🔟 Real-World Airflow Use Cases

| Use Case         | Example                  |
| ---------------- | ------------------------ |
| Data pipelines   | ETL pipelines            |
| ML workflows     | Model training pipelines |
| Cloud automation | Backup jobs              |
| Monitoring       | Website checks           |
| Batch processing | Data transformation      |

---

# 🔥 Practice Tasks (Recommended)

Try these exercises:

1️⃣ Create DAG that runs **every 5 minutes**

2️⃣ Create **3 tasks ETL pipeline**

3️⃣ Run **Linux command inside DAG**

4️⃣ Download file from internet using DAG

5️⃣ Trigger **AWS CLI command using Airflow**

---
