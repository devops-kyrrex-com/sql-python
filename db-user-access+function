import psycopg2
from psycopg2 import errors

# Функція python для встановлення підключення до бази postgresql
# A python function to establish a connection to the postgresql database
def connect_to_database(db_params):
    conn = psycopg2.connect(**db_params)
    cur = conn.cursor()
    return conn, cur

# Функція python для закриття бази данних
# Python function to close the database
def close_database_connection(conn, cur):
    cur.close()
    conn.close()

# Функція python для встановлення event trigger та function в базі данних
# Python function to set event trigger and function in database
def grant_read_access_to_public_schema(db_params, user):
    grant_query = "GRANT SELECT ON ALL TABLES IN SCHEMA public TO {0};".format(user)
    event_query = """CREATE OR REPLACE FUNCTION grant_read_access()
                      RETURNS event_trigger AS $$
                      DECLARE
                        obj record;
                      BEGIN
                        FOR obj IN SELECT * FROM pg_event_trigger_ddl_commands() WHERE command_tag IN ('CREATE TABLE')
                        LOOP
                          EXECUTE format('GRANT SELECT ON TABLE %s TO {0}', obj.object_identity);
                        END LOOP;
                      END;
                      $$ LANGUAGE plpgsql;
                      """.format(user)

    event_trigger_query = """CREATE EVENT TRIGGER grant_read_access_trigger ON ddl_command_end
                             WHEN TAG IN ('CREATE TABLE')
                             EXECUTE FUNCTION grant_read_access();
                             """

    conn, cur = connect_to_database(db_params)

    try:
        cur.execute(grant_query.format(user))
        cur.execute(event_query)
        cur.execute(event_trigger_query)
        conn.commit()
    except errors.DuplicateObject:
        # Якщо тригер вже існує, просто ігноруємо помилку
        print("Event trigger already exists")
        print("")
        conn.rollback()

    close_database_connection(conn, cur)

# Параметри підключення до PostgreSQL
# PostgreSQL connection options
db_params = {
    'host': 'host_name',
    'port': 'port_number',
    'user': 'user_name',
    'password': 'password',
}

user = 'name_user_who_need_access'

# Встановлення з'єднання з PostgreSQL та отримання списку баз даних
# Connecting to PostgreSQL and getting a list of databases
conn, cur = connect_to_database(db_params)
cur.execute("SELECT datname FROM pg_database WHERE datistemplate = false")
databases = cur.fetchall()
close_database_connection(conn, cur)

# Надання дозволів на читання таблиць у кожній базі даних
# Granting read permissions to tables in each database
for database in databases:
    db_name = database[0]
    print("DB name >" + db_name)

    db_params2 = db_params.copy()
    db_params2['database'] = db_name

    grant_read_access_to_public_schema(db_params2, user)
