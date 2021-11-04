from airflow.hooks.postgres_hook import PostgresHook
from airflow.models import BaseOperator
from airflow.utils.decorators import apply_defaults

class DataQualityOperator(BaseOperator):

    ui_color = '#89DA59'
    sql_check = """
                select count({key_field}) as nb_null
                from {table}
                where {key_field} is null
                """
    nb_rows = """
                select count({key_field}) as nb_rows
                from {table}
                """
    
    @apply_defaults
    def __init__(self,
                 redshift_conn_id="",
                 key_field="",
                 table="",
                 *args, **kwargs):

        super(DataQualityOperator, self).__init__(*args, **kwargs)
        # Map params here
        # Example:
        # self.conn_id = conn_id
        self.redshif_conn_id = redshift_conn_id
        self.table=table
        self.key_field=key_field


      
    def check_staging(*args, **kwargs):
        table = kwargs["params"]["table"]
        redshift_hook = PostgresHook("redshift")
        records = redshift_hook.get_records(sql_check)
        nb_rows = redshift_hook.get_records(nb_rows)
        if len(records) >0 or len(records[0]) >0:
            raise ValueError(f"Data quality check failed. {table}.{key_field} contains nulls")
        else:
            logging.info(f"Data quality on table {table} check passed with {nb_rows[0]} records")
        
     
    def execute(self, context):
        self.log.info('DataQualityOperator not implemented yet')
        redshift_hook=PostgresHook(postgres_conn_id=self.redshift_conn_id)
        self.log.info("Quality checks are applied to Staging Tables")
        sql_check1=DataQualityOperator.check_staging.format(staging_events, song)
        sql_check2=DataQualityOperator.check_staging.format(staging_events, artist)
        sql_check3=DataQualityOperator.check_staging.format(staging_events, length)
        sql_check4=DataQualityOperator.check_staging.format(staging_songs, title)
        sql_check5=DataQualityOperator.check_staging.format(staging_songs, artist_name)
        sql_check6=DataQualityOperator.check_staging.format(staging_songs, duration)
 
        redshift.run(sql_check1)
        redshift.run(sql_check2)
        redshift.run(sql_check3)
        redshift.run(sql_check4)
        redshift.run(sql_check5)
        redshift.run(sql_check6)