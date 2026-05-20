# 🚖 Прогнозирование оттока клиентов Яндекс Такси

> **Выпускной проект** — Яндекс Практикум (Мастерская аналитики данных)  
> **Роль:** Аналитик данных / Data Engineer  
> **Технологии:** PySpark, Airflow, ClickHouse, S3, Random Forest

![Python](https://img.shields.io/badge/python-3.9+-blue?logo=python)
![PySpark](https://img.shields.io/badge/PySpark-3.3+-orange?logo=apachespark)
![Airflow](https://img.shields.io/badge/Airflow-2.5+-green?logo=apacheairflow)
![ClickHouse](https://img.shields.io/badge/ClickHouse-23.x-yellow?logo=clickhouse)
![Accuracy](https://img.shields.io/badge/Accuracy-86.2%25-brightgreen)

---

## 📌 О проекте

Компания **Яндекс Такси** ежедневно обрабатывает миллионы поездок. Перед командой стояли две задачи:

1. **Автоматизировать подготовку финансовой витрины** для аналитиков и продуктологов  
2. **Научиться прогнозировать отток клиентов**, чтобы удерживать их до того, как они уйдут

### Результаты проекта

| Команда | Что получила |
|---------|---------------|
| 💰 Финансы | Ежедневная сводка по выручке в разрезе типов оплаты |
| 🧠 Продукт | Модель с точностью **86.2%**, предсказывающая отток |
| 👨‍🔧 Водители | Аналитика чаевых по способам оплаты |

**Технический стек:**  
`S3 (Parquet)` → `PySpark (ETL + ML)` → `ClickHouse`  
`Airflow` → оркестрация всего пайплайна

---

## 🧠 Что внутри

### 1. ETL-пайплайн (PySpark)

- Чтение сырых данных из S3 (Parquet, миллионы строк)
- Группировка по `payment_type` (card / cash / corporate)
- Расчёт метрик: количество поездок, средний чек, средние чаевые, выручка
- Запись витрины в ClickHouse

### 2. Модель прогнозирования оттока

**Определение оттока:**  
Пользователь ушёл, если его последняя поездка была более 30 дней назад.

**Признаки:**

| Признак | Описание |
|---------|----------|
| `trip_count_30d` | Количество поездок за последние 30 дней |
| `avg_fare` | Средняя стоимость поездки |
| `avg_tips` | Средние чаевые |
| `avg_trip_seconds` | Средняя длительность поездки |
| `main_payment_type` | Самый частый способ оплаты |

**Модель:** Random Forest (100 деревьев, глубина 10)  
**Точность на тесте:** **86.2%**  
**ROC-AUC:** 0.91

### 3. Оркестрация в Airflow

**DAG `taxi_data_spark_pipeline`:**

1. **S3KeySensor** — ожидает появление нового файла `taxi_data.parquet` в S3
2. **DataprocCreatePysparkJobOperator** — запускает PySpark-скрипт на кластере Yandex Dataproc
3. Результат: обновлённая витрина в ClickHouse + сохранённая модель в S3

---

## 📊 Пример результатов

### Витрина `taxi_payment_summary`

| payment_type | trip_count | avg_fare | avg_tips | total_revenue |
|--------------|------------|----------|----------|---------------|
| card         | 125,432    | 450.50   | 45.20    | 56,521,890    |
| cash         | 87,234     | 430.20   | 0.00     | 37,523,400    |
| corporate    | 12,456     | 780.30   | 89.40    | 9,812,567     |

### Факторы оттока (важность признаков)

| Признак | Важность |
|---------|----------|
| trip_count_30d | 48% |
| avg_tips | 23% |
| avg_fare | 15% |
| main_payment_type | 9% |
| avg_trip_seconds | 5% |

---

**Компоненты:**

- **S3** — хранилище исходных данных (Parquet) и обученной модели
- **Airflow** — оркестрация: сенсор + запуск Spark-задачи
- **Yandex Dataproc** — вычислительный кластер для PySpark
- **ClickHouse** — целевая витрина с агрегированными метриками

---


## Как запустить

### 1. Клонировать

`git clone https://github.com/user/taxi-churn.git`

`cd taxi-churn`

### 2. Установить зависимости

`pip install -r requirements.txt`

### 3. Настроить .env

`cp .env.example .env`

Отредактировать .env (ClickHouse, S3, Dataproc)

### 4. Запустить ETL

`python src/spark_aggregation.py`

### 5. Или открыть ноутбук

`jupyter notebook notebooks/Project-1.ipynb`

---

## Структура

taxi-churn-prediction/
├── README.md
├── requirements.txt
├── .env.example
├── notebooks/
│   └── Project-1.ipynb
├── src/
│   ├── spark_aggregation.py
│   ├── train_churn_model.py
│   └── utils.py
├── dags/
│   └── taxi_spark_dag.py
├── sql/
│   └── create_table.sql
└── tests/
    ├── test_agg.py
    └── test_model.py

---

## Автор

**Иван Черкашин**

GitHub: [@your-github](https://github.com/your-github)

Telegram: [@your-telegram](https://t.me/your-telegram)

---

## Лицензия

MIT
