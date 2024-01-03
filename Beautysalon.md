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
| date_updated      | date        | NOT NULL     | Дата изменения записи				  |
| comment           | varchar(255)| NOT NULL     | Комментарий						  |

**Первичный ключ:**<br>
    appointment_id<br>
**Индексы:**<br>
    appointment_id, date<br>
**Ссылки извне:**<br>
    FOREIGN KEY (client_id) REFERENCES client(client_id)<br>
    FOREIGN KEY (updater_person_id) REFERENCES person(person_id)<br>


#### <a id="title2.2">Таблица appointment_service</a>
**Описание:** Услуги, входящие в запись
|    Столбец        |   Тип       | Модификаторы |             Описание                                   |
|-------------------|-------------|--------------|--------------------------------------------------------|
| appointment_id    | serial      | NOT NULL     | Идентификатор записи					  |
| service_id        | int         | NOT NULL     | Идентификатор услуги					  |
| service_qty       | smallint    | NOT NULL     | Количество услуг					  |
| start_time        | time        | NOT NULL     | Время начала услуги					  |
| employee_id       | int         | NOT NULL     | Работник, осуществляющий услугу			  |
| booked_provided   | smallint    | NOT NULL     | Статус записи: 1 - booked, 2 - provided		  |

**Первичный ключ:**<br>
(appointment_id, employee_id, start_time, booked_provided)<br>
**Индексы:**<br>
    appointment_id, service_id, employee_id<br>
**Ссылки извне:**<br>
    FOREIGN KEY (appointment_id) REFERENCES appointment(appointment_id)<br>
    FOREIGN KEY (service_id) REFERENCES service(service_id)<br>
    FOREIGN KEY (employee_id ) REFERENCES employee(employee_id)<br>


#### <a id="title2.3">Таблица person</a>
**Описание:** Данные человека

|    Столбец        |   Тип       | Модификаторы |             Описание                                   |
|-------------------|-------------|--------------|--------------------------------------------------------|
| person_id         | serial      | NOT NULL     | Идентификатор человека                                 |
| first_name        | varchar(50) | NOT NULL     | Имя                                                    |
| last_name         | varchar(50) | NOT NULL     | Фамилия						  |
| email             | varchar(50) | NOT NULL     | Адрес электронной почты				  |
| phone_number      | varchar(25) | NOT NULL     | Номер телефона					  |

**Первичный ключ:**<br>
    person_id<br>
**Индексы:**<br>
    person_id<br>


#### <a id="title2.4">Таблица client</a>
**Описание:** Определяет человека в качестве клиента
|    Столбец        |   Тип       | Модификаторы |             Описание					  |
|-------------------|-------------|--------------|--------------------------------------------------------|
| client_id         | serial      | NOT NULL     | Идентификатор клиента				  |
| person_id         | int         | NOT NULL     | Идентификатор персоны				  |

**Первичный ключ:**<br>
    client_id<br>
**Индексы:**<br>
    client_id<br>
**Ссылки извне:**<br>
    FOREIGN KEY (person_id) REFERENCES person(person_id)<br>


#### <a id="title2.5">Таблица employee</a>
**Описание:** Определяет человека в качестве работника. Работник с end_date NULL - действующий.
|    Столбец        |   Тип       | Модификаторы |             Описание					  |
|-------------------|-------------|--------------|--------------------------------------------------------|
| employee_id       | serial      | NOT NULL     | Идентификатор работника				  |
| person_id         | int         | NOT NULL     | Идентификатор персоны				  |
| salary            | money       | NOT NULL     | Тариф заработной платы за 1 час			  |
| position          | varchar(50) | NOT NULL     | Наименование должности				  |
| start_date        | date        | NOT NULL     | Дата принятия на работу				  |
| end_date          | date        |              | Дата увольнения					  |

**Первичный ключ:**<br>
    employee_id<br>
**Индексы:**<br>
    employee_id<br>
**Ограничения-проверки:**<br>
    CHECK (end_date >= start_date or end_date is NULL)<br>
**Ссылки извне:**<br>
    FOREIGN KEY (person_id) REFERENCES person(person_id)<br>


#### <a id="title2.6">Таблица employee_week_schedule</a>
**Описание:** Расписание временных интервалов работы персонала по дням недели.
|    Столбец        |   Тип       | Модификаторы |             Описание					  |
|-------------------|-------------|--------------|--------------------------------------------------------|
| schedule_id       | serial      | NOT NULL     | Идентификатор расписания				  |
| employee_id       | int         | NOT NULL     | Идентификатор работника				  |
| day_of_week       | smallint    | NOT NULL     | День недели (1 - понедельник)			  |
| start_time        | time        | NOT NULL     | Начало временного интервала				  |
| end_time          | time        | NOT NULL     | Окончание временного интервала			  |

**Первичный ключ:**<br>
    schedule_id<br>
**Индексы:**<br>
    schedule_id , employee_id<br>
**Ограничения-проверки:**<br>
    CHECK (end_time >= start_time)<br>
**Ссылки извне:**<br>
    FOREIGN KEY (employee_id) REFERENCES employee (employee_id)<br>


#### <a id="title2.7">Таблица service</a>
**Описание:** Предоставляемые услуги 
|    Столбец        |   Тип       | Модификаторы |             Описание					  |
|-------------------|-------------|--------------|--------------------------------------------------------|
| service_id        | serial      | NOT NULL     | Идентификатор услуги					  |
| service_name      | varchar(50) | NOT NULL     | Наименование услуги					  |
| price             | money       | NOT NULL     | Стоимость услуги					  |
| duration          | time        | NOT NULL     | Продолжительность услуги				  |

**Первичный ключ:**<br>
    service_id<br>
**Индексы:**<br>
    service_id, price<br>


#### <a id="title2.8">Таблица employee_service</a>
**Описание:** Работник осуществляет одну и более услуг.
|    Столбец        |   Тип       | Модификаторы |             Описание					  |
|-------------------|-------------|--------------|--------------------------------------------------------|
| service_id        | int         | NOT NULL     | Идентификатор услуги					  |
| employee_id       | int         | NOT NULL     | Идентификатор работника				  |

**Первичный ключ:**<br>
    (service_id, employee_id)<br>
**Индексы:**<br>
    service_id<br>
**Ссылки извне:**<br>
    FOREIGN KEY (service_id) REFERENCES service(service_id)<br>
    FOREIGN KEY (employee_id) REFERENCES employee(employee_id)<br>


#### <a id="title2.9">Таблица category</a>
**Описание:** Категории услуг.
|    Столбец        |   Тип       | Модификаторы |             Описание					  |
|-------------------|-------------|--------------|--------------------------------------------------------|
| category_id       | serial      | NOT NULL     | Идентификатор категории				  |
| category_name     | varchar(50) | NOT NULL     | Наименование категории				  |
| comment           | varchar(255)| NOT NULL     | Комментарий к категории				  |

**Первичный ключ:**<br>
    category_id<br>
**Индексы:**<br>
    category_id<br>


#### <a id="title2.10">Таблица service_category</a>
**Описание:** Категория включает в себя одну и более услуг.
|    Столбец        |   Тип       | Модификаторы |             Описание					  |
|-------------------|-------------|--------------|--------------------------------------------------------|
| service_id        | int         | NOT NULL     | Идентификатор услуги					  |
| category_id       | int         | NOT NULL     | Идентификатор категории				  |

**Первичный ключ:**<br>
    (service_id, category_id)<br>
**Индексы:**<br>
    category_id<br>
**Ссылки извне:**<br>
    FOREIGN KEY (service_id) REFERENCES service(service_id)<br>
    FOREIGN KEY (category_id) REFERENCES category(category_id)<br>


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
**Ссылки извне:**<br>
    FOREIGN KEY (category_id) REFERENCES category(category_id)<br>


### <a id="title2">3. Примеры бизнес-задач</a>

   - Оформление и расписание записей.
   - Расчет оплаты сотрудников.
   - Расчет дохода и расхода в разрезе сотрудников за месяц.
