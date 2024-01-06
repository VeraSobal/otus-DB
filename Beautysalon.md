## <h>База данных "Салон красоты"</h>

### <h>Содержание</h>
1. [Диаграмма схемы данных](#title1)<br>
2. [Описание таблиц и полей](#title2)<br>
- [Таблица appointment](#title2.1)<br>
- [Таблица appointment_service](#title2.2)<br>
- [Таблица person](#title2.3)<br>
- [Таблица client](#title2.4)<br>
- [Таблица employee](#title2.5)<br>
- [Таблица employee_week_schedule](#title2.6)<br>
- [Таблица service](#title2.7)<br>
- [Таблица employee_service](#title2.8)<br>
- [Таблица category](#title2.9)<br>
- [Таблица service_category](#title2.10)<br>
- [Таблица discount](#title2.11)<br>
3. [Примеры бизнес-задач](#title3)<br>
   
### <a id="title1">1. Диаграмма схемы данных</a>

<image
  src="/schema.png"
  alt="Диаграмма схемы"
  caption="">


### <a id="title2">2. Описание таблиц и полей</a>

#### <a id="title2.1">Таблица appointment</a>
**Описание:** Запись в салон

|    Столбец        |   Тип       | Модификаторы |             Описание                                   | 
|-------------------|-------------|--------------|--------------------------------------------------------|
| appointment_id    | serial      | NOT NULL     | Идентификатор записи					  | 
| client_id         | int         | NOT NULL     | Идентификатор клиента				  | 
| date              | date        | NOT NULL     | Дата визита						  | 
| date_created      | date        | NOT NULL     | Дата создания записи					 `| 
| updater_person_id | int         | NOT NULL     | Идентификатор создателя записи			  | 
| status            | smallint    | NOT NULL     | Статус записи: 1 - active, 2 - provided, 3 - cancelled | 
| date_updated      | date        |              | Дата изменения записи				  | 
| comment           | varchar(255)| NOT NULL     | Комментарий						  |

**Первичный ключ:**<br>
    appointment_id<br>
**Индексы:**<br>
    PRIMARY KEY(appointment_id), btree(date, client_id)<br>
**Ограничения-проверки:**<br>
    CHECK (date_updated >= date_created or date_updated is NULL)<br>
    CHECK (date >= date_created)<br>
**Ссылки извне:**<br>
    FOREIGN KEY (client_id) REFERENCES client(client_id)<br>
    FOREIGN KEY (updater_person_id) REFERENCES person(person_id)<br>

Предполагаются запросы по полю date, appointment_id, client_id (в порядке убывания частоты запросов). 
Поле appointment_id является первичным ключом. У полей date и client_id высокая кардинальность, более высокая - у поля date.<br>
Создаем составной индекс:<br>
CREATE INDEX idx_appointment_date_clientid ON appointment (date, client_id)<br>

#### <a id="title2.2">Таблица appointment_service</a>
**Описание:** Услуги, входящие в запись
|    Столбец        |   Тип       |   Модификаторы   |             Описание		       		  |
|-------------------|-------------|------------------|----------------------------------------------------|
| appointment_id    | int         | NOT NULL         | Идентификатор записи	    			  |
| service_id        | int         | NOT NULL         | Идентификатор услуги			          |
| service_qty       | smallint    | NOT NULL         | Количество услуг					  |
| start_time        | time        | NOT NULL         | Время начала услуги			          |
| employee_id       | int         | NOT NULL         | Работник, осуществляющий услугу			  |
| booked_provided   | smallint    | NOT NULL         | Статус записи: 1 - booked, 2 - provided		  |

**Первичный ключ:**<br>
    (appointment_id, employee_id, start_time, booked_provided)<br>
**Индексы:**<br>
    PRIMARY KEY(appointment_id, service_id, employee_id, booked_provided), btree(appointment_id)<br>
**Ограничения-проверки:**<br>
    CHECK (service_qty >= 0)<br>
    CHECK (booked_provided = 1 or booked_provided = 2)<br>
**Ссылки извне:**<br>
    FOREIGN KEY (appointment_id) REFERENCES appointment(appointment_id)<br>
    FOREIGN KEY (service_id) REFERENCES service(service_id)<br>
    FOREIGN KEY (employee_id ) REFERENCES employee(employee_id)<br>

Предполагаются запросы по полю appointment_id, employee_id, service_id  (в порядке убывания частоты запросов). 
Поля (appointment_id, employee_id, start_time, booked_provided) являются составным первичным ключом. Из полей appointment_id, employee_id, service_id высокая кардинальность у поля appointment_id.<br>
Создаем индекс:<br>
CREATE INDEX idx_appointmentservice_appointmentid ON appointment_service (appointment_id);<br>

#### <a id="title2.3">Таблица person</a>
**Описание:** Данные человека.<br>

|    Столбец        |   Тип       |   Модификаторы   |             Описание		       		  |
|-------------------|-------------|------------------|----------------------------------------------------|
| person_id         | serial      | NOT NULL         | Идентификатор человека                             |
| first_name        | varchar(50) | NOT NULL         | Имя                                                |
| last_name         | varchar(50) | NOT NULL         | Фамилия						  |
| email             | varchar(50) | NOT NULL         | Адрес электронной почты				  |
| phone_number      | varchar(25) | NOT NULL         | Номер телефона					  |

**Первичный ключ:**<br>
    person_id<br>
**Индексы:**<br>
    PRIMARY KEY(person_id)<br>
**Ограничения-проверки:**<br>
    CHECK (email  маска REGEX)<br>
    CHECK (phone_number  маска REGEX)<br>

Предполагаются запросы по полю person_id. 
Поле person_id является первичным ключом.
Создание дополнительного индекса не предполагается.<br>

#### <a id="title2.4">Таблица client</a>
**Описание:** Определяет человека в качестве клиента.<br>

|    Столбец        |   Тип       |   Модификаторы   |             Описание		       		  |
|-------------------|-------------|------------------|----------------------------------------------------|
| client_id         | serial      | NOT NULL         | Идентификатор клиента				  |
| person_id         | int         | NOT NULL UNIQUE  | Идентификатор персоны				  |

**Первичный ключ:**<br>
    client_id<br>
**Индексы:**<br>
    PRIMARY KEY(client_id), UNIQUE(person_id)<br>
**Ссылки извне:**<br>
    FOREIGN KEY (person_id) REFERENCES person(person_id)<br>

Предполагаются запросы по полю person_id, client_id. 
Поле client_id является первичным ключом, поле person_id - уникальное. 
Создание дополнительного индекса не предполагается.<br>

#### <a id="title2.5">Таблица employee</a>
**Описание:** Определяет человека в качестве работника. Работник с end_date NULL - действующий.<br>

|    Столбец        |   Тип       |   Модификаторы   |             Описание		       		  |
|-------------------|-------------|------------------|----------------------------------------------------|
| employee_id       | serial      | NOT NULL         | Идентификатор работника				  |
| person_id         | int         | NOT NULL UNIQUE  | Идентификатор персоны				  |
| salary            | money       | NOT NULL         | Тариф заработной платы за 1 час			  |
| position          | varchar(50) | NOT NULL         | Наименование должности				  |
| start_date        | date        | NOT NULL         | Дата принятия на работу				  |
| end_date          | date        |                  | Дата увольнения					  |

**Первичный ключ:**<br>
    employee_id<br>
**Индексы:**<br>
    PRIMARY KEY(employee_id), UNIQUE(person_id), btree(start_date, end_date)<br>
**Ограничения-проверки:**<br>
    CHECK (end_date >= start_date or end_date is NULL)<br>
**Ссылки извне:**<br>
    FOREIGN KEY (person_id) REFERENCES person(person_id)<br>

Предполагаются запросы по полю employee_id, start_date, end_date, person_id  (в порядке убывания частоты запросов). 
Поле employee_id является первичным ключом, поле person_id - уникальное. У полей start_date, end_date высокая кардинальность, более высокая - у поля start_date.<br>
Создаем составной индекс:<br>
CREATE INDEX idx_employee_startdate_enddate ON employee (start_date, end_date);<br>

#### <a id="title2.6">Таблица employee_week_schedule</a>
**Описание:** Расписание временных интервалов работы персонала по дням недели.<br>

|    Столбец        |   Тип       |   Модификаторы   |             Описание		       		  |
|-------------------|-------------|------------------|----------------------------------------------------|
| schedule_id       | serial      | NOT NULL         | Идентификатор расписания				  |
| employee_id       | int         | NOT NULL         | Идентификатор работника				  |
| day_of_week       | smallint    | NOT NULL         | День недели (1 - понедельник)			  |
| start_time        | time        | NOT NULL         | Начало временного интервала			  |
| end_time          | time        | NOT NULL         | Окончание временного интервала			  |

**Первичный ключ:**<br>
    schedule_id<br>
**Индексы:**<br>
    PRIMARY KEY(schedule_id), btree(start_time, end_time)<br>
**Ограничения-проверки:**<br>
    CHECK (end_time > start_time)<br>
    CHECK (day_of_week IN (1,2,3,4,5,6,7)<br>
**Ссылки извне:**<br>
    FOREIGN KEY (employee_id) REFERENCES employee (employee_id)<br>
Предполагаются запросы по полю day_of_week, start_time, end_time, employee_id (в порядке убывания частоты запросов). 
У полей start_time, end_time, employee_id средняя кардинальность, у поля day_of_week - низкая. <br>
Создаем составной индекс:<br>
CREATE INDEX idx_employeeweekschedule_starttime_endtime ON employee_week_schedule (start_time, end_time);<br>

#### <a id="title2.7">Таблица service</a>
**Описание:** Предоставляемые услуги.<br>

|    Столбец        |   Тип       |   Модификаторы   |             Описание		       		  |
|-------------------|-------------|------------------|----------------------------------------------------|
| service_id        | serial      | NOT NULL         | Идентификатор услуги				  |
| service_name      | varchar(50) | NOT NULL         | Наименование услуги				  |
| price             | money       | NOT NULL         | Стоимость услуги					  |
| duration          | time        | NOT NULL         | Продолжительность услуги				  |

**Первичный ключ:**<br>
    service_id<br>
**Индексы:**<br>
    PRIMARY KEY(service_id), b-tree(duration)<br>
**Ограничения-проверки:**<br>
    CHECK (end_time > start_time)<br>

Предполагаются запросы по полю service_id, duration (в порядке убывания частоты запросов). 
Поле service_id является первичным ключом. Поле duration имеет среднюю кардинальность.<br>
Создаем индекс:<br>
CREATE INDEX idx_service_duration ON service (duration);<br>

#### <a id="title2.8">Таблица employee_service</a>
**Описание:** Работник осуществляет одну и более услуг.
|    Столбец        |   Тип       | Модификаторы |             Описание					  |
|-------------------|-------------|--------------|--------------------------------------------------------|
| service_id        | int         | NOT NULL     | Идентификатор услуги					  |
| employee_id       | int         | NOT NULL     | Идентификатор работника				  |

**Первичный ключ:**<br>
    (service_id, employee_id)<br>
**Индексы:**<br>
    PRIMARY KEY(service_id, employee_id)<br>
**Ссылки извне:**<br>
    FOREIGN KEY (service_id) REFERENCES service(service_id)<br>
    FOREIGN KEY (employee_id) REFERENCES employee(employee_id)<br>

Предполагаются запросы по полю service_id, employee_id. 
Поля service_id, employee_id являются составным первичным ключом. 
Создание дополнительного индекса не предполагается.<br>


#### <a id="title2.9">Таблица category</a>
**Описание:** Категории услуг.
|    Столбец        |   Тип       | Модификаторы |             Описание					  |
|-------------------|-------------|--------------|--------------------------------------------------------|
| category_id       | serial      | NOT NULL     | Идентификатор категории				  |
| category_name     | varchar(50) | NOT NULL     | Наименование категории				  |
| comment           | varchar(255)| NOT NULL     | Комментарий к категории				  |

**Первичный ключ:**<br>
    category_id, <br>
**Индексы:**<br>
    PRIMARY KEY(category_id), b-tree(category_name)<br>
Предполагаются запросы по полю category_id, category_name (в порядке убывания частоты запросов).
Поле category_id является первичным ключом. Поле category_name имеет высокую кардинальность.<br>
Создаем индекс:<br>
CREATE INDEX idx_category_category_name ON category (category_name);<br>

#### <a id="title2.10">Таблица service_category</a>
**Описание:** Категория включает в себя одну и более услуг.
|    Столбец        |   Тип       | Модификаторы |             Описание					  |
|-------------------|-------------|--------------|--------------------------------------------------------|
| service_id        | int         | NOT NULL     | Идентификатор услуги					  |
| category_id       | int         | NOT NULL     | Идентификатор категории				  |

**Первичный ключ:**<br>
    (service_id, category_id)<br>
**Индексы:**<br>
    PRIMARY KEY(service_id, category_id)<br>
**Ссылки извне:**<br>
    FOREIGN KEY (service_id) REFERENCES service(service_id)<br>
    FOREIGN KEY (category_id) REFERENCES category(category_id)<br>
Предполагаются запросы по полю service_id, employee_id. 
Поля service_id, employee_id являются составным первичным ключом. 
Создание дополнительного индекса не предполагается.<br>


#### <a id="title2.11">Таблица discount</a>
**Описание:** К категориям услуг применяется скидка.
|    Столбец        |   Тип       | Модификаторы |             Описание					  |
|-------------------|-------------|--------------|--------------------------------------------------------|
| discount_id       | serial      | NOT NULL     | Идентификатор услуги					  |
| category_id       | int         | NOT NULL     | Идентификатор категории				  |
| discount_pct      | smallint    | NOT NULL     | Скидка в процентах					  |
| discount_name     | varchar(50) | NOT NULL     | Наименование скидки				 	  |
| comment           | varchar(255)| NOT NULL     | Описание скидки					  |
| start_date        | date        | NOT NULL     | Дата начала применения скидки			  |
| end_date          | date        | NOT NULL     | Дата окончания применения скидки			  |

**Первичный ключ:**<br>
    discount_id<br>
**Индексы:**<br>
    discount_id, category_id, discount_pct<br>
**Ограничения-проверки:**<br>
    CHECK (end_date >= start_date)<br>
    CHECK (discount_pct > 0 and discount_pct<=100)<br>
**Ссылки извне:**<br>
    FOREIGN KEY (category_id) REFERENCES category(category_id)<br>
Предполагаются запросы по полю start_time, end_time, category_id (в порядке убывания частоты запросов).
Поле discount_id является первичным ключом.  
У полей start_date, end_date, category_id высокая кардинальность, более высокая - у полей start_date, end_date. <br>
Создаем составной индекс:<br>
CREATE INDEX idx_discount_startdate_enddate ON discount (start_date, end_date);<br>


### <a id="title3">3. Примеры бизнес-задач</a>

   - Оформление и расписание записей.
   - Расчет оплаты сотрудников.
   - Расчет дохода и расхода в разрезе сотрудников за месяц.
