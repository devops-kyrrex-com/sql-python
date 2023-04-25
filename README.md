# sql-python
this repositories show how create function and event trigger for allow read access permit for user 

Спочатку, код імпортує модулі psycopg2 та errors з бібліотеки psycopg2. Потім, він визначає три функції:
1. connect_to_database: функція, яка встановлює з'єднання з базою даних та повертає з'єднання та курсор.
2. close_database_connection: функція, яка закриває з'єднання з базою даних.
3. grant_read_access_to_public_schema: головна функція скрипту, яка надає дозвіл на читання таблиць в базі даних для вказаного користувача.

Далі, визначаються параметри підключення до бази даних PostgreSQL у змінній db_params та ім'я користувача, для якого потрібно надати дозвіл на читання таблиць, у змінній user.
Скрипт потім встановлює з'єднання з базою даних та отримує список баз даних. Для кожної бази даних у списку він викликає функцію grant_read_access_to_public_schema, яка надає дозвіл на читання таблиць в базі даних для вказаного користувача.
В функції grant_read_access_to_public_schema спочатку формується запит для надання дозволу на читання таблиць у схемі public для вказаного користувача. Потім він формує запит для створення event trigger та функції, яка буде виконуватися під час тригера. Після того, як обидва запити виконуються успішно, скрипт фіксує зміни у базі даних та закриває з'єднання.
Якщо тригер вже існує, скрипт ігнорує помилку та переходить до наступної бази даних.

event_query: 
Цей код створює функцію grant_read_access(), яка повертає event_trigger та містить в собі цикл FOR, що ітерується через об'єкти таблиць в базі даних, де тригер було встановлено. Об'єкти таблиць отримуються за допомогою запиту SELECT * FROM pg_event_trigger_ddl_commands() WHERE command_tag IN ('CREATE TABLE').
Кожен об'єкт таблиці обробляється функцією format, яка використовує значення obj.object_identity для створення запиту на GRANT SELECT на відповідну таблицю. Запит виконується за допомогою EXECUTE.
Отже, ця функція призначена для створення тригера, який автоматично додає дозвіл на читання (GRANT SELECT) до кожної таблиці, яка створена в базі даних, де тригер було встановлено.

event_trigger_query:
Цей код створює подійний тригер grant_read_access_trigger, який виконується при кінці виконання команди визначеного типу (ddl_command_end) та коли тег команди збігається з одним з переліку ('CREATE TABLE').
Коли тригер спрацьовує, він виконує функцію grant_read_access(), що була описана раніше, яка автоматично додає дозвіл на читання (GRANT SELECT) до кожної таблиці, яка створена в базі даних, де тригер було встановлено.
Отже, цей тригер призначений для автоматичного надання дозволів на читання для новостворених таблиць в базі даних, де він був встановлений, і збільшення безпеки бази даних, забезпечуючи тільки обмежені дозволи на доступ до даних.
