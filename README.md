# airflow-pagerduty-plugin
An Airflow plugin for triggering PagerDuty incidents.

# Usage

Move this to your plugins folder (`$AIRFLOW_HOME/plugins`) and then import it from `airflow.operators.pagerduty`.

## Failure Callback

```
from airflow.models import DAG, Variable
from airflow.operators.bash_operator import BashOperator
from airflow.operators.pagerduty import PagerDutyIncidentOperator

my_dag = DAG('tutorial')

op = BashOperator(
    dag=my_dag,
    task_id='my_task',
    provide_context=True,
    python_callable=my_python_job,
    on_failure_callback=pager_duty_incident
)
    
def pager_duty_incident(context):

    operator = PagerDutyIncidentOperator(
        task_id=str(context['task_instance_key_str']),
        title=str(context['task_instance']),
        api_key=Variable.get("pd_api_key"),
        service_id=Variable.get("pd_service_id"),
        details='Host: {}'.format(context['conf'].get('webserver', 'base_url'))
    )

    return operator.execute(context=context)
```

## Custom Operator

```
from airflow.models import DAG, Variable
from airflow.operators.python_operator import PythonOperator
from airflow.operators.pagerduty import PagerDutyIncidentOperator

class PagerDutyPythonOperator(PythonOperator):
    """
    A Python Operator that triggers PagerDuty incidents on failures.
    """

    def __init__(self, *args, **kwargs):
        super(PagerDutyPythonOperator, self).__init__(
            on_failure_callback=PagerDutyPythonOperator.on_failure_callback,
            *args,
            **kwargs)

    @staticmethod
    def on_failure_callback(context):
        operator = PagerDutyIncidentOperator(
            task_id=str(context['task_instance_key_str']),
            title=str(context['task_instance']),
            api_key=Variable.get("pd_api_key"),
            service_id=Variable.get("pd_service_id"),
            details='Host: {}'.format(context['conf'].get('webserver', 'base_url'))
        )

        return operator.execute(context=context)

my_dag = DAG('tutorial')

op = PagerDutyPythonOperator(
    dag=my_dag,
    task_id='my_task',
    provide_context=True,
    python_callable=my_python_job,
    on_failure_callback=pager_duty_incident
)
