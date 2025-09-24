# 🚀 Powerful SQL Commands for Backend/AI Engineers

<img width="800" height="800" alt="Image" src="https://github.com/user-attachments/assets/f6008016-0565-4b3e-8a32-7594b8faf137" />

## 📖 Overview
Основа основ! Для ML Platform Engineer'а знание мощных SQL команд = способность эффективно работать с feature stores, метриками моделей и большими датасетами. Поехали! 🔥

## 🛠️ Core Data Operations

### 📥 INSERT: Storing Data Like a Pro

<details>
<summary>📥 <strong>INSERT операции - от базы до продакшена</strong></summary>

```sql
-- 🎯 Basic INSERT - простая вставка одной записи
INSERT INTO users (name, email, role) 
VALUES ('John Doe', 'john@example.com', 'analyst');

-- 🚀 Bulk INSERT (для AI datasets!) - массовая загрузка данных
INSERT INTO model_metrics (model_id, accuracy, precision, recall, created_at)
VALUES 
  ('gpt-4', 0.95, 0.92, 0.88, NOW()),      -- GPT-4 метрики
  ('claude-3', 0.93, 0.90, 0.85, NOW()),   -- Claude метрики
  ('llama-2', 0.87, 0.84, 0.82, NOW());    -- LLaMA метрики
-- 💡 Bulk INSERT в 10-100 раз быстрее отдельных INSERT'ов!

-- 💪 INSERT with RETURNING (получаем ID сразу!)
INSERT INTO experiments (name, config)
VALUES ('sentiment_analysis_v2', '{"model": "bert", "epochs": 10}')
RETURNING id, created_at;
-- 🎯 Возвращает: id=123, created_at=2025-01-15 14:30:00
-- 💡 Избегаем дополнительный SELECT для получения ID!

-- 🔄 UPSERT (INSERT ON CONFLICT) - идеально для feature updates!
INSERT INTO user_features (user_id, feature_vector, last_computed)
VALUES (12345, '[0.1, 0.2, 0.8]', NOW())
ON CONFLICT (user_id)           -- если user_id уже существует
DO UPDATE SET 
  feature_vector = EXCLUDED.feature_vector,    -- обновляем вектор
  last_computed = NOW(),                       -- и время
  update_count = COALESCE(update_count, 0) + 1; -- считаем обновления
-- 🎯 Либо создает новую запись, либо обновляет существующую!
```

**🚨 Red Flags для INSERT:**
- Много отдельных INSERT вместо bulk операций
- Отсутствие транзакций для связанных данных
- Игнорирование RETURNING когда нужен ID

</details>

### 🔄 UPDATE: Smart Data Modifications

<details>
<summary>🔄 <strong>UPDATE операции - умные обновления данных</strong></summary>

```sql
-- 🎯 Basic UPDATE - простое обновление
UPDATE user_sessions 
SET last_activity = NOW()           -- обновляем время активности
WHERE session_id = 'abc123'         -- для конкретной сессии
  AND status = 'active';            -- только если сессия активна

-- 🧠 Conditional UPDATE (для AI модели states)
UPDATE ml_models 
SET 
  status = CASE                     -- условное обновление статуса
    WHEN accuracy > 0.95 THEN 'production'     -- высокая точность -> продакшен
    WHEN accuracy > 0.85 THEN 'staging'        -- средняя -> тестирование  
    WHEN accuracy > 0.70 THEN 'development'    -- низкая -> разработка
    ELSE 'failed'                              -- совсем плохо -> провал
  END,
  last_evaluated = NOW(),           -- время последней оценки
  deployment_ready = (accuracy > 0.90)  -- готовность к деплою
WHERE model_type = 'text_classifier'     -- только для текстовых классификаторов
  AND created_at >= NOW() - INTERVAL '7 days';  -- недавно созданные

-- 🔗 UPDATE with JOIN (обновляем через связанные таблицы)
UPDATE products p
SET category_popularity = c.avg_rating    -- обновляем популярность категории
FROM (
  SELECT 
    category_id,
    AVG(rating) as avg_rating,          -- средний рейтинг по категории
    COUNT(*) as review_count            -- количество отзывов
  FROM product_reviews 
  WHERE created_at >= NOW() - INTERVAL '30 days'  -- за последний месяц
  GROUP BY category_id
  HAVING COUNT(*) >= 10                 -- минимум 10 отзывов
) c
WHERE p.category_id = c.category_id;    -- связываем продукты с категориями

-- 📊 UPDATE с подзапросом для ML features
UPDATE user_profiles 
SET 
  top_categories = (                    -- топ категории пользователя
    SELECT array_agg(category ORDER BY interaction_count DESC)
    FROM user_interactions ui
    WHERE ui.user_id = user_profiles.user_id
      AND ui.created_at >= NOW() - INTERVAL '90 days'  -- за 3 месяца
    GROUP BY user_id
    LIMIT 5                             -- топ-5 категорий
  ),
  activity_score = (                    -- скор активности
    SELECT COUNT(*) * 1.0 / EXTRACT(days FROM NOW() - MIN(created_at))
    FROM user_interactions ui2
    WHERE ui2.user_id = user_profiles.user_id
  )
WHERE last_updated < NOW() - INTERVAL '24 hours';  -- обновляем раз в день
```

**🚨 Red Flags для UPDATE:**
- UPDATE без WHERE (обновит ВСЕ записи!)
- Отсутствие индексов на WHERE условиях
- UPDATE в цикле вместо batch операции

</details>

### 🗑️ DELETE: Clean Data Management

<details>
<summary>🗑️ <strong>DELETE операции - умная очистка данных</strong></summary>

```sql
-- 🎯 Basic DELETE - простое удаление старых данных
DELETE FROM system_logs 
WHERE created_at < NOW() - INTERVAL '90 days'    -- старше 90 дней
  AND log_level NOT IN ('ERROR', 'CRITICAL');    -- кроме ошибок

-- 🧹 DELETE with subquery (удаляем связанные неактивные записи)
DELETE FROM model_artifacts 
WHERE model_id IN (
  SELECT m.id FROM ml_models m
  WHERE m.status = 'deprecated'                   -- устаревшие модели
    AND m.last_used < NOW() - INTERVAL '180 days' -- не использовались полгода
    AND NOT EXISTS (                              -- нет активных предсказаний
      SELECT 1 FROM predictions p 
      WHERE p.model_id = m.id 
        AND p.created_at >= NOW() - INTERVAL '30 days'
    )
);

-- 📊 DELETE with RETURNING (логируем что удалили)
DELETE FROM failed_predictions 
WHERE confidence_score < 0.1              -- очень низкая уверенность
  AND created_at < NOW() - INTERVAL '7 days'  -- старше недели
RETURNING 
  model_id, 
  COUNT(*) as deleted_count,              -- количество удаленных
  AVG(confidence_score) as avg_confidence;  -- средняя уверенность удаленных

-- 🔄 Soft DELETE (помечаем как удаленные, не удаляем физически)
UPDATE user_accounts 
SET 
  deleted_at = NOW(),                     -- помечаем время удаления
  email = 'deleted_' || id || '@deleted.com',  -- анонимизируем email
  status = 'deleted'
WHERE user_id = $1                        -- конкретный пользователь
  AND deleted_at IS NULL;                 -- еще не удален
```

**🚨 Red Flags для DELETE:**
- DELETE без WHERE (удалит ВСЮ таблицу!)
- Каскадное удаление без проверки связей
- Отсутствие бэкапа перед массовым удалением

</details>

## 🔍 Advanced SELECT Patterns

### 🎨 Window Functions (мощь для аналитики!)

<details>
<summary>🎨 <strong>Window Functions - аналитика на стероидах</strong></summary>

```sql
-- 📈 Ranking models by performance
SELECT 
  model_name,
  accuracy,
  -- 🏆 Ранжирование моделей по точности
  ROW_NUMBER() OVER (ORDER BY accuracy DESC) as rank,
  -- 📊 Сравнение с предыдущей моделью по рангу
  LAG(accuracy) OVER (ORDER BY accuracy DESC) as prev_accuracy,
  -- 📈 Насколько лучше предыдущей модели
  accuracy - LAG(accuracy) OVER (ORDER BY accuracy DESC) as improvement,
  -- 🎯 Процентиль производительности
  PERCENT_RANK() OVER (ORDER BY accuracy) * 100 as percentile
FROM model_performance
WHERE evaluated_at >= NOW() - INTERVAL '30 days'  -- последний месяц
ORDER BY accuracy DESC;

-- 📊 Running totals для метрик (кумулятивная статистика)
SELECT 
  date_trunc('day', created_at) as day,
  COUNT(*) as daily_predictions,           -- предсказаний за день
  -- 📈 Кумулятивная сумма предсказаний
  SUM(COUNT(*)) OVER (
    ORDER BY date_trunc('day', created_at)
  ) as cumulative_predictions,
  -- 📊 Скользящее среднее за 7 дней
  AVG(COUNT(*)) OVER (
    ORDER BY date_trunc('day', created_at)
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
  ) as moving_avg_7_days,
  -- 🎯 Процентное изменение к предыдущему дню
  (COUNT(*) - LAG(COUNT(*)) OVER (ORDER BY date_trunc('day', created_at))) 
  / LAG(COUNT(*)) OVER (ORDER BY date_trunc('day', created_at))::float * 100 as daily_change_percent
FROM predictions
WHERE created_at >= NOW() - INTERVAL '60 days'
GROUP BY date_trunc('day', created_at)
ORDER BY day;

-- 🏷️ Partitioned ranking (ранжирование внутри групп)
SELECT 
  category,
  product_name,
  sales_amount,
  -- 🏆 Ранг внутри категории
  RANK() OVER (PARTITION BY category ORDER BY sales_amount DESC) as category_rank,
  -- 📊 Процент от общих продаж категории
  sales_amount / SUM(sales_amount) OVER (PARTITION BY category) * 100 as category_percent,
  -- 🎯 Топ-3 в категории?
  CASE 
    WHEN RANK() OVER (PARTITION BY category ORDER BY sales_amount DESC) <= 3 
    THEN 'Top 3' 
    ELSE 'Other' 
  END as performance_tier
FROM product_sales
WHERE sale_date >= NOW() - INTERVAL '30 days';
```

**💡 Объяснение Window Functions:**
- `OVER()` - определяет "окно" для расчета
- `PARTITION BY` - делит данные на группы
- `ORDER BY` - сортировка внутри окна
- `ROWS/RANGE` - границы окна для расчета

</details>

### 🔗 Advanced JOINs

<details>
<summary>🔗 <strong>Advanced JOIN операции - связываем данные эффективно</strong></summary>

```sql
-- 🎭 Comprehensive JOIN для полной картины данных
SELECT 
  u.user_id,
  u.email,
  u.registration_date,
  -- 📊 Данные профиля пользователя  
  p.age_group,
  p.location,
  -- 🤖 AI анализ поведения
  a.sentiment_score,
  a.engagement_level,
  a.predicted_churn_probability,
  -- 📈 Метрики активности
  s.total_sessions,
  s.avg_session_duration,
  s.last_activity,
  -- 💰 Данные подписки
  sub.plan_type,
  sub.monthly_revenue,
  sub.subscription_status
FROM users u
-- 👤 Профиль может отсутствовать у новых пользователей
LEFT JOIN user_profiles p ON u.user_id = p.user_id
-- 🤖 AI анализ только у активных пользователей  
LEFT JOIN ai_user_analysis a ON u.user_id = a.user_id 
  AND a.analysis_date >= NOW() - INTERVAL '7 days'
-- 📊 Сессии агрегируются на лету
LEFT JOIN (
  SELECT 
    user_id,
    COUNT(*) as total_sessions,
    AVG(duration_seconds) as avg_session_duration,
    MAX(created_at) as last_activity
  FROM user_sessions 
  WHERE created_at >= NOW() - INTERVAL '30 days'
  GROUP BY user_id
) s ON u.user_id = s.user_id
-- 💳 Подписка должна быть активной
INNER JOIN subscriptions sub ON u.user_id = sub.user_id 
  AND sub.status = 'active'
WHERE u.created_at >= NOW() - INTERVAL '90 days'  -- новые пользователи
  AND u.email_verified = true                     -- верифицированные
ORDER BY s.last_activity DESC NULLS LAST;

-- 🔄 Self JOIN для иерархических данных
SELECT 
  c.category_id,
  c.category_name,
  c.parent_id,
  -- 📊 Родительская категория
  parent.category_name as parent_category,
  -- 🏷️ Полный путь категории
  COALESCE(parent.category_name || ' > ', '') || c.category_name as full_path,
  -- 📈 Количество подкategорий
  (SELECT COUNT(*) FROM categories children 
   WHERE children.parent_id = c.category_id) as subcategories_count
FROM categories c
-- 🔗 Self JOIN для получения родительской категории
LEFT JOIN categories parent ON c.parent_id = parent.category_id
WHERE c.is_active = true
ORDER BY full_path;

-- 💡 LATERAL JOIN для сложной логики
SELECT 
  u.user_id,
  u.email,
  -- 🎯 Топ-3 недавние покупки для каждого пользователя
  recent_orders.product_names,
  recent_orders.total_amount,
  recent_orders.order_count
FROM users u
-- 📦 LATERAL позволяет ссылаться на u.user_id в подзапросе
LEFT JOIN LATERAL (
  SELECT 
    array_agg(p.product_name ORDER BY o.created_at DESC) as product_names,
    SUM(o.total_amount) as total_amount,
    COUNT(*) as order_count
  FROM orders o
  JOIN order_items oi ON o.order_id = oi.order_id
  JOIN products p ON oi.product_id = p.product_id
  WHERE o.user_id = u.user_id                    -- ссылка на внешнюю таблицу!
    AND o.created_at >= NOW() - INTERVAL '30 days'
  LIMIT 3
) recent_orders ON true
WHERE u.status = 'active';
```

**🚨 Red Flags для JOIN:**
- CROSS JOIN без WHERE (декартово произведение!)
- JOIN без индексов на ключах связи
- Множественные LEFT JOIN без необходимости

</details>

## 🎯 Specialized Commands for AI/Backend

### 📊 Aggregations for ML Metrics

<details>
<summary>📊 <strong>ML Aggregations - статистика для моделей</strong></summary>

```sql
-- 🧮 Model performance aggregation - полная статистика моделей
SELECT 
  model_name,
  model_version,
  -- 📊 Базовые метрики
  COUNT(*) as total_predictions,
  COUNT(*) FILTER (WHERE prediction_time < 100) as fast_predictions,  -- < 100ms
  -- 🎯 Статистика уверенности
  AVG(confidence_score) as avg_confidence,
  STDDEV(confidence_score) as confidence_std,
  MIN(confidence_score) as min_confidence,
  MAX(confidence_score) as max_confidence,
  -- 📈 Перцентили производительности
  PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY confidence_score) as median_confidence,
  PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY confidence_score) as p95_confidence,
  PERCENTILE_CONT(0.99) WITHIN GROUP (ORDER BY confidence_score) as p99_confidence,
  -- ✅ Точность модели (если есть ground truth)
  COUNT(*) FILTER (WHERE actual_result = predicted_result) / COUNT(*)::float as accuracy,
  -- ⚡ Производительность
  AVG(prediction_time_ms) as avg_prediction_time,
  PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY prediction_time_ms) as p95_prediction_time,
  -- 📅 Временные паттерны
  COUNT(*) FILTER (WHERE created_at >= NOW() - INTERVAL '1 hour') as predictions_last_hour,
  COUNT(*) FILTER (WHERE created_at >= NOW() - INTERVAL '24 hours') as predictions_last_day
FROM model_predictions
WHERE created_at >= NOW() - INTERVAL '7 days'    -- последняя неделя
GROUP BY model_name, model_version
HAVING COUNT(*) >= 100                           -- минимум 100 предсказаний
ORDER BY accuracy DESC, avg_confidence DESC;

-- 📊 User behavior aggregation для recommendation систем  
SELECT 
  user_id,
  -- 🎭 Поведенческие паттерны
  COUNT(DISTINCT session_date) as active_days,
  COUNT(*) as total_interactions,
  AVG(session_duration_minutes) as avg_session_duration,
  -- 🏷️ Предпочтения по категориям
  array_agg(DISTINCT category ORDER BY interaction_count DESC) 
    FILTER (WHERE category_rank <= 3) as top_categories,
  -- 📈 Временные паттерны
  COUNT(*) FILTER (WHERE EXTRACT(hour FROM created_at) BETWEEN 9 AND 17) as business_hours_interactions,
  COUNT(*) FILTER (WHERE EXTRACT(dow FROM created_at) IN (0,6)) as weekend_interactions,
  -- 🎯 Engagement метрики
  AVG(rating) FILTER (WHERE rating IS NOT NULL) as avg_rating,
  COUNT(*) FILTER (WHERE interaction_type = 'like') as likes_count,
  COUNT(*) FILTER (WHERE interaction_type = 'share') as shares_count,
  -- 💡 ML features для модели
  CASE 
    WHEN COUNT(*) > 1000 THEN 'power_user'
    WHEN COUNT(*) > 100 THEN 'regular_user'  
    ELSE 'casual_user'
  END as user_segment
FROM (
  -- 🔄 Подготовка данных с ранжированием категорий
  SELECT 
    ui.*,
    DATE(ui.created_at) as session_date,
    COUNT(*) OVER (PARTITION BY ui.user_id, ui.category) as interaction_count,
    ROW_NUMBER() OVER (PARTITION BY ui.user_id ORDER BY COUNT(*) DESC) as category_rank
  FROM user_interactions ui
  WHERE ui.created_at >= NOW() - INTERVAL '90 days'
) user_stats
GROUP BY user_id
ORDER BY total_interactions DESC;
```

**💡 Полезные функции для ML:**
- `FILTER (WHERE ...)` - условная агрегация
- `PERCENTILE_CONT()` - точные перцентили
- `STDDEV()` - стандартное отклонение
- `array_agg()` - агрегация в массив

</details>

### 🔍 Complex Filtering for Feature Engineering

<details>
<summary>🔍 <strong>Feature Engineering - создание признаков для ML</strong></summary>

```sql
-- 🎨 Feature selection query - комплексная подготовка признаков
WITH user_behavior AS (
  -- 📊 Агрегация поведения пользователя по категориям
  SELECT 
    user_id,
    category,
    COUNT(*) as interaction_count,           -- количество взаимодействий
    AVG(session_duration_minutes) as avg_duration,  -- средняя длительность
    COUNT(*) FILTER (WHERE rating >= 4) as positive_interactions,  -- позитивные
    MAX(created_at) as last_interaction,     -- последнее взаимодействие
    -- 📈 Временные признаки
    COUNT(*) FILTER (WHERE EXTRACT(hour FROM created_at) < 12) as morning_interactions,
    COUNT(*) FILTER (WHERE EXTRACT(dow FROM created_at) IN (0,6)) as weekend_interactions
  FROM user_interactions
  WHERE created_at >= NOW() - INTERVAL '60 days'   -- последние 2 месяца
  GROUP BY user_id, category
),
user_preferences AS (
  -- 🏷️ Обработка предпочтений пользователя
  SELECT 
    user_id,
    -- 🎯 Топ категории по убыванию активности
    array_agg(category ORDER BY interaction_count DESC) as preferred_categories,
    -- 📊 Общая статистика
    SUM(interaction_count) as total_interactions,
    AVG(avg_duration) as overall_avg_duration,
    SUM(positive_interactions) as total_positive,
    MAX(last_interaction) as most_recent_activity,
    -- 💡 Признаки разнообразия
    COUNT(DISTINCT category) as categories_explored,
    -- 🕐 Временные паттерны
    SUM(morning_interactions) / SUM(interaction_count)::float as morning_ratio,
    SUM(weekend_interactions) / SUM(interaction_count)::float as weekend_ratio
  FROM user_behavior
  GROUP BY user_id
  HAVING SUM(interaction_count) >= 10        -- минимум 10 взаимодействий
),
user_segments AS (
  -- 🎭 Сегментация пользователей
  SELECT 
    *,
    -- 🏆 Уровень активности
    CASE 
      WHEN total_interactions >= 500 THEN 'power_user'
      WHEN total_interactions >= 100 THEN 'regular_user'
      WHEN total_interactions >= 20 THEN 'casual_user'
      ELSE 'new_user'
    END as activity_segment,
    -- 🎨 Тип поведения
    CASE 
      WHEN categories_explored >= 10 THEN 'explorer'
      WHEN total_positive / total_interactions::float > 0.8 THEN 'enthusiast'
      WHEN weekend_ratio > 0.6 THEN 'weekend_warrior'
      WHEN morning_ratio > 0.6 THEN 'early_bird'
      ELSE 'standard'
    END as behavior_type,
    -- 📈 Скор вовлеченности (0-100)
    LEAST(100, (
      (total_positive / total_interactions::float * 30) +           -- позитивность 30%
      (LEAST(categories_explored, 10) * 4) +                       -- разнообразие 40%  
      (LEAST(overall_avg_duration, 60) / 60 * 20) +               -- длительность 20%
      (CASE WHEN most_recent_activity >= NOW() - INTERVAL '7 days' THEN 10 ELSE 0 END)  -- недавность 10%
    )::int) as engagement_score
  FROM user_preferences
)
-- 🎯 Финальный набор признаков для ML модели
SELECT 
  us.user_id,
  -- 📊 Базовые признаки
  us.preferred_categories[1:3] as top_3_categories,    -- топ-3 категории
  us.total_interactions,
  us.categories_explored,
  us.overall_avg_duration,
  -- 🏷️ Категориальные признаки
  us.activity_segment,
  us.behavior_type,
  -- 📈 Численные признаки
  us.engagement_score,
  us.morning_ratio,
  us.weekend_ratio,
  (us.total_positive::float / us.total_interactions) as positivity_ratio,
  -- ⏰ Временные признаки
  EXTRACT(days FROM NOW() - us.most_recent_activity) as days_since_last_activity,
  -- 🎯 Бинарные признаки (для tree-based моделей)
  (us.engagement_score >= 70) as high_engagement,
  (us.categories_explored >= 5) as diverse_interests,
  (us.total_interactions >= 100) as active_user,
  -- 📊 Интерактивные признаки
  us.total_interactions * us.engagement_score as weighted_activity_score
FROM user_segments us
WHERE us.most_recent_activity >= NOW() - INTERVAL '30 days'  -- активные за месяц
ORDER BY us.engagement_score DESC, us.total_interactions DESC;
```

**🎯 Feature Engineering Tips:**
- Создавай ratio-признаки (пропорции) для нормализации
- Используй временные окна разной длины
- Комбинируй категориальные и численные признаки
- Создавай интерактивные признаки (произведения)

</details>

## 🔧 Performance & Monitoring Commands

### 📈 Query Optimization & Diagnostics

<details>
<summary>📈 <strong>EXPLAIN ANALYZE и мониторинг производительности</strong></summary>

```sql
-- 🔍 EXPLAIN ANALYZE для диагностики векторных запросов
EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON) 
SELECT 
  product_id,
  product_name,
  embedding <-> '[0.1, 0.2, 0.3, ...]'::vector as similarity_score
FROM products 
WHERE embedding <-> '[0.1, 0.2, 0.3, ...]'::vector < 0.5    -- similarity threshold
ORDER BY embedding <-> '[0.1, 0.2, 0.3, ...]'::vector       -- order by distance
LIMIT 20;
-- 🎯 Параметры:
-- ANALYZE = реально выполняет и замеряет время ⏱️
-- BUFFERS = показывает использование памяти 💾
-- FORMAT JSON = структурированный вывод 📊

-- 📊 Index usage analysis - поиск неэффективных индексов
SELECT 
  schemaname,
  tablename,
  indexname,
  idx_scan,                                    -- количество использований индекса
  idx_tup_read,                               -- строк прочитано через индекс
  idx_tup_fetch,                              -- строк получено из таблицы
  pg_size_pretty(pg_relation_size(indexname::regclass)) as index_size,
  -- 📈 Эффективность индекса (соотношение fetch/read)
  CASE 
    WHEN idx_tup_read > 0 
    THEN round((idx_tup_fetch::numeric / idx_tup_read * 100), 2)
    ELSE 0 
  END as efficiency_percent
FROM pg_stat_user_indexes
WHERE idx_scan < 1000                         -- 🚨 подозрительно мало использований
ORDER BY pg_relation_size(indexname::regclass) DESC;

-- ⚡ Медленные запросы (требует pg_stat_statements)
SELECT 
  LEFT(query, 100) as query_preview,          -- первые 100 символов запроса
  calls,                                      -- количество выполнений
  round(mean_time::numeric, 2) as avg_time_ms, -- среднее время в мс
  round(total_time::numeric, 2) as total_time_ms, -- общее время
  round((100 * total_time / sum(total_time) OVER())::numeric, 2) as percent_total_time,
  rows as avg_rows_returned                   -- среднее количество строк
FROM pg_stat_statements 
WHERE mean_time > 50                         -- 🚨 медленнее 50ms
  AND calls > 10                             -- выполнялся минимум 10 раз
ORDER BY total_time DESC                     -- самые затратные сверху
LIMIT 20;

-- 🔒 Блокировки и конкурентный доступ
SELECT 
  bl.pid,
  a.usename,
  a.query_start,
  a.state,
  a.query as blocked_query,
  -- 🚨 Информация о блокирующем процессе
  kl.pid as blocking_pid,
  ka.usename as blocking_user,  
  ka.query as blocking_query,
  -- ⏰ Как долго заблокирован
  NOW() - a.query_start as blocked_duration
FROM pg_catalog.pg_locks bl
JOIN pg_catalog.pg_stat_activity a ON bl.pid = a.pid
JOIN pg_catalog.pg_locks kl ON bl.transactionid = kl.transactionid
JOIN pg_catalog.pg_stat_activity ka ON kl.pid = ka.pid
WHERE NOT bl.granted                         -- только заблокированные
  AND bl.pid != ka.pid                       -- исключаем self-блокировки
ORDER BY a.query_start;
```

**🚨 Red Flags в EXPLAIN результатах:**
- `Seq Scan` на больших таблицах без WHERE фильтров
- `Execution Time` > 1000ms для простых запросов  
- `Buffers: read` >> `shared hit` (данные не в кэше)
- `Nested Loop` с большим количеством строк
- Отсутствие использования индексов

</details>

## 🚀 Pro Tips для Production

### 💡 Best Practices & Performance Hacks

<details>
<summary>💡 <strong>Production-Ready SQL паттерны</strong></summary>

```sql
```sql
-- 🎯 Parameterized queries (защита от SQL injection)
SELECT * FROM users 
WHERE email = $1                    -- ✅ ПРАВИЛЬНО - параметризованный запрос
  AND status = $2;
-- ❌ НЕПРАВИЛЬНО: WHERE email = '" + user_input + "'"

-- 📊 Эффективная пагинация (избегаем OFFSET)
SELECT * FROM products
WHERE id > $1                       -- последний ID с предыдущей страницы
ORDER BY id
LIMIT 20;
-- 🚀 В разы быстрее чем OFFSET 1000000 LIMIT 20

-- 🔄 Batch operations вместо циклов
INSERT INTO user_stats (user_id, metric_value, calculated_at)
SELECT 
  user_id,
  COUNT(*) as interaction_count,
  NOW()
FROM user_interactions
WHERE created_at >= CURRENT_DATE
GROUP BY user_id;
-- 💡 Одним запросом вместо тысяч отдельных INSERT

-- ⚡ Быстрый подсчет строк (оценка)
SELECT reltuples::bigint as estimated_rows
FROM pg_class
WHERE relname = 'large_table';
-- 🚀 Мгновенно vs медленный COUNT(*)

-- 🛡️ Безопасные транзакции для связанных операций
BEGIN;
  INSERT INTO orders (user_id, total_amount) 
  VALUES ($1, $2) 
  RETURNING id INTO @order_id;
  
  INSERT INTO order_items (order_id, product_id, quantity)
  SELECT @order_id, product_id, quantity 
  FROM cart_items 
  WHERE user_id = $1;
  
  DELETE FROM cart_items WHERE user_id = $1;
COMMIT;
-- 🎯 Либо все операции успешны, либо откат всех изменений

-- 📈 Conditional aggregation (избегаем множественных запросов)
SELECT 
  DATE(created_at) as date,
  COUNT(*) as total_events,
  COUNT(*) FILTER (WHERE event_type = 'click') as clicks,
  COUNT(*) FILTER (WHERE event_type = 'view') as views,
  COUNT(*) FILTER (WHERE event_type = 'purchase') as purchases,
  -- 📊 Конверсии на лету
  COUNT(*) FILTER (WHERE event_type = 'purchase') * 100.0 / 
  NULLIF(COUNT(*) FILTER (WHERE event_type = 'view'), 0) as conversion_rate
FROM events
WHERE created_at >= NOW() - INTERVAL '30 days'
GROUP BY DATE(created_at)
ORDER BY date;

-- 🔍 Существование записи (эффективно)
SELECT EXISTS(
  SELECT 1 FROM user_subscriptions 
  WHERE user_id = $1 
    AND status = 'active'
    AND expires_at > NOW()
) as has_active_subscription;
-- ⚡ Останавливается на первом найденном

-- 💾 Memory-efficient CTEs с материализацией
WITH MATERIALIZED popular_categories AS (
  SELECT category_id, COUNT(*) as product_count
  FROM products
  WHERE created_at >= NOW() - INTERVAL '30 days'
  GROUP BY category_id
  HAVING COUNT(*) >= 100
)
SELECT 
  c.category_name,
  pc.product_count,
  c.description
FROM popular_categories pc
JOIN categories c ON pc.category_id = c.id
ORDER BY pc.product_count DESC;
-- 🎯 MATERIALIZED заставляет PostgreSQL сохранить результат CTE в памяти
```

**⚡ Performance Optimization Checklist:**

1. **Индексы на WHERE/JOIN колонках** 📊
2. **LIMIT для больших результатов** 🔢  
3. **Параметризованные запросы** 🛡️
4. **Batch операции вместо циклов** 🔄
5. **EXISTS вместо COUNT > 0** ✅
6. **Cursor-based pagination** 📄

</details>

## 🎨 Mermaid: SQL Operations Flow

```mermaid
flowchart TD
    A[📊 Data Input] --> B{Operation Type?}
    
    B -->|Store| C[📥 INSERT]
    B -->|Modify| D[🔄 UPDATE]  
    B -->|Remove| E[🗑️ DELETE]
    B -->|Retrieve| F[🔍 SELECT]
    B -->|Analyze| G[📈 EXPLAIN]
    
    C --> H{Batch or Single?}
    H -->|Single| I[Simple INSERT]
    H -->|Batch| J[Bulk INSERT]
    J --> K[💪 RETURNING]
    I --> L[🔄 UPSERT Check]
    
    D --> M[🧠 Conditional Logic]
    M --> N[🔗 JOIN Updates] 
    N --> K
    
    E --> O[🚨 Safety Check]
    O --> P[🗑️ Soft Delete]
    O --> Q[💀 Hard Delete]
    P --> K
    Q --> K
    
    F --> R[🎨 Window Functions]
    F --> S[🔗 Complex JOINs]
    F --> T[📊 Aggregations]
    
    R --> U[📈 Analytics]
    S --> U
    T --> U
    
    G --> V[🚨 Performance Issues]
    V --> W[🔧 Optimization]
    W --> X[✅ Improved Performance]
    
    K --> Y[✅ Result]
    U --> Y
    X --> Y
    
    style A fill:#e1f5fe
    style G fill:#fff3e0
    style U fill:#f3e5f5
    style V fill:#ffebee
    style Y fill:#e8f5e8
```

## 🎯 Summary

Бро, это **арсенал SQL команд для production систем**! 🔥 

**Ключевые принципы:**
- **Безопасность**: параметризованные запросы, транзакции
- **Производительность**: индексы, batch операции, правильная пагинация  
- **Мониторинг**: EXPLAIN ANALYZE, статистика индексов
- **Масштабируемость**: window functions, эффективные JOIN'ы
- **Maintenance**: умное удаление данных, feature engineering

С такой базой ты готов строить **высокопроизводительные ML платформы** и **enterprise backend системы**! 🚀✨

# 🚀 Step-by-Step SQL Queries: От простых к сложным

<img width="800" height="800" alt="Image" src="https://github.com/user-attachments/assets/100bb263-080a-471b-afe9-a9b7d567e3e2" />

## 📖 Overview
Бро, это пошаговый гайд по построению SQL запросов - от базовых SELECT до enterprise-уровня аналитики! 🎯 Для Backend/AI Engineer'а умение декомпозировать сложную задачу на простые SQL шаги = ключ к эффективной работе с данными. Поехали строить запросы как профи! 🔥

## 🎯 Level 1: Simple Queries - Основа основ

<details>
<summary>🔍 <strong>Базовые SELECT операции - строим фундамент</strong></summary>

```sql
-- 🎯 Шаг 1: Простейший SELECT - получаем все данные
SELECT * FROM users;
-- 💡 Что делает: Возвращает ВСЕ колонки и строки из таблицы users
-- 🚨 Red Flag: Никогда не используй SELECT * в production! Трафик и память!

-- 🔍 Шаг 2: Выбираем конкретные колонки
SELECT user_id, email, created_at FROM users;
-- ✅ Правильно: Только нужные данные = быстрее и меньше трафика

-- 🎯 Шаг 3: Добавляем простое условие WHERE
SELECT user_id, email, created_at 
FROM users
WHERE status = 'active';
-- 💡 Фильтруем только активных пользователей

-- 🔢 Шаг 4: Работа с числовыми условиями
SELECT user_id, email, age
FROM users  
WHERE age >= 18              -- совершеннолетние
  AND age < 65               -- но не пенсионеры
  AND status = 'active';     -- и активные
-- 🎯 Логические операторы AND/OR для комбинирования условий

-- 📅 Шаг 5: Временные фильтры
SELECT user_id, email, created_at
FROM users
WHERE created_at >= '2024-01-01'                    -- с начала года
  AND created_at < NOW() - INTERVAL '7 days';       -- но не последнюю неделю
-- 💡 INTERVAL для относительных дат - очень удобно!

-- 🔤 Шаг 6: Текстовый поиск
SELECT user_id, email, first_name, last_name
FROM users
WHERE email ILIKE '%gmail.com'              -- регистронезависимый поиск
  OR first_name ILIKE 'john%'                -- имя начинается с John
  AND last_name IS NOT NULL;                 -- фамилия заполнена
-- 🎯 ILIKE вместо LIKE для игнорирования регистра

-- 📊 Шаг 7: Сортировка и лимит
SELECT user_id, email, created_at, last_login
FROM users
WHERE status = 'active'
ORDER BY last_login DESC                     -- сначала недавно активные
LIMIT 100;                                   -- только топ-100
-- ⚡ LIMIT экономит ресурсы и время выполнения
```

**🎯 Ключевые принципы простых запросов:**
- Всегда указывай конкретные колонки вместо `SELECT *`
- Используй индексы в WHERE условиях  
- Добавляй LIMIT для больших таблиц
- Проверяй NULL значения через `IS NULL/IS NOT NULL`

**🚨 Red Flags на базовом уровне:**
- `SELECT *` в production коде
- WHERE условия без индексов
- Забытый LIMIT на больших таблицах
- Сравнение NULL через `= NULL` (неработает!)

</details>

## 🔄 Level 2: Aggregations & Grouping - Суммируем и группируем

<details>
<summary>📊 <strong>GROUP BY и агрегации - превращаем данные в инсайты</strong></summary>

```sql
-- 🎯 Шаг 1: Простой подсчет записей
SELECT COUNT(*) as total_users FROM users;
-- 💡 Количество всех пользователей в системе

-- 📊 Шаг 2: Группировка по одному полю
SELECT 
  status,                           -- группируем по статусу
  COUNT(*) as user_count           -- считаем пользователей в каждой группе
FROM users
GROUP BY status                     -- обязательно GROUP BY для агрегации!
ORDER BY user_count DESC;          -- сначала самые большие группы
-- 🎯 Результат: active: 1500, inactive: 300, banned: 50

-- 🔍 Шаг 3: Множественные агрегации
SELECT 
  status,
  COUNT(*) as user_count,                    -- количество
  AVG(age) as avg_age,                      -- средний возраст  
  MIN(created_at) as first_registration,     -- первая регистрация
  MAX(last_login) as most_recent_login      -- последний вход
FROM users
WHERE age IS NOT NULL                       -- исключаем NULL возрасты
GROUP BY status
ORDER BY user_count DESC;

-- 📈 Шаг 4: Фильтрация групп через HAVING
SELECT 
  DATE(created_at) as registration_date,
  COUNT(*) as daily_registrations,
  AVG(age) as avg_age_of_day
FROM users
WHERE created_at >= NOW() - INTERVAL '30 days'  -- последний месяц
GROUP BY DATE(created_at)                        -- группируем по дням
HAVING COUNT(*) >= 10                           -- только дни с 10+ регистрациями  
ORDER BY registration_date DESC;
-- 💡 HAVING фильтрует группы ПОСЛЕ агрегации, WHERE - ДО

-- 🎭 Шаг 5: Сложная группировка по нескольким полям
SELECT 
  DATE_TRUNC('week', created_at) as week,    -- группируем по неделям
  country,                                   -- и по странам
  COUNT(*) as registrations,
  COUNT(DISTINCT email_domain) as unique_domains,  -- уникальные домены
  AVG(age) FILTER (WHERE age BETWEEN 18 AND 65) as working_age_avg  -- условная агрегация
FROM users u
JOIN user_profiles up ON u.user_id = up.user_id
WHERE created_at >= NOW() - INTERVAL '90 days'
GROUP BY DATE_TRUNC('week', created_at), country
HAVING COUNT(*) >= 5                         -- минимум 5 регистраций в неделю
ORDER BY week DESC, registrations DESC;

-- 📊 Шаг 6: Продвинутые агрегации для ML метрик
SELECT 
  model_name,
  DATE(created_at) as prediction_date,
  COUNT(*) as total_predictions,
  -- 📈 Статистические метрики
  AVG(confidence_score) as avg_confidence,
  STDDEV(confidence_score) as confidence_std,
  -- 🎯 Перцентили производительности  
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY response_time_ms) as median_response_time,
  PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY response_time_ms) as p95_response_time,
  -- ✅ Точность модели
  COUNT(*) FILTER (WHERE prediction_correct = true) * 100.0 / COUNT(*) as accuracy_percent,
  -- 🚨 Процент низкой уверенности
  COUNT(*) FILTER (WHERE confidence_score < 0.7) * 100.0 / COUNT(*) as low_confidence_percent
FROM model_predictions
WHERE created_at >= CURRENT_DATE - INTERVAL '7 days'
GROUP BY model_name, DATE(created_at)
HAVING COUNT(*) >= 100                       -- минимум 100 предсказаний для статистики
ORDER BY prediction_date DESC, accuracy_percent DESC;
```

**🎯 Пошаговый алгоритм построения GROUP BY запросов:**
1. **Определи что группируешь** (GROUP BY колонки)  
2. **Выбери агрегации** (COUNT, AVG, SUM, etc.)
3. **Добавь WHERE** для фильтрации исходных данных
4. **Используй HAVING** для фильтрации результатов групп  
5. **Сортируй результат** (ORDER BY)

**🚨 Red Flags в агрегациях:**
- Колонки в SELECT, которых нет в GROUP BY
- WHERE вместо HAVING для фильтрации групп
- Агрегация без учета NULL значений
- Отсутствие минимального порога в HAVING

</details>

## 🔗 Level 3: Joins - Связываем таблицы

<details>
<summary>🔗 <strong>JOIN операции - от простых до сложных связей</strong></summary>

```sql
-- 🎯 Шаг 1: Простой INNER JOIN - только совпадающие записи
SELECT 
  u.user_id,
  u.email,
  p.first_name,
  p.last_name
FROM users u
INNER JOIN user_profiles p ON u.user_id = p.user_id
WHERE u.status = 'active';
-- 💡 Получаем только пользователей, у которых есть профиль

-- 🔍 Шаг 2: LEFT JOIN - сохраняем все записи слева  
SELECT 
  u.user_id,
  u.email,
  u.created_at,
  p.first_name,           -- может быть NULL если профиля нет
  p.last_name,            -- может быть NULL
  COALESCE(p.first_name, 'Unknown') as display_name  -- заменяем NULL
FROM users u  
LEFT JOIN user_profiles p ON u.user_id = p.user_id
WHERE u.created_at >= NOW() - INTERVAL '30 days'
ORDER BY u.created_at DESC;
-- 🎯 Получаем ВСЕХ пользователей, профиль опционален

-- 🔄 Шаг 3: Множественные JOIN'ы - строим цепочку связей
SELECT 
  u.email,
  p.first_name,
  p.last_name,
  s.subscription_type,
  s.expires_at,
  -- 📊 Считаем активность пользователя
  COUNT(a.activity_id) as recent_activities
FROM users u
LEFT JOIN user_profiles p ON u.user_id = p.user_id
LEFT JOIN subscriptions s ON u.user_id = s.user_id 
  AND s.status = 'active'                    -- только активные подписки
LEFT JOIN user_activities a ON u.user_id = a.user_id
  AND a.created_at >= NOW() - INTERVAL '7 days'  -- только недавние активности
WHERE u.status = 'active'
GROUP BY u.user_id, u.email, p.first_name, p.last_name, s.subscription_type, s.expires_at
ORDER BY recent_activities DESC;
-- 💡 Пошагово джоиним: users -> profiles -> subscriptions -> activities

-- 🎭 Шаг 4: Self JOIN - связываем таблицу саму с собой
SELECT 
  e.employee_id,
  e.name as employee_name,
  e.position,
  m.name as manager_name,        -- имя руководителя
  m.position as manager_position
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id  -- self join!
WHERE e.department = 'Engineering'
ORDER BY m.name, e.name;
-- 🎯 Получаем сотрудников с информацией об их руководителях

-- 📊 Шаг 5: JOIN с агрегированными данными
SELECT 
  u.user_id,
  u.email,
  u.created_at,
  -- 🛒 Статистика заказов из подзапроса
  order_stats.total_orders,
  order_stats.total_spent,
  order_stats.avg_order_value,
  order_stats.last_order_date,
  -- 🏷️ Категоризация клиентов
  CASE 
    WHEN order_stats.total_spent > 1000 THEN 'VIP'
    WHEN order_stats.total_spent > 100 THEN 'Regular'
    ELSE 'New'
  END as customer_segment
FROM users u
LEFT JOIN (
  -- 📈 Агрегируем статистику заказов для каждого пользователя
  SELECT 
    user_id,
    COUNT(*) as total_orders,
    SUM(total_amount) as total_spent,
    AVG(total_amount) as avg_order_value,
    MAX(created_at) as last_order_date
  FROM orders
  WHERE status = 'completed'
    AND created_at >= NOW() - INTERVAL '1 year'  -- за последний год
  GROUP BY user_id
) order_stats ON u.user_id = order_stats.user_id
WHERE u.status = 'active'
ORDER BY order_stats.total_spent DESC NULLS LAST;  -- VIP клиенты сверху

-- 🤖 Шаг 6: Сложный JOIN для ML pipeline
SELECT 
  u.user_id,
  u.email,
  -- 👤 Профильные данные
  p.age_group,
  p.country,
  -- 🎭 Поведенческие метрики
  behavior.session_count,
  behavior.avg_session_duration,
  behavior.total_interactions,
  -- 🤖 AI предсказания
  predictions.churn_probability,
  predictions.recommended_actions,
  predictions.confidence_score,
  -- 📊 Engagement скор  
  engagement.email_open_rate,
  engagement.click_through_rate,
  engagement.last_engagement_date
FROM users u
LEFT JOIN user_profiles p ON u.user_id = p.user_id
-- 📈 Агрегированное поведение за последние 30 дней
LEFT JOIN (
  SELECT 
    user_id,
    COUNT(DISTINCT session_id) as session_count,
    AVG(duration_minutes) as avg_session_duration,
    SUM(interaction_count) as total_interactions
  FROM user_sessions
  WHERE created_at >= NOW() - INTERVAL '30 days'
  GROUP BY user_id
) behavior ON u.user_id = behavior.user_id
-- 🧠 Последние AI предсказания  
LEFT JOIN (
  SELECT DISTINCT ON (user_id)  -- только последний прогноз
    user_id,
    churn_probability,
    recommended_actions,
    confidence_score
  FROM ml_user_predictions
  WHERE model_version = 'v2.1'
    AND created_at >= NOW() - INTERVAL '7 days'
  ORDER BY user_id, created_at DESC
) predictions ON u.user_id = predictions.user_id
-- 📧 Email engagement метрики
LEFT JOIN (
  SELECT 
    user_id,
    COUNT(*) FILTER (WHERE opened = true) * 100.0 / COUNT(*) as email_open_rate,
    COUNT(*) FILTER (WHERE clicked = true) * 100.0 / COUNT(*) as click_through_rate,
    MAX(created_at) as last_engagement_date
  FROM email_campaigns
  WHERE sent_at >= NOW() - INTERVAL '90 days'
  GROUP BY user_id
) engagement ON u.user_id = engagement.user_id
WHERE u.status = 'active'
  AND u.created_at >= NOW() - INTERVAL '180 days'  -- пользователи за полгода
ORDER BY predictions.churn_probability DESC NULLS LAST;
```

**🎯 Пошаговый алгоритм построения JOIN'ов:**
1. **Начни с главной таблицы** (FROM)
2. **Определи тип связи** (INNER/LEFT/RIGHT/FULL)  
3. **Укажи условие связи** (ON колонка1 = колонка2)
4. **Добавляй JOIN'ы поэтапно**, проверяя каждый
5. **Группируй при необходимости** (если есть агрегации)
6. **Оптимизируй порядок JOIN'ов** (меньшие таблицы первыми)

**🚨 Red Flags в JOIN'ах:**
- JOIN без индексов на связанных колонках
- CROSS JOIN вместо INNER JOIN (декартово произведение!)
- Множественные LEFT JOIN без необходимости
- Забытый GROUP BY при агрегации в JOIN'ах

</details>

## 🎨 Level 4: Window Functions - Аналитические функции

<details>
<summary>🎨 <strong>Window Functions - аналитика без GROUP BY</strong></summary>

```sql
-- 🎯 Шаг 1: Базовая оконная функция - ранжирование
SELECT 
  product_id,
  product_name,
  category,
  price,
  -- 🏆 Ранг по цене внутри категории
  RANK() OVER (PARTITION BY category ORDER BY price DESC) as price_rank,
  -- 📊 Порядковый номер (без пропусков)
  ROW_NUMBER() OVER (PARTITION BY category ORDER BY price DESC) as price_position
FROM products
WHERE is_active = true
ORDER BY category, price_rank;
-- 💡 PARTITION BY = группировка, ORDER BY = сортировка внутри группы

-- 📈 Шаг 2: Сравнение с предыдущими/следующими записями
SELECT 
  user_id,
  login_date,
  session_duration_minutes,
  -- 📊 Сравнение с предыдущей сессией
  LAG(session_duration_minutes) OVER (
    PARTITION BY user_id ORDER BY login_date
  ) as prev_session_duration,
  -- 🔍 Изменение длительности сессии  
  session_duration_minutes - LAG(session_duration_minutes) OVER (
    PARTITION BY user_id ORDER BY login_date  
  ) as duration_change,
  -- 📈 Следующая сессия
  LEAD(login_date) OVER (
    PARTITION BY user_id ORDER BY login_date
  ) as next_login_date,
  -- ⏰ Интервал между сессиями
  LEAD(login_date) OVER (
    PARTITION BY user_id ORDER BY login_date
  ) - login_date as time_to_next_session
FROM user_sessions
WHERE login_date >= NOW() - INTERVAL '30 days'
ORDER BY user_id, login_date;
-- 🎯 Анализируем изменения поведения пользователей во времени

-- 📊 Шаг 3: Накопительные (кумулятивные) функции  
SELECT 
  DATE(order_date) as day,
  daily_revenue,
  daily_orders,
  -- 📈 Кумулятивная выручка
  SUM(daily_revenue) OVER (
    ORDER BY DATE(order_date)
    ROWS UNBOUNDED PRECEDING  -- от начала до текущей строки
  ) as cumulative_revenue,
  -- 📊 Скользящая средняя за 7 дней
  AVG(daily_revenue) OVER (
    ORDER BY DATE(order_date)
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW  -- текущая + 6 предыдущих
  ) as moving_avg_7_days,
  -- 🎯 Процентное изменение к предыдущему дню
  (daily_revenue - LAG(daily_revenue) OVER (ORDER BY DATE(order_date))) 
  / LAG(daily_revenue) OVER (ORDER BY DATE(order_date)) * 100 as daily_change_percent
FROM (
  -- 📈 Агрегируем данные по дням
  SELECT 
    DATE(created_at) as order_date,
    SUM(total_amount) as daily_revenue,
    COUNT(*) as daily_orders
  FROM orders
  WHERE status = 'completed'
    AND created_at >= NOW() - INTERVAL '90 days'
  GROUP BY DATE(created_at)
) daily_stats
ORDER BY day;

-- 🎭 Шаг 4: Процентили и статистические функции
SELECT 
  model_name,
  prediction_date,
  response_time_ms,
  confidence_score,
  -- 📊 Процентильный ранг производительности
  PERCENT_RANK() OVER (
    PARTITION BY model_name 
    ORDER BY response_time_ms
  ) * 100 as response_time_percentile,
  -- 🎯 Квартили уверенности
  NTILE(4) OVER (
    PARTITION BY model_name 
    ORDER BY confidence_score
  ) as confidence_quartile,
  -- 📈 Z-score для выявления аномалий
  (confidence_score - AVG(confidence_score) OVER (PARTITION BY model_name)) 
  / STDDEV(confidence_score) OVER (PARTITION BY model_name) as confidence_z_score,
  -- 🚨 Флаг аномальных предсказаний
  CASE 
    WHEN ABS((confidence_score - AVG(confidence_score) OVER (PARTITION BY model_name)) 
        / STDDEV(confidence_score) OVER (PARTITION BY model_name)) > 2 
    THEN 'Anomaly'
    ELSE 'Normal'
  END as anomaly_flag
FROM model_predictions
WHERE prediction_date >= CURRENT_DATE - INTERVAL '7 days'
ORDER BY model_name, prediction_date, response_time_ms;

-- 🎨 Шаг 5: Комплексный анализ с множественными окнами
SELECT 
  u.user_id,
  u.email,
  s.session_date,
  s.duration_minutes,
  s.pages_viewed,
  -- 🏆 Ранжирование сессий пользователя  
  ROW_NUMBER() OVER (
    PARTITION BY u.user_id 
    ORDER BY s.duration_minutes DESC
  ) as session_rank_by_duration,
  -- 📊 Процентиль длительности среди всех пользователей
  PERCENT_RANK() OVER (
    ORDER BY s.duration_minutes
  ) * 100 as global_duration_percentile,
  -- 📈 Скользящее среднее активности пользователя
  AVG(s.pages_viewed) OVER (
    PARTITION BY u.user_id 
    ORDER BY s.session_date
    ROWS BETWEEN 4 PRECEDING AND CURRENT ROW  -- последние 5 сессий
  ) as user_avg_activity_trend,
  -- 🎯 Сравнение с общим средним
  s.duration_minutes - AVG(s.duration_minutes) OVER () as duration_vs_global_avg,
  -- 📊 Первая и последняя сессия пользователя
  FIRST_VALUE(s.session_date) OVER (
    PARTITION BY u.user_id 
    ORDER BY s.session_date 
    ROWS UNBOUNDED PRECEDING
  ) as first_session_date,
  LAST_VALUE(s.session_date) OVER (
    PARTITION BY u.user_id 
    ORDER BY s.session_date 
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
  ) as last_session_date,
  -- ⏰ Дни с первой сессии
  s.session_date - FIRST_VALUE(s.session_date) OVER (
    PARTITION BY u.user_id 
    ORDER BY s.session_date 
    ROWS UNBOUNDED PRECEDING
  ) as days_since_first_session
FROM users u
JOIN user_sessions s ON u.user_id = s.user_id
WHERE s.session_date >= NOW() - INTERVAL '60 days'
  AND u.status = 'active'
ORDER BY u.user_id, s.session_date;
```

**🎯 Ключевые концепции Window Functions:**
- **PARTITION BY** - как GROUP BY, но без схлопывания строк
- **ORDER BY** - сортировка внутри окна  
- **ROWS/RANGE** - границы окна для расчета
- **Аналитические функции** работают ПОСЛЕ WHERE/GROUP BY/HAVING

**🚨 Red Flags в Window Functions:**
- Отсутствие индексов на PARTITION BY/ORDER BY колонках
- Слишком большие окна (UNBOUNDED) на больших таблицах
- Множественные одинаковые OVER() клаузы (можно вынести в WINDOW)

</details>

## 🧠 Level 5: CTEs & Subqueries - Разбиваем сложность

<details>
<summary>🧠 <strong>Common Table Expressions - структурируем сложную логику</strong></summary>

```sql
-- 🎯 Шаг 1: Базовый CTE - заменяем подзапрос на читаемый блок
WITH active_users AS (
  -- 👥 Первый шаг: определяем активных пользователей
  SELECT user_id, email, created_at
  FROM users  
  WHERE status = 'active'
    AND last_login >= NOW() - INTERVAL '30 days'
)
SELECT 
  au.user_id,
  au.email,
  COUNT(o.order_id) as recent_orders
FROM active_users au  -- используем CTE как обычную таблицу
LEFT JOIN orders o ON au.user_id = o.user_id
  AND o.created_at >= NOW() - INTERVAL '30 days'
GROUP BY au.user_id, au.email
ORDER BY recent_orders DESC;
-- 💡 CTE делает запрос читаемым и позволяет переиспользовать логику

-- 📊 Шаг 2: Множественные CTE - пошаговая декомпозиция задачи
WITH user_stats AS (
  -- 📈 Шаг 1: Агрегируем статистику пользователей
  SELECT 
    user_id,
    COUNT(*) as total_sessions,
    AVG(duration_minutes) as avg_session_duration,
    SUM(pages_viewed) as total_pages_viewed,
    MAX(created_at) as last_activity
  FROM user_sessions
  WHERE created_at >= NOW() - INTERVAL '90 days'
  GROUP BY user_id
),
purchase_stats AS (
  -- 💰 Шаг 2: Статистика покупок
  SELECT 
    user_id,
    COUNT(*) as total_orders,
    SUM(total_amount) as total_spent,
    AVG(total_amount) as avg_order_value
  FROM orders
  WHERE status = 'completed'
    AND created_at >= NOW() - INTERVAL '90 days'
  GROUP BY user_id  
),
engagement_scores AS (
  -- 🎯 Шаг 3: Вычисляем engagement score
  SELECT 
    u.user_id,
    u.email,
    us.total_sessions,
    us.avg_session_duration,
    ps.total_orders,
    ps.total_spent,
    -- 📊 Комплексный engagement score
    (
      COALESCE(us.total_sessions, 0) * 0.3 +           -- вес активности 30%
      COALESCE(us.avg_session_duration, 0) * 0.2 +     -- вес времени 20% 
      COALESCE(ps.total_orders, 0) * 5 * 0.3 +         -- вес покупок 30%
      CASE WHEN ps.total_spent > 100 THEN 20 ELSE 0 END * 0.2  -- бонус за трату 20%
    ) as engagement_score
  FROM users u
  LEFT JOIN user_stats us ON u.user_id = us.user_id  
  LEFT JOIN purchase_stats ps ON u.user_id = ps.user_id
  WHERE u.status = 'active'
)
-- 🏆 Финальный SELECT с использованием всех CTE  
SELECT 
  user_id,
  email,
  total_sessions,
  total_orders,
  total_spent,
  engagement_score,
  -- 🎭 Сегментация по engagement
  CASE 
    WHEN engagement_score >= 80 THEN 'Champion'
    WHEN engagement_score >= 60 THEN 'Loyal'  
    WHEN engagement_score >= 40 THEN 'Potential'
    WHEN engagement_score >= 20 THEN 'New'
    ELSE 'At Risk'
  END as user_segment,
  -- 📈 Ранк внутри сегмента
  RANK() OVER (
    PARTITION BY CASE 
      WHEN engagement_score >= 80 THEN 'Champion'
      WHEN engagement_score >= 60 THEN 'Loyal'
      ELSE 'Other'
    END 
    ORDER BY engagement_score DESC
  ) as segment_rank
FROM engagement_scores
ORDER BY engagement_score DESC;

-- 🔄 Шаг 3: Recursive CTE - работаем с иерархическими данными
WITH RECURSIVE category_hierarchy AS (
  -- 🌱 Базовый случай: корневые категории
  SELECT 
    category_id,
    category_name,
    parent_id,
    0 as level,                    -- уровень вложенности
    category_name as full_path     -- полный путь
  FROM categories
  WHERE parent_id IS NULL         -- начинаем с корневых категорий
  
  UNION ALL
  
  -- 🌳 Рекурсивная часть: дочерние категории
  SELECT 
    c.category_id,
    c.category_name,
    c.parent_id,
    ch.level + 1,                               -- увеличиваем уровень
    ch.full_path || ' > ' || c.category_name    -- строим полный путь
  FROM categories c
  INNER JOIN category_hierarchy ch ON c.parent_id = ch.category_id
  WHERE ch.level < 5              -- ограничиваем глубину рекурсии
),
category_with_products AS (
  -- 📦 Добавляем количество продуктов в каждой категории
  SELECT
```sql
  ch.*,
  COALESCE(p.product_count, 0) as product_count,
  COALESCE(p.avg_price, 0) as avg_price
FROM category_hierarchy ch
LEFT JOIN (
  SELECT 
    category_id,
    COUNT(*) as product_count,
    AVG(price) as avg_price
  FROM products
  WHERE is_active = true
  GROUP BY category_id
) p ON ch.category_id = p.category_id
)
SELECT 
  category_id,
  REPEAT('  ', level) || category_name as indented_name,  -- визуальная иерархия
  full_path,
  level,
  product_count,
  avg_price::numeric(10,2),
  -- 🎯 Индикаторы категории
  CASE 
    WHEN level = 0 THEN 'Root Category'
    WHEN product_count = 0 THEN 'Empty Category'
    WHEN product_count > 100 THEN 'Popular Category'
    ELSE 'Standard Category'
  END as category_type
FROM category_with_products
ORDER BY full_path;

-- 🚀 Шаг 4: Материализованный CTE для производительности
WITH MATERIALIZED popular_products AS (
  -- 💪 MATERIALIZED заставляет PostgreSQL сохранить результат в памяти
  SELECT 
    product_id,
    product_name,
    category_id,
    price,
    AVG(rating) as avg_rating,
    COUNT(review_id) as review_count
  FROM products p
  LEFT JOIN product_reviews pr ON p.product_id = pr.product_id
  WHERE p.is_active = true
    AND p.created_at >= NOW() - INTERVAL '1 year'
  GROUP BY p.product_id, p.product_name, p.category_id, p.price
  HAVING COUNT(pr.review_id) >= 10        -- минимум 10 отзывов
    AND AVG(pr.rating) >= 4.0             -- высокий рейтинг
),
category_stats AS (
  -- 📊 Статистика по категориям на основе популярных продуктов
  SELECT 
    c.category_name,
    COUNT(pp.product_id) as popular_product_count,
    AVG(pp.avg_rating) as category_avg_rating,
    AVG(pp.price) as category_avg_price,
    SUM(pp.review_count) as total_reviews
  FROM categories c
  JOIN popular_products pp ON c.category_id = pp.category_id
  GROUP BY c.category_id, c.category_name
)
-- 🏆 Финальная аналитика категорий
SELECT 
  category_name,
  popular_product_count,
  category_avg_rating::numeric(3,2),
  category_avg_price::numeric(10,2),
  total_reviews,
  -- 📈 Ранжирование категорий
  RANK() OVER (ORDER BY popular_product_count DESC) as popularity_rank,
  RANK() OVER (ORDER BY category_avg_rating DESC) as quality_rank,
  -- 🎯 Комбинированный score
  (popular_product_count * 0.4 + category_avg_rating * 20 * 0.6) as combined_score
FROM category_stats
ORDER BY combined_score DESC;

-- 🎨 Шаг 5: Сложный CTE для ML feature engineering
WITH user_behavior_windows AS (
  -- ⏰ Создаем временные окна для анализа поведения
  SELECT 
    user_id,
    -- 📊 Метрики за разные периоды
    COUNT(*) FILTER (WHERE created_at >= NOW() - INTERVAL '7 days') as actions_7d,
    COUNT(*) FILTER (WHERE created_at >= NOW() - INTERVAL '30 days') as actions_30d,
    COUNT(*) FILTER (WHERE created_at >= NOW() - INTERVAL '90 days') as actions_90d,
    -- 🎭 Типы активности
    COUNT(*) FILTER (WHERE action_type = 'view' AND created_at >= NOW() - INTERVAL '7 days') as views_7d,
    COUNT(*) FILTER (WHERE action_type = 'click' AND created_at >= NOW() - INTERVAL '7 days') as clicks_7d,
    COUNT(*) FILTER (WHERE action_type = 'purchase' AND created_at >= NOW() - INTERVAL '7 days') as purchases_7d,
    -- ⏰ Временные паттерны
    COUNT(*) FILTER (WHERE EXTRACT(hour FROM created_at) BETWEEN 9 AND 17) as business_hours_actions,
    COUNT(*) FILTER (WHERE EXTRACT(dow FROM created_at) IN (0,6)) as weekend_actions
  FROM user_actions
  WHERE created_at >= NOW() - INTERVAL '90 days'
  GROUP BY user_id
),
user_features AS (
  -- 🧠 Инженерия признаков для ML модели
  SELECT 
    ubw.*,
    -- 📈 Тренды активности
    CASE 
      WHEN actions_90d > 0 THEN actions_7d::float / (actions_90d::float / 13)  -- нормализация на неделю
      ELSE 0 
    END as activity_trend,
    -- 🎯 Конверсионные показатели
    CASE 
      WHEN views_7d > 0 THEN clicks_7d::float / views_7d 
      ELSE 0 
    END as click_through_rate,
    CASE 
      WHEN clicks_7d > 0 THEN purchases_7d::float / clicks_7d 
      ELSE 0 
    END as purchase_conversion_rate,
    -- 🕐 Поведенческие паттерны
    CASE 
      WHEN (business_hours_actions + weekend_actions) > 0 
      THEN business_hours_actions::float / (business_hours_actions + weekend_actions)
      ELSE 0 
    END as business_hours_ratio,
    -- 📊 Категоризация активности
    CASE 
      WHEN actions_7d >= 50 THEN 'high_activity'
      WHEN actions_7d >= 10 THEN 'medium_activity'  
      WHEN actions_7d >= 1 THEN 'low_activity'
      ELSE 'inactive'
    END as activity_segment
  FROM user_behavior_windows ubw
),
ml_dataset AS (
  -- 🤖 Финальный датасет для ML модели
  SELECT 
    uf.user_id,
    u.registration_date,
    u.age,
    u.country,
    -- 📊 Поведенческие признаки
    uf.actions_7d,
    uf.actions_30d,
    uf.activity_trend,
    uf.click_through_rate,
    uf.purchase_conversion_rate,
    uf.business_hours_ratio,
    uf.activity_segment,
    -- 🎯 Целевая переменная (churn prediction)
    CASE 
      WHEN uf.actions_7d = 0 AND uf.actions_30d <= 2 THEN 1 
      ELSE 0 
    END as churn_risk_label,
    -- 📈 Дополнительные признаки
    EXTRACT(days FROM NOW() - u.registration_date) as days_since_registration,
    COALESCE(s.subscription_tier, 'free') as subscription_tier
  FROM user_features uf
  JOIN users u ON uf.user_id = u.user_id
  LEFT JOIN subscriptions s ON u.user_id = s.user_id 
    AND s.status = 'active'
  WHERE u.status = 'active'
    AND u.registration_date <= NOW() - INTERVAL '30 days'  -- исключаем совсем новых
)
-- 🎯 Итоговый датасет с дополнительной аналитикой
SELECT 
  *,
  -- 📊 Статистики по сегментам
  AVG(click_through_rate) OVER (PARTITION BY activity_segment) as segment_avg_ctr,
  AVG(purchase_conversion_rate) OVER (PARTITION BY country) as country_avg_conversion,
  -- 🏆 Ранжирование пользователей
  NTILE(10) OVER (ORDER BY activity_trend DESC) as activity_decile,
  -- 🚨 Флаги для аномалий
  CASE 
    WHEN activity_trend > 5 THEN 'potential_bot'
    WHEN churn_risk_label = 1 AND subscription_tier != 'free' THEN 'revenue_at_risk'
    ELSE 'normal'
  END as anomaly_flag
FROM ml_dataset
ORDER BY churn_risk_label DESC, activity_trend DESC;
```

**🎯 Пошаговый подход к CTE:**
1. **Разбей сложную задачу** на логические шаги
2. **Каждый CTE = один шаг** обработки данных  
3. **Называй CTE описательно** (user_stats, не tmp1)
4. **Используй MATERIALIZED** для тяжелых промежуточных результатов
5. **Тестируй каждый CTE отдельно** перед объединением

**🚨 Red Flags в CTE:**
- Слишком много вложенных CTE (>5-6)
- CTE без MATERIALIZED для тяжелых вычислений
- Дублирование логики между CTE
- Отсутствие комментариев к назначению каждого CTE

</details>

## 🔥 Level 6: Advanced Patterns - Production-Ready Queries

<details>
<summary>🚀 <strong>Enterprise-уровень запросы - для production систем</strong></summary>

```sql
-- 🎯 Шаг 1: Динамический поиск с полнотекстовым индексом
SELECT 
  p.product_id,
  p.product_name,
  p.description,
  c.category_name,
  p.price,
  -- 📊 Ранжирование релевантности поиска  
  ts_rank(
    to_tsvector('russian', p.product_name || ' ' || p.description),
    plainto_tsquery('russian', $1)  -- поисковый запрос пользователя
  ) as relevance_score,
  -- 🎯 Подсветка совпадений
  ts_headline(
    'russian',
    p.description,
    plainto_tsquery('russian', $1),
    'MaxWords=20, MinWords=5, StartSel=<b>, StopSel=</b>'
  ) as highlighted_description
FROM products p
JOIN categories c ON p.category_id = c.category_id  
WHERE to_tsvector('russian', p.product_name || ' ' || p.description) 
      @@ plainto_tsquery('russian', $1)    -- полнотекстовый поиск
  AND p.is_active = true
  AND p.stock_quantity > 0                 -- только в наличии
  AND ($2::text IS NULL OR c.category_name = $2)  -- опциональный фильтр категории
  AND ($3::numeric IS NULL OR p.price <= $3)      -- опциональный фильтр цены
ORDER BY relevance_score DESC, p.created_at DESC
LIMIT 50;

-- 🚀 Шаг 2: Сложная аналитика продаж с временными рядами
WITH RECURSIVE date_series AS (
  -- 📅 Генерируем непрерывную серию дат
  SELECT DATE($1::timestamp) as date_val  -- начальная дата
  UNION ALL  
  SELECT date_val + INTERVAL '1 day'
  FROM date_series
  WHERE date_val < DATE($2::timestamp)    -- конечная дата
),
daily_sales AS (
  -- 💰 Агрегируем продажи по дням
  SELECT 
    DATE(o.created_at) as sale_date,
    COUNT(o.order_id) as orders_count,
    SUM(o.total_amount) as revenue,
    COUNT(DISTINCT o.user_id) as unique_customers,
    AVG(o.total_amount) as avg_order_value,
    -- 📊 Метрики по типам клиентов  
    COUNT(o.order_id) FILTER (WHERE o.is_first_order = true) as new_customer_orders,
    SUM(o.total_amount) FILTER (WHERE o.is_first_order = false) as returning_customer_revenue
  FROM orders o
  WHERE o.status = 'completed'
    AND DATE(o.created_at) BETWEEN DATE($1::timestamp) AND DATE($2::timestamp)
  GROUP BY DATE(o.created_at)
),
sales_with_trends AS (
  -- 📈 Добавляем тренды и сравнения
  SELECT 
    ds.date_val,
    COALESCE(s.orders_count, 0) as orders_count,
    COALESCE(s.revenue, 0) as revenue,
    COALESCE(s.unique_customers, 0) as unique_customers,
    COALESCE(s.avg_order_value, 0) as avg_order_value,
    -- 📊 Скользящие средние
    AVG(COALESCE(s.revenue, 0)) OVER (
      ORDER BY ds.date_val 
      ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) as revenue_7day_ma,
    AVG(COALESCE(s.orders_count, 0)) OVER (
      ORDER BY ds.date_val
      ROWS BETWEEN 6 PRECEDING AND CURRENT ROW  
    ) as orders_7day_ma,
    -- 📈 Сравнение с предыдущим периодом
    LAG(COALESCE(s.revenue, 0), 7) OVER (ORDER BY ds.date_val) as revenue_week_ago,
    LAG(COALESCE(s.orders_count, 0), 7) OVER (ORDER BY ds.date_val) as orders_week_ago,
    -- 🎯 Кумулятивные метрики  
    SUM(COALESCE(s.revenue, 0)) OVER (ORDER BY ds.date_val) as cumulative_revenue,
    -- 📅 Информация о дне недели
    EXTRACT(dow FROM ds.date_val) as day_of_week,
    CASE EXTRACT(dow FROM ds.date_val)
      WHEN 0 THEN 'Sunday'    WHEN 1 THEN 'Monday'
      WHEN 2 THEN 'Tuesday'   WHEN 3 THEN 'Wednesday'  
      WHEN 4 THEN 'Thursday'  WHEN 5 THEN 'Friday'
      WHEN 6 THEN 'Saturday'
    END as day_name
  FROM date_series ds
  LEFT JOIN daily_sales s ON ds.date_val = s.sale_date
)
SELECT 
  date_val,
  day_name,
  orders_count,
  revenue::numeric(10,2),
  unique_customers,
  avg_order_value::numeric(10,2),
  revenue_7day_ma::numeric(10,2),
  -- 📈 Процентные изменения
  CASE 
    WHEN revenue_week_ago > 0 
    THEN ((revenue - revenue_week_ago) / revenue_week_ago * 100)::numeric(5,2)
    ELSE NULL 
  END as revenue_wow_change_percent,
  -- 🎯 Классификация дней
  CASE 
    WHEN revenue > revenue_7day_ma * 1.2 THEN 'High Performance'
    WHEN revenue < revenue_7day_ma * 0.8 THEN 'Low Performance'  
    ELSE 'Normal'
  END as performance_category,
  -- 📊 Флаги аномалий
  CASE
    WHEN orders_count = 0 AND day_name NOT IN ('Sunday', 'Saturday') THEN 'No Sales'
    WHEN revenue > revenue_7day_ma * 3 THEN 'Revenue Spike'
    WHEN orders_count > orders_7day_ma * 3 THEN 'Volume Spike'
    ELSE 'Normal'  
  END as anomaly_flag
FROM sales_with_trends
ORDER BY date_val;

-- 🤖 Шаг 3: ML модель scoring с batch обработкой
WITH user_features AS (
  -- 🧠 Подготавливаем признаки для ML модели
  SELECT 
    u.user_id,
    u.email,
    u.registration_date,
    -- 📊 Поведенческие метрики за разные окна
    COALESCE(recent.actions_30d, 0) as actions_30d,
    COALESCE(recent.avg_session_duration, 0) as avg_session_duration,
    COALESCE(recent.pages_per_session, 0) as pages_per_session,
    COALESCE(purchase.orders_90d, 0) as orders_90d,
    COALESCE(purchase.total_spent_90d, 0) as total_spent_90d,
    COALESCE(purchase.avg_order_value, 0) as avg_order_value,
    -- ⏰ Временные признаки
    EXTRACT(days FROM NOW() - u.registration_date) as days_since_registration,
    EXTRACT(days FROM NOW() - u.last_login) as days_since_last_login,
    -- 🎭 Категориальные признаки
    COALESCE(u.country, 'Unknown') as country,
    COALESCE(u.age_group, 'Unknown') as age_group,
    COALESCE(s.tier, 'free') as subscription_tier
  FROM users u
  -- 📈 Недавняя активность
  LEFT JOIN (
    SELECT 
      user_id,
      COUNT(*) as actions_30d,
      AVG(session_duration_minutes) as avg_session_duration,
      AVG(pages_viewed) as pages_per_session
    FROM user_sessions
    WHERE created_at >= NOW() - INTERVAL '30 days'
    GROUP BY user_id
  ) recent ON u.user_id = recent.user_id
  -- 💰 История покупок
  LEFT JOIN (
    SELECT 
      user_id,
      COUNT(*) as orders_90d,
      SUM(total_amount) as total_spent_90d,
      AVG(total_amount) as avg_order_value
    FROM orders
    WHERE status = 'completed' 
      AND created_at >= NOW() - INTERVAL '90 days'
    GROUP BY user_id
  ) purchase ON u.user_id = purchase.user_id
  LEFT JOIN subscriptions s ON u.user_id = s.user_id 
    AND s.status = 'active'
  WHERE u.status = 'active'
),
ml_scores AS (
  -- 🎯 Применяем ML модель для scoring (упрощенная логика)
  SELECT 
    *,
    -- 🧮 Churn probability (simplified model)
    CASE 
      WHEN days_since_last_login > 30 THEN 0.8
      WHEN days_since_last_login > 14 THEN 0.6  
      WHEN actions_30d = 0 THEN 0.7
      WHEN orders_90d = 0 AND days_since_registration > 60 THEN 0.5
      ELSE 0.1
    END as churn_probability,
    -- 💰 Lifetime Value prediction  
    CASE 
      WHEN subscription_tier = 'premium' THEN total_spent_90d * 4 * 1.5
      WHEN orders_90d >= 3 THEN total_spent_90d * 4 * 1.2
      ELSE total_spent_90d * 4
    END as predicted_ltv_12m,
    -- 📈 Engagement score
    LEAST(100, (
      COALESCE(actions_30d, 0) * 2 +
      CASE WHEN days_since_last_login <= 7 THEN 20 ELSE 0 END +
      COALESCE(orders_90d, 0) * 10 +
      CASE WHEN subscription_tier != 'free' THEN 15 ELSE 0 END
    )) as engagement_score
  FROM user_features
),
segmentation AS (
  -- 🎭 Сегментация пользователей на основе scores
  SELECT 
    *,
    -- 🚨 Risk segments
    CASE 
      WHEN churn_probability >= 0.7 THEN 'High Risk'
      WHEN churn_probability >= 0.4 THEN 'Medium Risk'  
      ELSE 'Low Risk'
    END as churn_risk_segment,
    -- 💎 Value segments  
    CASE 
      WHEN predicted_ltv_12m >= 1000 THEN 'High Value'
      WHEN predicted_ltv_12m >= 200 THEN 'Medium Value'
      ELSE 'Low Value' 
    END as value_segment,
    -- 🎯 Action recommendations
    CASE 
      WHEN churn_probability >= 0.7 AND predicted_ltv_12m >= 500 
        THEN 'Urgent: Personal Outreach'
      WHEN churn_probability >= 0.7 
        THEN 'Send Retention Campaign'
      WHEN engagement_score >= 80 AND subscription_tier = 'free'
        THEN 'Upsell Opportunity'  
      WHEN engagement_score < 20
        THEN 'Re-engagement Campaign'
      ELSE 'Monitor'
    END as recommended_action
  FROM ml_scores
)
-- 📊 Финальный результат с приоритизацией
SELECT 
  user_id,
  email,
  churn_probability,
  predicted_ltv_12m::numeric(10,2),
  engagement_score,
  churn_risk_segment,
  value_segment,
  recommended_action,
  -- 📈 Приоритет действий
  CASE 
    WHEN recommended_action = 'Urgent: Personal Outreach' THEN 1
    WHEN recommended_action = 'Send Retention Campaign' THEN 2
    WHEN recommended_action = 'Upsell Opportunity' THEN 3
    WHEN recommended_action = 'Re-engagement Campaign' THEN 4
    ELSE 5
  END as action_priority,
  -- 📅 Рекомендуемая дата действия
  CASE 
    WHEN recommended_action LIKE 'Urgent%' THEN CURRENT_DATE
    WHEN churn_probability >= 0.5 THEN CURRENT_DATE + INTERVAL '2 days'
    ELSE CURRENT_DATE + INTERVAL '1 week'
  END as recommended_action_date
FROM segmentation  
WHERE recommended_action != 'Monitor'  -- исключаем пользователей без действий
ORDER BY action_priority, predicted_ltv_12m DESC
LIMIT 1000;  -- топ-1000 для обработки

-- 🔧 Шаг 4: Performance monitoring и query optimization
WITH query_performance AS (
  -- 📊 Анализ производительности запросов
  SELECT 
    LEFT(query, 80) as query_preview,
    calls,
    total_time,
    mean_time,
    stddev_time,
    rows,
    -- 📈 Вычисляем метрики производительности
    (total_time / sum(total_time) OVER() * 100)::numeric(5,2) as percent_total_time,
    (calls / sum(calls) OVER() * 100)::numeric(5,2) as percent_total_calls,
    -- 🎯 Классификация запросов
    CASE 
      WHEN mean_time > 1000 THEN 'Slow'
      WHEN calls > 1000 AND mean_time > 100 THEN 'High Frequency + Slow'
      WHEN calls > 10000 THEN 'High Frequency'
      ELSE 'Normal'
    END as query_category
  FROM pg_stat_statements
  WHERE calls >= 10  -- минимум 10 выполнений
),
index_efficiency AS (
  -- 🔍 Анализ эффективности индексов
  SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch,
    pg_size_pretty(pg_relation_size(indexname::regclass)) as index_size,
    -- 📊 Метрики эффективности
    CASE 
      WHEN idx_tup_read > 0 
      THEN (idx_tup_fetch::float / idx_tup_read * 100)::numeric(5,2)
      ELSE 0 
    END as selectivity_percent,
    -- 🚨 Проблемные индексы
    CASE 
      WHEN idx_scan = 0 THEN 'Unused'
      WHEN idx_scan < 100 AND pg_relation_size(indexname::regclass) > 1024*1024 
        THEN 'Underused Large Index'
      WHEN idx_tup_read > idx_tup_fetch * 10 
        THEN 'Low Selectivity'
      ELSE 'Healthy'
    END as index_health
  FROM pg_stat_user_indexes
)
-- 🎯 Рекомендации по оптимизации
SELECT 
  'Query Performance' as analysis_type,
  query_preview as item_name,
  query_category as status,
  mean_time::numeric(10,2) as avg_time_ms,
  calls as frequency,
  -- 💡 Рекомендации
  CASE 
    WHEN query_category = 'Slow' 
      THEN 'Add indexes, optimize WHERE clauses, consider rewriting'
    WHEN query_category = 'High Frequency + Slow'
      THEN 'URGENT: Optimize this query, major performance impact'  
    WHEN query_category = 'High Frequency'
      THEN 'Consider caching, connection pooling'
    ELSE 'Monitor'
  END as recommendation
FROM query_performance
WHERE query_category != 'Normal'

UNION ALL

SELECT 
  'Index Health' as analysis_type,
  indexname as item_name,
  index_health as status, 
  selectivity_percent as avg_time_ms,
  idx_scan as frequency,
  CASE 
    WHEN index_health = 'Unused' 
      THEN 'Consider dropping this index'
    WHEN index_health = 'Underused Large Index'
      THEN 'Evaluate if this large index is necessary'
    WHEN index_health = 'Low Selectivity'
      THEN 'Index may need tuning or composite columns'
    ELSE 'Index is healthy'
  END as recommendation
FROM index_efficiency
WHERE index_health != 'Healthy'
ORDER BY analysis_type, frequency DESC;
```

**🎯 Принципы enterprise-запросов:**
- **Параметризация**: Всегда используй `$1, $2` для пользовательского ввода
- **Ограничения**: LIMIT на все большие результаты
- **Мониторинг**: Встроенная аналитика производительности  
- **Обработка ошибок**: COALESCE для NULL значений
- **Масштабируемость**: Пагинация, индексы, оптимизированные JOIN'ы

**🚨 Critical Red Flags в Production:**
- Запросы без LIMIT на больших таблицах
- String concatenation вместо параметризации  
- Отсутствие мониторинга производительности
- Nested loops на миллионных таблицах без индексов
- Транзакции без timeout'ов

</details>

## 🎨 Mermaid: SQL Query Complexity Progression

```mermaid
flowchart TD
    A[🎯 Level 1: Simple Queries] --> B[📊 Basic SELECT + WHERE + ORDER BY]
    B --> C[🔢 Simple aggregations]
    
    C --> D[🎯 Level 2: Grouping & Aggregations]  
    D --> E[📊 GROUP BY + HAVING]
    E --> F[🧮 Complex aggregations + FILTER]
    
    F --> G[🎯 Level 3: JOIN Operations]
    G --> H[🔗 INNER/LEFT/RIGHT JOINs]
    H --> I[🎭 Self JOINs + Multiple JOINs]
    
    I --> J[🎯 Level 4: Window Functions]
    J --> K[🎨 RANK/ROW_NUMBER + LAG/LEAD] 
    K --> L[📈 Moving averages + Percentiles]
    
    L --> M[🎯 Level 5: CTEs & Subqueries]
    M --> N[🧠 Single + Multiple CTEs]
    N --> O[🔄 Recursive CTEs]
    
    O --> P[🎯 Level 6: Advanced Patterns]
    P --> Q[🚀 Full-text search + ML scoring]
    Q --> R[⚡ Performance monitoring]
    
    style A fill:#e8f5e8
    style D fill:#e1f5fe  
    style G fill:#f3e5f5
    style J fill:#fff3e0
    style M fill:#fce4ec
    style P fill:#ffebee
```

## 🎯 Summary

Теперь у тебя есть **полная прогрессия SQL мастерства**! 🔥

**Ключевые принципы построения сложных запросов:**

1. **🎯 Начинай просто** - сначала базовый SELECT, потом добавляй сложность
2. **📊 Разбивай на этапы** - используй CTE для декомпозиции логики  
3. **🔍 Тестируй пошагово** - проверяй каждый уровень отдельно
4. **⚡ Думай о производительности** - индексы, LIMIT, параметризация
5. **🧠 Структурируй код** - комментарии, отступы, логические блоки
6. **🚨 Предотвращай ошибки** - проверка NULL, валидация данных

**От простого к сложному:**
- **Level 1-2**: Основы для junior разработчиков
- **Level 3-4**: Middle уровень для backend систем  
- **Level 5-6**: Senior уровень для ML Platform и enterprise

# 🏗️ PostgreSQL: Creating Database & Tables from Scratch

<img width="800" height="800" alt="Image" src="https://github.com/user-attachments/assets/dacba4a8-a9eb-4e44-8443-5e74d114f49b" />

## 📖 Overview
Полное руководство по созданию PostgreSQL базы данных с нуля и управлению таблицами! 🚀 Для Backend/AI Engineer это фундаментальные навыки для построения production-ready систем. От простейшей схемы до enterprise архитектуры с партиционированием и оптимизацией.

## 🎯 Level 1: Database Creation - Основы

<details>
<summary>🏗️ <strong>Создание базы данных - первые шаги</strong></summary>

```sql
-- 🎯 Простое создание базы данных
CREATE DATABASE ml_platform;
-- 💡 Создает БД с настройками по умолчанию

-- 🔧 Создание с дополнительными параметрами
CREATE DATABASE ml_platform_prod
WITH 
  OWNER = ml_admin              -- владелец БД
  ENCODING = 'UTF8'             -- кодировка (поддержка всех языков)
  LC_COLLATE = 'en_US.UTF-8'    -- правила сортировки
  LC_CTYPE = 'en_US.UTF-8'      -- классификация символов
  TABLESPACE = pg_default       -- пространство таблиц
  CONNECTION LIMIT = 100;       -- максимум подключений
-- 🎯 Production-ready настройки для enterprise систем

-- 📊 Создание БД из шаблона
CREATE DATABASE ml_platform_test 
WITH TEMPLATE ml_platform_prod;
-- 💡 Копирует структуру и настройки существующей БД

-- 🔍 Информация о созданных базах данных
SELECT 
  datname as database_name,
  datowner::regrole as owner,
  encoding,
  datcollate as collation,
  datctype as character_classification,
  datconnlimit as connection_limit
FROM pg_database 
WHERE datname NOT IN ('template0', 'template1', 'postgres');
-- 📈 Показывает все пользовательские БД и их настройки

-- 🗑️ Удаление базы данных (осторожно!)
DROP DATABASE IF EXISTS ml_platform_test;
-- 🚨 Red Flag: Нельзя отменить! Всегда делай бэкап перед удалением

-- 🔄 Подключение к созданной БД
\c ml_platform
-- 💡 psql команда для переключения на другую БД
```

**🎯 Рекомендации для production БД:**
- Всегда указывай `OWNER` для контроля доступа
- Используй `UTF8` encoding для международной поддержки  
- Устанавливай `CONNECTION LIMIT` для контроля ресурсов
- Создавай отдельные БД для dev/test/prod окружений

**🚨 Red Flags при создании БД:**
- Создание БД без указания owner'а (будет текущий пользователь)
- Использование устаревших кодировок (не UTF8)
- Отсутствие ограничений подключений в production
- Создание production БД под суперпользователем postgres

</details>

## 🏢 Level 2: Basic Table Creation - Простые таблицы

<details>
<summary>📋 <strong>CREATE TABLE - от простого к сложному</strong></summary>

```sql
-- 🎯 Простейшая таблица пользователей
CREATE TABLE users (
  user_id SERIAL PRIMARY KEY,        -- автоинкремент + первичный ключ
  email VARCHAR(255) NOT NULL,       -- обязательное поле
  created_at TIMESTAMP DEFAULT NOW() -- значение по умолчанию
);
-- 💡 Минимальная структура с основными constraint'ами

-- 📊 Расширенная таблица пользователей  
CREATE TABLE users_extended (
  user_id SERIAL PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,           -- уникальное поле
  username VARCHAR(50) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  first_name VARCHAR(100),
  last_name VARCHAR(100),
  age INTEGER CHECK (age >= 13 AND age <= 120), -- валидация возраста
  country_code CHAR(2),                         -- ISO код страны
  is_active BOOLEAN DEFAULT true,               -- статус аккаунта
  email_verified BOOLEAN DEFAULT false,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  last_login TIMESTAMP WITH TIME ZONE
);
-- 🎯 Production-ready структура с валидацией и метаданными

-- 🔗 Таблица профилей с внешним ключом
CREATE TABLE user_profiles (
  profile_id SERIAL PRIMARY KEY,
  user_id INTEGER NOT NULL,
  bio TEXT,                                     -- длинный текст
  avatar_url VARCHAR(500),
  date_of_birth DATE,
  phone VARCHAR(20),
  timezone VARCHAR(50) DEFAULT 'UTC',
  preferences JSONB,                            -- JSON данные с индексацией
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  -- 🔗 Внешний ключ с каскадным удалением
  FOREIGN KEY (user_id) REFERENCES users_extended(user_id) 
    ON DELETE CASCADE ON UPDATE CASCADE
);
-- 💡 Один профиль на пользователя, удаляется вместе с пользователем

-- 🏷️ Enum типы для категориальных данных
CREATE TYPE user_status AS ENUM ('active', 'inactive', 'suspended', 'deleted');
CREATE TYPE subscription_tier AS ENUM ('free', 'basic', 'premium', 'enterprise');

-- 💳 Таблица подписок с enum'ами
CREATE TABLE subscriptions (
  subscription_id SERIAL PRIMARY KEY,
  user_id INTEGER NOT NULL,
  tier subscription_tier NOT NULL DEFAULT 'free',
  status user_status NOT NULL DEFAULT 'active',
  price DECIMAL(10,2) CHECK (price >= 0),       -- точная арифметика для денег
  billing_cycle INTEGER CHECK (billing_cycle IN (1, 3, 6, 12)), -- месяцы
  starts_at TIMESTAMP WITH TIME ZONE NOT NULL,
  expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
  auto_renewal BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  -- 🔗 Связь с пользователем
  FOREIGN KEY (user_id) REFERENCES users_extended(user_id) ON DELETE CASCADE,
  
  -- ✅ Проверка логичности дат
  CHECK (expires_at > starts_at),
  -- 🎯 Уникальность активной подписки на пользователя
  UNIQUE (user_id) WHERE status = 'active'
);

-- 🤖 Таблица для ML моделей и экспериментов
CREATE TABLE ml_models (
  model_id UUID PRIMARY KEY DEFAULT gen_random_uuid(), -- UUID вместо SERIAL
  model_name VARCHAR(100) NOT NULL,
  model_version VARCHAR(20) NOT NULL,
  model_type VARCHAR(50) NOT NULL,              -- 'classification', 'regression', etc.
  framework VARCHAR(30),                        -- 'scikit-learn', 'pytorch', etc.
  hyperparameters JSONB,                        -- конфигурация модели
  training_dataset_path VARCHAR(500),
  model_artifact_path VARCHAR(500),
  -- 📊 Метрики производительности
  accuracy DECIMAL(5,4) CHECK (accuracy >= 0 AND accuracy <= 1),
  precision_score DECIMAL(5,4) CHECK (precision_score >= 0 AND precision_score <= 1),
  recall_score DECIMAL(5,4) CHECK (recall_score >= 0 AND recall_score <= 1),
  f1_score DECIMAL(5,4) CHECK (f1_score >= 0 AND f1_score <= 1),
  -- 🏷️ Статус и метаданные
  status VARCHAR(20) DEFAULT 'training' 
    CHECK (status IN ('training', 'completed', 'deployed', 'deprecated')),
  trained_by INTEGER NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  deployed_at TIMESTAMP WITH TIME ZONE,
  
  -- 🔗 Связь с создателем модели
  FOREIGN KEY (trained_by) REFERENCES users_extended(user_id),
  -- 🎯 Уникальность версии модели
  UNIQUE (model_name, model_version)
);

-- 📈 Таблица предсказаний с партиционированием по времени
CREATE TABLE predictions (
  prediction_id BIGSERIAL,
  model_id UUID NOT NULL,
  user_id INTEGER,                              -- может быть NULL для batch предсказаний
  input_features JSONB NOT NULL,               -- входные признаки
  prediction_result JSONB NOT NULL,            -- результат предсказания
  confidence_score DECIMAL(5,4) CHECK (confidence_score >= 0 AND confidence_score <= 1),
  processing_time_ms INTEGER CHECK (processing_time_ms >= 0),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW() NOT NULL,
  
  -- 🔗 Внешние ключи
  FOREIGN KEY (model_id) REFERENCES ml_models(model_id),
  FOREIGN KEY (user_id) REFERENCES users_extended(user_id) ON DELETE SET NULL,
  
  -- 📅 Первичный ключ включает время для партиционирования
  PRIMARY KEY (prediction_id, created_at)
) PARTITION BY RANGE (created_at);
-- 🎯 Партиционирование по времени для больших объемов данных
```

**🎯 Ключевые принципы создания таблиц:**
- **PRIMARY KEY** на каждой таблице (SERIAL или UUID)
- **NOT NULL** для обязательных полей
- **CHECK constraints** для валидации данных
- **FOREIGN KEY** для ссылочной целостности
- **DEFAULT** значения для автоматического заполнения

**🚨 Red Flags в CREATE TABLE:**
- Таблицы без первичного ключа
- VARCHAR без ограничения длины
- Отсутствие NOT NULL на критичных полях  
- Использование TEXT вместо VARCHAR для коротких строк
- Игнорирование временных зон (TIMESTAMP vs TIMESTAMP WITH TIME ZONE)

</details>

## 🔧 Level 3: Table Modifications - ALTER TABLE

<details>
<summary>🔧 <strong>Изменение структуры таблиц - безопасные модификации</strong></summary>

```sql
-- 📊 Добавление новых колонок
ALTER TABLE users_extended 
ADD COLUMN middle_name VARCHAR(100);
-- 💡 Добавляет колонку в конец таблицы

-- 🔄 Добавление колонки с DEFAULT (безопасно для больших таблиц)
ALTER TABLE users_extended 
ADD COLUMN account_status VARCHAR(20) DEFAULT 'active' NOT NULL;
-- 🎯 DEFAULT позволяет избежать блокировки при добавлении NOT NULL

-- 🗑️ Удаление колонки (осторожно!)
ALTER TABLE users_extended 
DROP COLUMN middle_name;
-- 🚨 Red Flag: Данные будут потеряны навсегда!

-- 🔄 Изменение типа данных (может быть опасно)
ALTER TABLE users_extended 
ALTER COLUMN username TYPE VARCHAR(100);
-- ✅ Безопасно: увеличение размера VARCHAR

-- ⚠️ Потенциально опасное изменение типа
ALTER TABLE users_extended 
ALTER COLUMN age TYPE SMALLINT USING age::SMALLINT;
-- 🎯 USING clause для контроля преобразования типов

-- 🔒 Добавление и удаление constraint'ов
-- Добавление проверки
ALTER TABLE users_extended 
ADD CONSTRAINT check_email_format 
  CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$');
-- 📧 Валидация формата email через регулярное выражение

-- Добавление уникального ограничения
ALTER TABLE users_extended 
ADD CONSTRAINT unique_username UNIQUE (username);
-- 🎯 Гарантирует уникальность username'ов

-- Удаление constraint'а
ALTER TABLE users_extended 
DROP CONSTRAINT check_email_format;
-- 💡 Удаляет ранее созданное ограничение

-- 🔗 Работа с внешними ключами
-- Добавление FK с отложенной проверкой
ALTER TABLE user_profiles 
ADD CONSTRAINT fk_user_profiles_user_id 
  FOREIGN KEY (user_id) REFERENCES users_extended(user_id) 
  ON DELETE CASCADE ON UPDATE CASCADE 
  DEFERRABLE INITIALLY DEFERRED;
-- 🎯 DEFERRABLE позволяет отложить проверку до конца транзакции

-- 🏷️ Переименование элементов
-- Переименование таблицы
ALTER TABLE users_extended RENAME TO users;

-- Переименование колонки  
ALTER TABLE users RENAME COLUMN username TO user_name;

-- Переименование constraint'а
ALTER TABLE users RENAME CONSTRAINT unique_username TO unique_user_name;

-- 🔄 Изменение DEFAULT значений
ALTER TABLE users ALTER COLUMN is_active SET DEFAULT false;
-- Установка нового значения по умолчанию

ALTER TABLE users ALTER COLUMN is_active DROP DEFAULT;
-- Удаление значения по умолчанию

-- 📊 Сложные модификации с транзакциями
BEGIN;
  -- 🔄 Добавляем новую колонку
  ALTER TABLE ml_models 
  ADD COLUMN model_size_mb INTEGER;
  
  -- 📈 Заполняем данными на основе существующих
  UPDATE ml_models 
  SET model_size_mb = 
    CASE 
      WHEN model_type = 'deep_learning' THEN 150
      WHEN model_type = 'tree_based' THEN 50
      ELSE 25
    END;
  
  -- ✅ Делаем колонку обязательной
  ALTER TABLE ml_models 
  ALTER COLUMN model_size_mb SET NOT NULL;
  
  -- 🎯 Добавляем валидацию
  ALTER TABLE ml_models 
  ADD CONSTRAINT check_model_size 
    CHECK (model_size_mb > 0 AND model_size_mb < 10000);
COMMIT;
-- 💡 Все изменения в одной транзакции - либо все, либо ничего

-- 🚀 Production-safe колонка добавление (для больших таблиц)
-- Шаг 1: Добавляем nullable колонку
ALTER TABLE predictions 
ADD COLUMN batch_id UUID;

-- Шаг 2: Заполняем данными небольшими порциями (в отдельных транзакциях)
UPDATE predictions 
SET batch_id = gen_random_uuid() 
WHERE batch_id IS NULL 
  AND prediction_id BETWEEN 1 AND 10000;
-- Повторяем для всех диапазонов...

-- Шаг 3: Делаем колонку NOT NULL после заполнения всех данных
ALTER TABLE predictions 
ALTER COLUMN batch_id SET NOT NULL;

-- 🔍 Добавление индексов (отдельно от CREATE TABLE)
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_created_at ON users(created_at);
CREATE INDEX idx_users_active_users ON users(is_active) WHERE is_active = true;
-- 🎯 Partial index только для активных пользователей

-- 🤖 Специальные индексы для AI/ML данных
CREATE INDEX idx_ml_models_status_type ON ml_models(status, model_type);
CREATE INDEX idx_predictions_model_created 
  ON predictions(model_id, created_at) 
  WHERE confidence_score >= 0.8;
-- 📊 Составной индекс с условием для high-confidence предсказаний

-- 🧹 VACUUM и ANALYZE после больших изменений
VACUUM ANALYZE users;
VACUUM ANALYZE ml_models;
-- 💡 Обновляет статистику планировщика и очищает мертвые строки
```

**🎯 Best Practices для ALTER TABLE:**
- **Тестируй на копии данных** перед production изменениями
- **Используй транзакции** для множественных связанных изменений
- **Добавляй DEFAULT** при создании NOT NULL колонок
- **Планируй downtime** для критичных изменений типов данных
- **Создавай индексы CONCURRENTLY** на live системах

**🚨 Опасные операции ALTER TABLE:**
- DROP COLUMN без резервной копии
- Уменьшение размера VARCHAR на колонках с данными
- Изменение типов данных без USING clause
- Добавление NOT NULL без DEFAULT на больших таблицах
- Переименование активно используемых колонок без координации

</details>

## 🏗️ Level 4: Advanced Table Features - Продвинутые возможности

<details>
<summary>🚀 <strong>Продвинутые техники создания таблиц</strong></summary>

```sql
-- 🎯 Создание таблиц с наследованием
CREATE TABLE events (
  event_id BIGSERIAL PRIMARY KEY,
  event_type VARCHAR(50) NOT NULL,
  user_id INTEGER,
  event_data JSONB,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 🏷️ Дочерние таблицы наследуют структуру
CREATE TABLE user_events (
  session_id UUID,
  page_url VARCHAR(500)
) INHERITS (events);

CREATE TABLE system_events (
  severity_level INTEGER CHECK (severity_level BETWEEN 1 AND 5),
  component VARCHAR(100)
) INHERITS (events);
-- 💡 Наследование позволяет специализировать таблицы

-- 📅 Партиционирование по времени (современный подход)
CREATE TABLE analytics_events (
  event_id BIGSERIAL,
  event_type VARCHAR(50) NOT NULL,
  user_id INTEGER,
  event_data JSONB,
  revenue DECIMAL(10,2),
  created_at TIMESTAMP WITH TIME ZONE NOT NULL,
  PRIMARY KEY (event_id, created_at)
) PARTITION BY RANGE (created_at);

-- 📊 Создание партиций
CREATE TABLE analytics_events_2024_01 
PARTITION OF analytics_events 
FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE analytics_events_2024_02 
PARTITION OF analytics_events 
FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');

-- 🔄 Автоматическое создание партиций (через расширение pg_partman)
-- CREATE EXTENSION pg_partman;
-- SELECT partman.create_parent(
--   p_parent_table => 'analytics_events',
--   p_control => 'created_at',
--   p_type => 'range',
--   p_interval => 'monthly'
-- );

-- 🗂️ Партиционирование по хешу для равномерного распределения
CREATE TABLE user_interactions (
  interaction_id BIGSERIAL,
  user_id INTEGER NOT NULL,
  action_type VARCHAR(50),
  target_id INTEGER,
  metadata JSONB,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  PRIMARY KEY (interaction_id, user_id)
) PARTITION BY HASH (user_id);

-- Создание хеш-партиций
CREATE TABLE user_interactions_hash_0 
PARTITION OF user_interactions 
FOR VALUES WITH (modulus 4, remainder 0);

CREATE TABLE user_interactions_hash_1 
PARTITION OF user_interactions 
FOR VALUES WITH (modulus 4, remainder 1);

CREATE TABLE user_interactions_hash_2 
PARTITION OF user_interactions 
FOR VALUES WITH (modulus 4, remainder 2);

CREATE TABLE user_interactions_hash_3 
PARTITION OF user_interactions 
FOR VALUES WITH (modulus 4, remainder 3);

-- 🎯 Таблица с полнотекстовым поиском
CREATE TABLE documents (
  document_id SERIAL PRIMARY KEY,
  title VARCHAR(200) NOT NULL,
  content TEXT NOT NULL,
  author_id INTEGER,
  category VARCHAR(50),
  -- 🔍 Поле для полнотекстового поиска
  search_vector tsvector GENERATED ALWAYS AS (
    to_tsvector('english', title || ' ' || content)
  ) STORED,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  FOREIGN KEY (author_id) REFERENCES users(user_id)
);

-- Индекс для полнотекстового поиска
CREATE INDEX idx_documents_search ON documents USING GIN (search_vector);

-- 🤖 Таблица для векторного поиска (pgvector extension)
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE embeddings (
  embedding_id SERIAL PRIMARY KEY,
  content_id INTEGER NOT NULL,
  content_type VARCHAR(50) NOT NULL, -- 'product', 'article', 'user_profile'
  embedding vector(768),              -- 768-dimensional embedding
  model_name VARCHAR(100) DEFAULT 'text-embedding-ada-002',
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  -- 🎯 Составной индекс для фильтрации по типу + векторный поиск
  UNIQUE (content_id, content_type)
);

-- Векторные индексы для similarity search
CREATE INDEX idx_embeddings_cosine ON embeddings 
  USING ivfflat (embedding vector_cosine_ops) 
  WITH (lists = 100);

CREATE INDEX idx_embeddings_l2 ON embeddings 
  USING ivfflat (embedding vector_l2_ops) 
  WITH (lists = 100);

-- 📊 Таблица с JSON колонками и индексами
CREATE TABLE user_analytics (
  analytics_id SERIAL PRIMARY KEY,
  user_id INTEGER NOT NULL,
  -- 📈 Структурированные метрики
  session_metrics JSONB NOT NULL DEFAULT '{}',
  behavioral_features JSONB NOT NULL DEFAULT '{}',
  ml_predictions JSONB NOT NULL DEFAULT '{}',
  last_updated TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
);

-- 🔍 GIN индексы для эффективного поиска по JSON
CREATE INDEX idx_user_analytics_session_gin 
  ON user_analytics USING GIN (session_metrics);

-- 🎯 Частичные индексы для специфических JSON запросов
CREATE INDEX idx_user_analytics_high_engagement 
  ON user_analytics (user_id) 
  WHERE (session_metrics->>'engagement_score')::numeric > 80;

-- 📈 Индекс на JSON path для прямых запросов
CREATE INDEX idx_user_analytics_churn_risk 
  ON user_analytics ((ml_predictions->>'churn_probability')) 
  WHERE (ml_predictions->>'churn_probability')::numeric > 0.7;

-- 🏷️ Таблица с enum и массивами
CREATE TYPE skill_level AS ENUM ('beginner', 'intermediate', 'advanced', 'expert');

CREATE TABLE user_skills (
  skill_id SERIAL PRIMARY KEY,
  user_id INTEGER NOT NULL,
  programming_languages VARCHAR(50)[],      -- массив языков программирования
  frameworks VARCHAR(100)[],                -- массив фреймворков  
  skill_levels skill_level[] NOT NULL,      -- массив уровней навыков
  certifications JSONB DEFAULT '[]',        -- JSON массив сертификатов
  last_assessment_date DATE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE,
  
  -- ✅ Проверка соответствия размеров массивов
  CHECK (array_length(programming_languages, 1) = array_length(skill_levels, 1))
);

-- 🔍 Индексы для работы с массивами
CREATE INDEX idx_user_skills_languages 
  ON user_skills USING GIN (programming_languages);

-- 🎯 Временная таблица для ETL процессов
CREATE TEMPORARY TABLE temp_data_import (
  raw_id INTEGER,
  raw_data JSONB,
  processed BOOLEAN DEFAULT false,
  import_timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
-- 💡 TEMPORARY таблицы автоматически удаляются после сессии

-- 🔄 Таблица с автоматическим обновлением timestamp'а
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ language 'plpgsql';

-- Применяем trigger к таблице
CREATE TRIGGER update_users_updated_at 
  BEFORE UPDATE ON users 
  FOR EACH ROW 
  EXECUTE FUNCTION update_updated_at_column();
-- 🎯 Автоматически обновляет updated_at при любом UPDATE

-- 📊 Materialized View для сложной аналитики
CREATE MATERIALIZED VIEW user_engagement_summary AS
SELECT 
  u.user_id,
  u.email,
  COUNT(ui.interaction_id) as total_interactions,
  AVG((ua.session_metrics->>'engagement_score')::numeric) as avg_engagement,
  MAX(ui.created_at) as last_activity,
  CASE 
    WHEN AVG((ua.session_metrics->>'engagement_score')::numeric) >= 80 THEN 'High'
    WHEN AVG((ua.session_metrics->>'engagement_score')::numeric) >= 50 THEN 'Medium'
    ELSE 'Low'
  END as engagement_tier
FROM users u
LEFT JOIN user_interactions ui ON u.user_id = ui.user_id
LEFT JOIN user_analytics ua ON u.user_id = ua.user_id  
WHERE u.is_active = true
GROUP BY u.user_id, u.email;

-- Индекс на Materialized View
CREATE INDEX idx_engagement_summary_tier ON user_engagement_summary(engagement_tier);

-- 🔄 Обновление Materialized View
REFRESH MATERIALIZED VIEW CONCURRENTLY user_engagement_summary;
-- 💡 CONCURRENTLY позволяет обновлять без блокировки читающих запросов
```

**🎯 Когда использовать продвинутые техники:**
- **Партиционирование**: Таблицы >100GB, время-ориентированные данные
- **Наследование**: Схожие таблицы с разными специализациями
- **JSONB**: Полуструктурированные данные, flexible schema
- **Vectors**: AI/ML embeddings, similarity search
- **Materialized Views**: Сложная аналитика с предвычислениями

**🚨 Red Flags в продвинутых техниках:**
- Преждевременное партиционирование маленьких таблиц
- Использование наследования вместо простых внешних ключей
- JSONB для данных, которые лучше подходят для реляционной модели
- Materialized Views без стратегии обновления
- Векторные индексы на малых данных (< 10K записей)

</details>

## 🔧 Level 5: Table Maintenance & Optimization

<details>
<summary>⚡ <strong>Обслуживание и оптимизация таблиц</strong></summary>

```sql
-- 📊 Анализ размеров таблиц и индексов
SELECT 
  schemaname,
  tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as total_size,
  pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) as table_size,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename) - 
                 pg_relation_size(schemaname||'.'||tablename)) as index_size,
  -- 📈 Процент индексов от общего размера
  ROUND(
    (pg_total_relation_size(schemaname||'.'||tablename) - 
     pg_relation_size(schemaname||'.'||tablename)) * 100.0 / 
    pg_total_relation_size(schemaname||'.'||tablename), 2
  ) as index_ratio_percent
FROM pg_tables 
WHERE schemaname NOT IN ('information_schema', 'pg_catalog')
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
LIMIT 20;

-- 🧹 VACUUM и ANALYZE операции
-- Обычный VACUUM (освобождает место для переиспользования)
VACUUM users;

-- VACUUM FULL (возвращает место операционной системе, блокирует таблицу)
VACUUM FULL users;  -- ⚠️ Осторожно: полная блокировка!

-- ANALYZE обновляет статистику планировщика
ANALYZE users;

-- Комбинированная операция
VACUUM ANALYZE users;

-- 📊 Автоматический VACUUM (настройка на уровне таблицы)
ALTER TABLE user_interactions SET (
  autovacuum_vacuum_scale_factor = 0.1,      -- VACUUM при 10% мертвых строк
  autovacuum_analyze_scale_factor = 0.05,    -- ANALYZE при 5% изменений
  autovacuum_vacuum_cost_limit = 2000,       -- лимит стоимости операций
  autovacuum_naptime = 60                    -- проверка каждые 60 секунд
);

-- 🔍 Мониторинг статистики VACUUM
SELECT 
  schemaname,
  tablename,
  n_live_tup as live_rows,
  n_dead_tup as dead_rows,
  CASE 
    WHEN n_live_tup > 0 
    THEN ROUND(n_dead_tup * 100.0 / (n_live_tup + n_dead_tup), 2)
    ELSE 0 
  END as dead_row_percent,
  last_vacuum,
  last_autovacuum,
  last_analyze,
  last_autoanalyze,
  vacuum_count,
  autovacuum_count
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC;

-- 📈 Реиндексация для восстановления производительности
-- Проверка bloat индексов
SELECT 
  schemaname,
  tablename,
  indexname,
  pg_size_pretty(pg_relation_size(indexname::regclass)) as index_size,
  idx_scan as times_used,
  idx_tup_read as tuples_read,
  idx_tup_fetch as tuples_fetched
FROM pg_stat_user_indexes
ORDER BY pg_relation_size(indexname::regclass) DESC;

-- Реиндексация конкретного индекса
REINDEX INDEX idx_users_email;

-- Реиндексация всей таблицы  
REINDEX TABLE users;

-- 🔄 REINDEX CONCURRENTLY (PostgreSQL 12+, не блокирует)
REINDEX INDEX CONCURRENTLY idx_users_email;

-- 🎯 Создание нового индекса взамен старого без downtime
CREATE INDEX CONCURRENTLY idx_users_email_new ON users(email);
-- Проверяем что индекс создался успешно
DROP INDEX idx_users_email;
ALTER INDEX idx_users_email_new RENAME TO idx_users_email;

-- 📊 Статистика использования колонок (для выявления неиспользуемых)
SELECT 
  schemaname,
  tablename,
  attname as column_name,
  n_distinct,      -- количество уникальных значений
  correlation,     -- корреляция с физическим порядком
  most_common_vals[1:3] as top_values,  -- самые частые значения
  most_common_freqs[1:3] as frequencies -- их частоты
FROM pg_stats
WHERE schemaname NOT IN ('information_schema', 'pg_catalog')
  AND tablename = 'users'
ORDER BY n_distinct DESC;

-- 🔧 Управление constraint'ами для производительности
-- Отключение FK проверок для bulk операций
ALTER TABLE user_profiles DISABLE TRIGGER ALL;
-- Массовая вставка данных...
ALTER TABLE user_profiles ENABLE TRIGGER ALL;

-- 🎯 Проверка целостности после bulk операций
-- Проверка FK constraint'ов
ALTER TABLE user_profiles 
  VALIDATE CONSTRAINT fk_user_profiles_user_id;

-- 📈 Сбор расширенной статистики для сложных запросов
-- Статистика по множественным колонкам
CREATE STATISTICS stats_users_country_age 
ON country_code, age 
FROM users;

-- Статистика по выражениям
CREATE STATISTICS stats_users_full_name 
ON (first_name || ' ' || last_name)
FROM users;

-- Обновление статистики
ANALYZE users;

-- 🔍 Monitoring долгих операций DDL
SELECT 
  pid,
  now() - pg_stat_activity.query_start as duration,
  query,
  state
FROM pg_stat_activity 
WHERE state != 'idle'
  AND query ILIKE '%ALTER%'
     OR query ILIKE '%CREATE INDEX%'
     OR query ILIKE '%REINDEX%'
ORDER BY duration DESC;

-- 🚨 Принудительная остановка долгой операции (осторожно!)
-- SELECT pg_cancel_backend(pid);  -- мягкая остановка
-- SELECT pg_terminate_backend(pid);  -- принудительная остановка

-- 📊 Checkpoint и WAL мониторинг
SELECT 
  checkpoint_timed,
  checkpoint_req,
  checkpoint_write_time,
  checkpoint_sync_time,
  buffers_checkpoint,
  buffers_clean,
  maxwritten_clean
FROM pg_stat_bgwriter;

-- 🔄 Ручной checkpoint (для консистентного бэкапа)
CHECKPOINT;

-- 📈 Анализ табличных пространств
SELECT 
  spcname as tablespace_name,
  pg_tablespace_location(oid) as location,
  pg_size_pretty(pg_tablespace_size(spcname)) as size
FROM pg_tablespace;

-- 🎯 Создание отдельного табличного пространства для больших данных
-- CREATE TABLESPACE large_data 
--   LOCATION '/var/lib/postgresql/large_data';

-- CREATE TABLE big_analytics_data (
--   ...
-- ) TABLESPACE large_data;

-- 🧹 Автоматизация maintenance задач
-- Пример скрипта для регулярного обслуживания
DO $
DECLARE
  table_record RECORD;
  dead_ratio NUMERIC;
BEGIN
  FOR table_record IN 
    SELECT schemaname, tablename 
    FROM pg_tables 
    WHERE schemaname = 'public'
  LOOP
    -- Получаем процент мертвых строк
    SELECT 
      CASE 
        WHEN n_live_tup > 0 
        THEN n_dead_tup * 100.0 / (n_live_tup + n_dead_tup)
        ELSE 0 
      END 
    INTO dead_ratio
    FROM pg_stat_user_tables 
    WHERE schemaname = table_record.schemaname 
      AND tablename = table_record.tablename;
    
    -- VACUUM если больше 20% мертвых строк
    IF dead_ratio > 20 THEN
      EXECUTE format('VACUUM ANALYZE %I.%I', 
                     table_record.schemaname, 
                     table_record.tablename);
      RAISE NOTICE 'VACUUM performed on %.% (dead ratio: %)', 
                   table_record.schemaname,
                   table_record.tablename, 
                   dead_ratio;
    END IF;
  END LOOP;
END $;
```

**🎯 Регулярные maintenance задачи:**
- **Еженедельно**: VACUUM ANALYZE на активных таблицах
- **Ежемесячно**: REINDEX на сильно изменяемых индексах
- **По необходимости**: Сбор расширенной статистики
- **Мониторинг**: Размеры таблиц, bloat индексов, мертвые строки

**🚨 Red Flags в maintenance:**
- VACUUM FULL на production без maintenance window
- REINDEX без CONCURRENTLY на активных системах
- Игнорирование autovacuum warnings
- Отсутствие мониторинга размеров и производительности
- Ручное отключение constraint'ов без последующей валидации

</details>

## 🎨 Mermaid: Database Creation Workflow

```mermaid
flowchart TD
    A[🎯 Database Planning] --> B[📋 Requirements Analysis]
    B --> C[🏗️ CREATE DATABASE]
    
    C --> D[🔧 Basic Tables]
    D --> E[📊 Primary Keys + Constraints]
    E --> F[🔗 Foreign Keys + Relationships]
    
    F --> G[📈 Indexes Creation]
    G --> H[🎯 Data Population]
    
    H --> I{Production Ready?}
    I -->|No| J[🔧 ALTER TABLE Modifications]
    J --> K[📊 Test Performance]
    K --> I
    
    I -->|Yes| L[🚀 Advanced Features]
    L --> M[📅 Partitioning]
    L --> N[🔍 Full-text Search]  
    L --> O[🤖 Vector Extensions]
    
    M --> P[⚡ Maintenance Setup]
    N --> P
    O --> P
    
    P --> Q[🧹 VACUUM/ANALYZE]
    P --> R[📊 Monitoring]
    P --> S[🔄 Backup Strategy]
    
    Q --> T[✅ Production Database]
    R --> T
    S --> T
    
    style A fill:#e8f5e8
    style C fill:#e1f5fe
    style I fill:#fff3e0
    style L fill:#f3e5f5
    style T fill:#e8f5e8
```

## 🎯 Summary

Теперь у вас есть **полное руководство по созданию PostgreSQL баз данных**! 🚀

**Ключевые этапы разработки БД:**

### 📋 **Planning Phase**
- Анализ требований и data model
- Выбор типов данных и constraint'ов
- Планирование индексной стратегии

### 🏗️ **Implementation Phase**  
- CREATE DATABASE с правильными настройками
- CREATE TABLE с валидацией и связями
- Создание индексов для производительности

### 🔧 **Evolution Phase**
- ALTER TABLE для изменения структуры
- Добавление продвинутых возможностей
- Партиционирование и оптимизация

### ⚡ **Maintenance Phase**
- Регулярный VACUUM и ANALYZE
- Мониторинг размеров и производительности  
- Реиндексация и оптимизация

**Production-Ready принципы:**
- ✅ Всегда используй транзакции для связанных изменений
- ✅ Тестируй ALTER TABLE на копии данных
- ✅ Создавай индексы CONCURRENTLY на live системах
- ✅ Настраивай autovacuum для каждой таблицы
- ✅ Мониторь размеры, bloat и производительность

От простейших таблиц до enterprise-архитектуры с партиционированием - теперь вы готовы создавать **масштабируемые и производительные** PostgreSQL системы! 🎯✨

# 🔄 PostgreSQL UPSERT: INSERT + UPDATE в одном запросе

<img width="800" height="800" alt="Image" src="https://github.com/user-attachments/assets/283fe459-a2f7-491d-a975-66b2caeea658" />

## 📖 Overview
Полное руководство по UPSERT операциям в PostgreSQL! 🚀 UPSERT (INSERT + UPDATE) - критически важная техника для Backend/AI Engineers при работе с данными, которые могут как создаваться, так и обновляться. От простейших случаев до сложной логики с множественными conflict resolution стратегиями.

## 🎯 Level 1: Basic UPSERT - Основы ON CONFLICT

<details>
<summary>🔄 <strong>INSERT ... ON CONFLICT - базовый UPSERT</strong></summary>

```sql
-- 🎯 Простейший UPSERT для пользователей
INSERT INTO users (email, username, first_name, last_name)
VALUES ('john.doe@example.com', 'johndoe', 'John', 'Doe')
ON CONFLICT (email)           -- если email уже существует
DO UPDATE SET 
  username = EXCLUDED.username,      -- обновляем из новых данных
  first_name = EXCLUDED.first_name,  -- EXCLUDED = новые значения
  last_name = EXCLUDED.last_name,
  updated_at = NOW();               -- обновляем timestamp
-- 💡 Если email существует - обновляем, если нет - создаем запись

-- 🔍 UPSERT с условной логикой
INSERT INTO user_profiles (user_id, bio, location, website)
VALUES (12345, 'AI Engineer passionate about ML', 'San Francisco', 'https://johndoe.dev')
ON CONFLICT (user_id)
DO UPDATE SET 
  bio = CASE 
    WHEN EXCLUDED.bio IS NOT NULL AND LENGTH(EXCLUDED.bio) > LENGTH(user_profiles.bio)
    THEN EXCLUDED.bio           -- обновляем только если новое описание длиннее
    ELSE user_profiles.bio      -- иначе оставляем старое
  END,
  location = COALESCE(EXCLUDED.location, user_profiles.location),  -- обновляем если не NULL
  website = EXCLUDED.website,   -- всегда обновляем website
  updated_at = NOW();
-- 🎯 Умная логика обновления - сохраняем лучшие данные

-- 📊 UPSERT для счетчиков и метрик
INSERT INTO user_activity_counters (user_id, activity_date, page_views, clicks, time_spent_minutes)
VALUES (12345, CURRENT_DATE, 1, 1, 5)
ON CONFLICT (user_id, activity_date)    -- составной ключ конфликта
DO UPDATE SET 
  page_views = user_activity_counters.page_views + EXCLUDED.page_views,
  clicks = user_activity_counters.clicks + EXCLUDED.clicks,
  time_spent_minutes = user_activity_counters.time_spent_minutes + EXCLUDED.time_spent_minutes,
  updated_at = NOW();
-- 💡 Накапливаем метрики вместо перезаписи

-- 🏷️ UPSERT с использованием WHERE условия
INSERT INTO product_inventory (product_id, warehouse_id, quantity, reserved_quantity)
VALUES (67890, 1, 100, 0)
ON CONFLICT (product_id, warehouse_id)
DO UPDATE SET 
  quantity = EXCLUDED.quantity,
  updated_at = NOW()
WHERE product_inventory.quantity != EXCLUDED.quantity;  -- обновляем только при изменении
-- ⚡ Условие WHERE экономит ненужные обновления

-- 🚨 UPSERT с игнорированием конфликтов  
INSERT INTO user_login_attempts (user_id, login_date, attempt_count)
VALUES (12345, CURRENT_DATE, 1)
ON CONFLICT (user_id, login_date) 
DO NOTHING;                    -- просто игнорируем если запись существует
-- 💡 Полезно для предотвращения дублирования данных без обновления

-- 📈 UPSERT для временных рядов данных
INSERT INTO daily_metrics (metric_date, total_users, active_users, revenue)
VALUES ('2024-01-15', 15000, 8500, 125000.00)
ON CONFLICT (metric_date)
DO UPDATE SET 
  total_users = GREATEST(daily_metrics.total_users, EXCLUDED.total_users),    -- берем максимум
  active_users = GREATEST(daily_metrics.active_users, EXCLUDED.active_users),
  revenue = daily_metrics.revenue + EXCLUDED.revenue,  -- накапливаем доход
  calculation_version = COALESCE(daily_metrics.calculation_version, 0) + 1,   -- версионность
  updated_at = NOW();
-- 🎯 Сложная логика для агрегированных метрик

-- 🔄 UPSERT с RETURNING для получения результата
INSERT INTO user_sessions (user_id, session_token, expires_at, device_info)
VALUES (12345, 'abc123token', NOW() + INTERVAL '24 hours', '{"device": "mobile", "os": "iOS"}')
ON CONFLICT (user_id)          -- один активный сессион на пользователя
DO UPDATE SET 
  session_token = EXCLUDED.session_token,
  expires_at = EXCLUDED.expires_at,
  device_info = EXCLUDED.device_info,
  updated_at = NOW()
RETURNING 
  session_id,                  -- получаем ID сессии
  session_token,               -- новый токен
  CASE 
    WHEN xmax = 0 THEN 'inserted'    -- xmax = 0 означает INSERT
    ELSE 'updated'                   -- иначе был UPDATE  
  END as operation_type;
-- 💡 RETURNING показывает что именно произошло
```

**🎯 Основные принципы базового UPSERT:**
- **ON CONFLICT** указывает колонку/constraint для проверки конфликта
- **EXCLUDED** ссылается на новые значения из VALUES
- **DO UPDATE** выполняет обновление при конфликте
- **DO NOTHING** игнорирует конфликт без изменений
- **WHERE** в DO UPDATE добавляет условия для обновления

**🚨 Red Flags в базовом UPSERT:**
- ON CONFLICT без указания конкретной колонки/constraint
- Забытый SET updated_at при обновлении временных меток
- UPSERT без проверки NULL значений через COALESCE
- Использование DO NOTHING когда нужно логирование операций
- Отсутствие RETURNING когда нужно знать результат операции

</details>

## 📊 Level 2: Bulk UPSERT - Массовые операции

<details>
<summary>🚀 <strong>Массовый UPSERT для больших объемов данных</strong></summary>

```sql
-- 🎯 Bulk UPSERT с VALUES списком
INSERT INTO product_prices (product_id, price, currency, effective_date)
VALUES 
  (1001, 29.99, 'USD', '2024-01-15'),
  (1002, 45.50, 'USD', '2024-01-15'),
  (1003, 12.75, 'USD', '2024-01-15'),
  (1004, 67.25, 'USD', '2024-01-15'),
  (1001, 34.99, 'EUR', '2024-01-15')    -- тот же продукт, другая валюта
ON CONFLICT (product_id, currency, effective_date)
DO UPDATE SET 
  price = EXCLUDED.price,
  updated_at = NOW();
-- 💡 Обновляет существующие цены и создает новые за один запрос

-- 📈 UPSERT из SELECT запроса (ETL pattern)
INSERT INTO user_daily_stats (user_id, stat_date, sessions_count, total_time_minutes, pages_viewed)
SELECT 
  user_id,
  DATE(created_at) as stat_date,
  COUNT(DISTINCT session_id) as sessions_count,
  SUM(duration_minutes) as total_time_minutes,
  SUM(pages_viewed) as pages_viewed
FROM user_sessions 
WHERE DATE(created_at) = CURRENT_DATE - INTERVAL '1 day'  -- вчерашние данные
GROUP BY user_id, DATE(created_at)
ON CONFLICT (user_id, stat_date)
DO UPDATE SET 
  sessions_count = EXCLUDED.sessions_count,
  total_time_minutes = EXCLUDED.total_time_minutes,
  pages_viewed = EXCLUDED.pages_viewed,
  recalculated_at = NOW();
-- 🎯 Агрегируем сырые данные в дневную статистику

-- 🤖 ML model predictions bulk UPSERT
WITH new_predictions AS (
  SELECT 
    user_id,
    'churn_model_v2.1' as model_name,
    CURRENT_DATE as prediction_date,
    -- Упрощенная ML логика для примера
    CASE 
      WHEN last_login < NOW() - INTERVAL '30 days' THEN 0.8
      WHEN last_login < NOW() - INTERVAL '14 days' THEN 0.6
      WHEN session_count_30d = 0 THEN 0.7
      ELSE 0.2
    END as churn_probability,
    CASE 
      WHEN subscription_tier = 'premium' THEN predicted_ltv * 1.5
      ELSE predicted_ltv
    END as adjusted_ltv,
    NOW() as created_at
  FROM user_features_current  -- предвычисленная таблица с признаками
  WHERE is_active = true
)
INSERT INTO ml_predictions (user_id, model_name, prediction_date, churn_probability, predicted_ltv, created_at)
SELECT * FROM new_predictions
ON CONFLICT (user_id, model_name, prediction_date)
DO UPDATE SET 
  churn_probability = EXCLUDED.churn_probability,
  predicted_ltv = EXCLUDED.predicted_ltv,
  model_confidence = CASE 
    WHEN ABS(ml_predictions.churn_probability - EXCLUDED.churn_probability) < 0.1 
    THEN LEAST(ml_predictions.model_confidence + 0.1, 1.0)  -- увеличиваем уверенность
    ELSE GREATEST(ml_predictions.model_confidence - 0.1, 0.3) -- уменьшаем при больших изменениях
  END,
  updated_at = NOW();
-- 🧠 Batch обновление ML предсказаний с логикой уверенности

-- 📊 UPSERT для аналитических кубов
INSERT INTO sales_cube (date_key, product_category, region, channel, revenue, units_sold, unique_customers)
SELECT 
  DATE(order_date) as date_key,
  p.category as product_category,
  c.region,
  o.sales_channel as channel,
  SUM(oi.price * oi.quantity) as revenue,
  SUM(oi.quantity) as units_sold,
  COUNT(DISTINCT o.customer_id) as unique_customers
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
JOIN customers c ON o.customer_id = c.customer_id
WHERE DATE(o.order_date) = CURRENT_DATE - INTERVAL '1 day'  -- ETL для предыдущего дня
GROUP BY DATE(order_date), p.category, c.region, o.sales_channel
ON CONFLICT (date_key, product_category, region, channel)
DO UPDATE SET 
  revenue = EXCLUDED.revenue,
  units_sold = EXCLUDED.units_sold,
  unique_customers = EXCLUDED.unique_customers,
  -- 📈 Вычисляем дополнительные метрики при обновлении
  avg_order_value = EXCLUDED.revenue / NULLIF(EXCLUDED.unique_customers, 0),
  units_per_customer = EXCLUDED.units_sold::numeric / NULLIF(EXCLUDED.unique_customers, 0),
  last_updated = NOW();
-- 💡 ETL pipeline для бизнес-аналитики с автоматическими метриками

-- 🔄 Batch UPSERT с обработкой ошибок
DO $$
DECLARE
  batch_size INTEGER := 10000;
  offset_val INTEGER := 0;
  total_processed INTEGER := 0;
  error_count INTEGER := 0;
BEGIN
  LOOP
    BEGIN
      WITH batch_data AS (
        SELECT 
          user_id,
          feature_name,
          feature_value,
          CURRENT_DATE as calculation_date
        FROM temp_ml_features  -- временная таблица с обработанными данными
        LIMIT batch_size OFFSET offset_val
      )
      INSERT INTO user_features (user_id, feature_name, feature_value, calculation_date, created_at)
      SELECT 
        user_id,
        feature_name, 
        feature_value,
        calculation_date,
        NOW()
      FROM batch_data
      ON CONFLICT (user_id, feature_name, calculation_date)
      DO UPDATE SET 
        feature_value = EXCLUDED.feature_value,
        updated_at = NOW(),
        version_number = COALESCE(user_features.version_number, 0) + 1;
      
      -- Проверяем количество обработанных записей
      GET DIAGNOSTICS total_processed = ROW_COUNT;
      
      -- Если меньше чем batch_size, то закончили
      EXIT WHEN total_processed < batch_size;
      
      offset_val := offset_val + batch_size;
      
      -- Логирование прогресса
      RAISE NOTICE 'Processed % records, offset: %', total_processed, offset_val;
      
      -- Небольшая пауза между batch'ами для снижения нагрузки
      PERFORM pg_sleep(0.1);
      
    EXCEPTION WHEN OTHERS THEN
      error_count := error_count + 1;
      RAISE WARNING 'Error in batch starting at offset %: %', offset_val, SQLERRM;
      
      -- Прерываем если слишком много ошибок
      IF error_count > 5 THEN
        RAISE EXCEPTION 'Too many errors in batch processing';
      END IF;
      
      offset_val := offset_val + batch_size;
    END;
  END LOOP;
  
  RAISE NOTICE 'Batch UPSERT completed. Total errors: %', error_count;
END $$;

-- 📊 Мониторинг производительности bulk UPSERT
SELECT 
  schemaname,
  tablename,
  n_tup_ins as inserts,           -- количество вставок
  n_tup_upd as updates,           -- количество обновлений  
  n_tup_del as deletes,           -- количество удалений
  n_live_tup as live_rows,        -- живые строки
  n_dead_tup as dead_rows,        -- мертвые строки (нуждаются в VACUUM)
  last_vacuum,
  last_autovacuum
FROM pg_stat_user_tables 
WHERE tablename IN ('ml_predictions', 'user_features', 'sales_cube')
ORDER BY n_tup_upd DESC;

-- 🧹 VACUUM после массовых UPSERT операций
VACUUM ANALYZE ml_predictions;
VACUUM ANALYZE user_features;
-- 💡 Необходимо после больших bulk операций для обновления статистики
```

**🎯 Принципы эффективного bulk UPSERT:**
- **Batch processing** для контроля памяти и блокировок
- **Обработка ошибок** с логированием и retry логикой
- **Мониторинг прогресса** для длительных операций  
- **VACUUM ANALYZE** после массовых операций
- **Pause между batches** для снижения нагрузки на систему

**🚨 Red Flags в bulk UPSERT:**
- Массовые операции без batch'ей (может исчерпать память)
- Отсутствие обработки ошибок в длительных операциях
- Bulk UPSERT в prime time без координации
- Игнорирование deadlock'ов при параллельных операциях
- Отсутствие мониторинга прогресса для долгих процессов

</details>

## 🔗 Level 3: Advanced UPSERT Patterns - Сложные сценарии

<details>
<summary>🧠 <strong>Продвинутые паттерны UPSERT для enterprise систем</strong></summary>

```sql
-- 🎭 Multi-table UPSERT с транзакциями
BEGIN;
  -- UPSERT пользователя
  INSERT INTO users (email, username, first_name, last_name)
  VALUES ('alice@example.com', 'alice', 'Alice', 'Johnson')
  ON CONFLICT (email)
  DO UPDATE SET 
    username = EXCLUDED.username,
    first_name = EXCLUDED.first_name,
    last_name = EXCLUDED.last_name,
    updated_at = NOW()
  RETURNING user_id INTO @user_id;  -- сохраняем ID для следующих операций

  -- UPSERT профиля с полученным user_id
  INSERT INTO user_profiles (user_id, bio, location, skills)
  VALUES (@user_id, 'Senior Backend Engineer', 'New York', ARRAY['Python', 'PostgreSQL', 'Docker'])
  ON CONFLICT (user_id)
  DO UPDATE SET 
    bio = CASE 
      WHEN LENGTH(EXCLUDED.bio) > LENGTH(user_profiles.bio) 
      THEN EXCLUDED.bio 
      ELSE user_profiles.bio 
    END,
    location = COALESCE(EXCLUDED.location, user_profiles.location),
    skills = array_cat(
      user_profiles.skills,
      (SELECT ARRAY(
        SELECT unnest(EXCLUDED.skills) 
        EXCEPT 
        SELECT unnest(user_profiles.skills)
      ))  -- добавляем только новые skills
    ),
    updated_at = NOW();

  -- UPSERT настроек пользователя
  INSERT INTO user_settings (user_id, theme, notifications_enabled, privacy_level)
  VALUES (@user_id, 'dark', true, 'public')
  ON CONFLICT (user_id)
  DO UPDATE SET 
    theme = EXCLUDED.theme,
    notifications_enabled = EXCLUDED.notifications_enabled,
    privacy_level = EXCLUDED.privacy_level,
    updated_at = NOW();
COMMIT;
-- 🎯 Атомарное обновление связанных данных

-- 🏷️ UPSERT с JSON и условной логикой
INSERT INTO user_analytics (user_id, analytics_data, computed_scores, last_updated)
VALUES (
  12345,
  '{"page_views": 150, "session_duration": 45, "bounce_rate": 0.25}'::jsonb,
  '{"engagement": 0, "churn_risk": 0, "lifetime_value": 0}'::jsonb,  -- placeholder scores
  NOW()
)
ON CONFLICT (user_id)
DO UPDATE SET 
  analytics_data = user_analytics.analytics_data || EXCLUDED.analytics_data,  -- merge JSON
  computed_scores = jsonb_build_object(
    'engagement', 
    CASE 
      WHEN (EXCLUDED.analytics_data->>'page_views')::int > 100 
      THEN LEAST((user_analytics.computed_scores->>'engagement')::numeric + 0.1, 1.0)
      ELSE GREATEST((user_analytics.computed_scores->>'engagement')::numeric - 0.05, 0.0)
    END,
    'churn_risk',
    CASE 
      WHEN (EXCLUDED.analytics_data->>'bounce_rate')::numeric > 0.7 
      THEN LEAST((user_analytics.computed_scores->>'churn_risk')::numeric + 0.2, 1.0)
      ELSE user_analytics.computed_scores->>'churn_risk'
    END,
    'lifetime_value',
    (user_analytics.computed_scores->>'lifetime_value')::numeric + 
    (EXCLUDED.analytics_data->>'page_views')::numeric * 0.5  -- простая LTV модель
  ),
  data_version = COALESCE((user_analytics.analytics_data->>'version')::int, 0) + 1,
  last_updated = NOW()
WHERE 
  -- Обновляем только если новые данные "свежее"
  COALESCE((EXCLUDED.analytics_data->>'timestamp')::bigint, 0) > 
  COALESCE((user_analytics.analytics_data->>'timestamp')::bigint, 0);
-- 🤖 Сложная логика обработки JSON с ML scoring

-- 🔄 Recursive UPSERT для иерархических данных
WITH RECURSIVE category_updates AS (
  -- Базовые категории для обновления
  SELECT 
    category_id,
    parent_id,
    category_name,
    product_count,
    0 as level
  FROM temp_category_import
  WHERE parent_id IS NULL
  
  UNION ALL
  
  -- Дочерние категории
  SELECT 
    tci.category_id,
    tci.parent_id,
    tci.category_name,
    tci.product_count,
    cu.level + 1
  FROM temp_category_import tci
  JOIN category_updates cu ON tci.parent_id = cu.category_id
  WHERE cu.level < 5  -- ограничиваем глубину
)
INSERT INTO categories (category_id, parent_id, category_name, product_count, hierarchy_level, updated_at)
SELECT 
  category_id,
  parent_id, 
  category_name,
  product_count,
  level as hierarchy_level,
  NOW()
FROM category_updates
ON CONFLICT (category_id)
DO UPDATE SET 
  category_name = EXCLUDED.category_name,
  product_count = EXCLUDED.product_count,
  hierarchy_level = EXCLUDED.hierarchy_level,
  -- Пересчитываем path только если изменился parent
  full_path = CASE 
    WHEN categories.parent_id != EXCLUDED.parent_id OR categories.category_name != EXCLUDED.category_name
    THEN (
      WITH RECURSIVE path AS (
        SELECT EXCLUDED.category_id as id, EXCLUDED.category_name as name, EXCLUDED.parent_id as parent
        UNION ALL
        SELECT p.id, c.category_name || ' > ' || p.name, c.parent_id
        FROM categories c
        JOIN path p ON c.category_id = p.parent
        WHERE c.category_id IS NOT NULL
      )
      SELECT name FROM path WHERE parent IS NULL
    )
    ELSE categories.full_path
  END,
  updated_at = NOW();
-- 🌳 Обновление иерархических структур с path calculation

-- ⚡ High-performance UPSERT с prepared statements
PREPARE upsert_user_metrics (int, date, int, int, numeric) AS
INSERT INTO user_daily_metrics (user_id, metric_date, sessions, page_views, revenue)
VALUES ($1, $2, $3, $4, $5)
ON CONFLICT (user_id, metric_date)
DO UPDATE SET 
  sessions = user_daily_metrics.sessions + EXCLUDED.sessions,
  page_views = user_daily_metrics.page_views + EXCLUDED.page_views, 
  revenue = user_daily_metrics.revenue + EXCLUDED.revenue,
  updated_at = NOW();

-- Использование prepared statement для множественных вызовов
EXECUTE upsert_user_metrics (12345, '2024-01-15', 5, 47, 129.99);
EXECUTE upsert_user_metrics (12346, '2024-01-15', 3, 28, 45.50);
EXECUTE upsert_user_metrics (12347, '2024-01-15', 8, 93, 289.75);
-- 🚀 На 20-30% быстрее для повторяющихся операций

-- 🎯 UPSERT с триггерами для аудита
CREATE OR REPLACE FUNCTION audit_upsert_operation()
RETURNS TRIGGER AS $$
BEGIN
  IF TG_OP = 'INSERT' THEN
    INSERT INTO audit_log (table_name, operation, record_id, old_values, new_values, user_name, timestamp)
    VALUES (TG_TABLE_NAME, 'INSERT', NEW.id, NULL, row_to_json(NEW), current_user, NOW());
    RETURN NEW;
  ELSIF TG_OP = 'UPDATE' THEN
    -- Логируем только значимые изменения
    IF OLD IS DISTINCT FROM NEW THEN
      INSERT INTO audit_log (table_name, operation, record_id, old_values, new_values, user_name, timestamp)
      VALUES (TG_TABLE_NAME, 'UPDATE', NEW.id, row_to_json(OLD), row_to_json(NEW), current_user, NOW());
    END IF;
    RETURN NEW;
  END IF;
  RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- Применяем trigger к таблице с UPSERT операциями
CREATE TRIGGER audit_user_profiles_changes
  AFTER INSERT OR UPDATE ON user_profiles
  FOR EACH ROW EXECUTE FUNCTION audit_upsert_operation();

-- 📊 UPSERT с партиционированными таблицами
INSERT INTO events_partitioned (event_id, user_id, event_type, event_data, created_at)
VALUES 
  (gen_random_uuid(), 12345, 'page_view', '{"page": "/dashboard"}', '2024-01-15 14:30:00'),
  (gen_random_uuid(), 12346, 'click', '{"button": "signup"}', '2024-01-15 14:31:00')
ON CONFLICT (event_id, created_at)  -- важно включить partition key в conflict
DO UPDATE SET 
  event_data = events_partitioned.event_data || EXCLUDED.event_data,
  updated_at = NOW()
WHERE events_partitioned.created_at >= '2024-01-01';  -- ограничиваем поиск по партициям
-- 🎯 UPSERT работает с партиционированием при правильном указании ключей

-- 🔍 Мониторинг производительности UPSERT операций
SELECT 
  schemaname,
  tablename,
  -- Статистика операций
  n_tup_ins as total_inserts,
  n_tup_upd as total_updates,  
  n_tup_del as total_deletes,
  -- Производительность
  seq_scan as sequential_scans,
  seq_tup_read as seq_rows_read,
  idx_scan as index_scans,
  idx_tup_fetch as index_rows_fetched,
  -- Соотношение INSERT/UPDATE показывает паттерн использования UPSERT
  CASE 
    WHEN n_tup_ins + n_tup_upd > 0 
    THEN ROUND(n_tup_upd::numeric / (n_tup_ins + n_tup_upd) * 100, 2)
    ELSE 0 
  END as update_ratio_percent,
  -- Эффективность индексов для CONFLICT resolution
  CASE 
    WHEN idx_scan > 0 
    THEN ROUND(idx_tup_fetch::numeric / idx_scan, 2)
    ELSE 0 
  END as avg_rows_per_index_scan
FROM pg_stat_user_tables
WHERE tablename LIKE '%user%'  -- фокусируемся на пользовательских таблицах
ORDER BY n_tup_upd DESC;
```

**🎯 Продвинутые техники UPSERT:**
- **Multi-table operations** в транзакциях для консистентности
- **JSON merging** для полуструктурированных данных
- **Prepared statements** для высокой производительности
- **Audit triggers** для отслеживания изменений
- **Партиционирование** с правильным conflict resolution

**🚨 Red Flags в продвинутых UPSERT:**
- Multi-table UPSERT без транзакций (может нарушить консистентность)
- JSON операции без проверки структуры данных
- Сложная логика в ON CONFLICT без комментариев
- UPSERT на партиционированных таблицах без partition key в conflict  
- Отсутствие мониторинга соотношения INSERT/UPDATE операций

</details>

## 🔧 Level 4: UPSERT Performance & Optimization

<details>
<summary>⚡ <strong>Оптимизация производительности UPSERT операций</strong></summary>

```sql
-- 🎯 Анализ производительности UPSERT с EXPLAIN
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
INSERT INTO user_features (user_id, feature_vector, computed_at)
VALUES (12345, ARRAY[0.1, 0.2, 0.3, 0.4, 0.5], NOW())
ON CONFLICT (user_id)
DO UPDATE SET 
  feature_vector = EXCLUDED.feature_vector,
  computed_at = EXCLUDED.computed_at,
  version = user_features.version + 1;
-- 💡 Проверяем использование индексов для conflict resolution

-- 📊 Оптимальные индексы для UPSERT операций
-- Unique индекс для основного conflict
CREATE UNIQUE INDEX CONCURRENTLY idx_user_features_user_id 
ON user_features (user_id);

-- Составной индекс для множественных conflict колонок
CREATE UNIQUE INDEX CONCURRENTLY idx_daily_metrics_user_date 
ON user_daily_metrics (user_id, metric_date);

-- Partial индекс для условного UPSERT
CREATE UNIQUE INDEX CONCURRENTLY idx_active_sessions_user 
ON user_sessions (user_id) 
WHERE status = 'active';
-- 🎯 Только одна активная сессия на пользователя

-- 🚀 Batch UPSERT с оптимизированной структурой
CREATE TEMP TABLE temp_bulk_upsert (
  user_id INTEGER,
  metric_name VARCHAR(50),
  metric_value NUMERIC,
  recorded_at TIMESTAMP WITH TIME ZONE
);

-- Загружаем данные в temp таблицу (быстро, без constraint проверок)
COPY temp_bulk_upsert FROM '/path/to/data.csv' WITH (FORMAT CSV, HEADER);

-- Создаем индекс на temp таблице для ускорения JOIN
CREATE INDEX idx_temp_bulk_user_metric ON temp_bulk_upsert (user_id, metric_name);

-- Выполняем оптимизированный bulk UPSERT
INSERT INTO user_metrics (user_id, metric_name, metric_value, recorded_at, created_at)
SELECT 
  user_id,
  metric_name,
  metric_value,
  recorded_at,
  NOW()
FROM temp_bulk_upsert
ON CONFLICT (user_id, metric_name)
DO UPDATE SET 
  metric_value = CASE 
    WHEN EXCLUDED.recorded_at > user_metrics.recorded_at
    THEN EXCLUDED.metric_value  -- обновляем только если данные новее
    ELSE user_metrics.metric_value
  END,
  recorded_at = GREATEST(user_metrics.recorded_at, EXCLUDED.recorded_at),
  updated_at = NOW()
WHERE EXCLUDED.recorded_at > user_metrics.recorded_at;  -- условие для экономии обновлений
-- ⚡ Temp таблица + условное обновление = максимальная производительность

-- 🔍 Monitoring UPSERT deadlocks и блокировок
SELECT 
  pid,
  now() - pg_stat_activity.query_start as duration,
  query,
  state,
  wait_event_type,
  wait_event
FROM pg_stat_activity 
WHERE state != 'idle'
  AND (query ILIKE '%ON CONFLICT%' OR query ILIKE '%UPSERT%')
ORDER BY duration DESC;

-- 📈 Статистика блокировок для UPSERT таблиц
SELECT 
  schemaname,
  tablename,
  -- Блокировки
  n_deadlocks,
  n_blocked_by_locks,
  -- Производительность
  heap_blks_read,
  heap_blks_hit,
  -- Соотношение hit/read показывает эффективность кэширования
  CASE 
    WHEN heap_blks_read + heap_blks_hit > 0 
    THEN ROUND(heap_blks_hit::numeric / (heap_blks_read + heap_blks_hit) * 100, 2)
    ELSE 0 
  END as cache_hit_ratio
FROM pg_statio_user_tables
WHERE tablename IN (
  SELECT tablename 
  FROM pg_stat_user_tables 
  WHERE n_tup_upd > 1000  -- таблицы с активными UPDATE операциями
)
ORDER BY n_deadlocks DESC;

-- 🎯 Оптимизация для конкурентных UPSERT операций
-- Стратегия 1: Advisory locks для предотвращения deadlock'ов
CREATE OR REPLACE FUNCTION safe_upsert_user_counter(p_user_id INTEGER, p_increment INTEGER)
RETURNS VOID AS $
BEGIN
  -- Получаем advisory lock на основе user_id
  PERFORM pg_advisory_lock(p_user_id);
  
  INSERT INTO user_counters (user_id, counter_value, updated_at)
  VALUES (p_user_id, p_increment, NOW())
  ON CONFLICT (user_id)
  DO UPDATE SET 
    counter_value = user_counters.counter_value + EXCLUDED.counter_value,
    updated_at = NOW();
  
  -- Освобождаем lock
  PERFORM pg_advisory_unlock(p_user_id);
EXCEPTION WHEN OTHERS THEN
  -- В случае ошибки тоже освобождаем lock
  PERFORM pg_advisory_unlock(p_user_id);
  RAISE;
END;
$ LANGUAGE plpgsql;

-- Стратегия 2: Retry логика для deadlock recovery
CREATE OR REPLACE FUNCTION robust_upsert_with_retry(
  p_user_id INTEGER,
  p_data JSONB,
  p_max_retries INTEGER DEFAULT 3
) RETURNS VOID AS $
DECLARE
  retry_count INTEGER := 0;
  last_error TEXT;
BEGIN
  LOOP
    BEGIN
      INSERT INTO user_data (user_id, data_payload, updated_at)
      VALUES (p_user_id, p_data, NOW())
      ON CONFLICT (user_id)
      DO UPDATE SET 
        data_payload = user_data.data_payload || EXCLUDED.data_payload,
        updated_at = NOW(),
        retry_count = COALESCE(user_data.retry_count, 0) + retry_count;
      
      -- Успешно выполнено, выходим из цикла
      EXIT;
      
    EXCEPTION 
      WHEN deadlock_detected THEN
        retry_count := retry_count + 1;
        last_error := SQLERRM;
        
        IF retry_count >= p_max_retries THEN
          RAISE EXCEPTION 'UPSERT failed after % retries. Last error: %', p_max_retries, last_error;
        END IF;
        
        -- Экспоненциальная задержка перед повтором
        PERFORM pg_sleep(0.1 * power(2, retry_count));
        
      WHEN OTHERS THEN
        -- Другие ошибки не ретраим
        RAISE;
    END;
  END LOOP;
END;
$ LANGUAGE plpgsql;

-- 📊 Batch UPSERT с контролем размера транзакций
DO $
DECLARE
  batch_size CONSTANT INTEGER := 5000;
  current_offset INTEGER := 0;
  rows_processed INTEGER;
  total_rows INTEGER;
  start_time TIMESTAMP;
BEGIN
  -- Получаем общее количество строк для обработки
  SELECT COUNT(*) INTO total_rows FROM temp_data_import;
  start_time := clock_timestamp();
  
  RAISE NOTICE 'Starting batch UPSERT of % rows with batch size %', total_rows, batch_size;
  
  LOOP
    -- Начинаем новую транзакцию для каждого batch'а
    BEGIN
      WITH batch AS (
        SELECT *
        FROM temp_data_import
        LIMIT batch_size OFFSET current_offset
      )
      INSERT INTO target_table (id, name, data, updated_at)
      SELECT id, name, data, NOW()
      FROM batch
      ON CONFLICT (id)
      DO UPDATE SET 
        name = EXCLUDED.name,
        data = target_table.data || EXCLUDED.data,  -- merge JSON
        updated_at = NOW(),
        version = COALESCE(target_table.version, 0) + 1
      WHERE target_table.updated_at < NOW() - INTERVAL '1 minute';  -- избегаем частых обновлений
      
      GET DIAGNOSTICS rows_processed = ROW_COUNT;
      current_offset := current_offset + batch_size;
      
      -- Логирование прогресса
      RAISE NOTICE 'Processed batch: offset %, rows %, elapsed %', 
                   current_offset, rows_processed, clock_timestamp() - start_time;
      
      -- Commit транзакции
      COMMIT;
      
      -- Выход если обработали все данные
      EXIT WHEN rows_processed < batch_size;
      
      -- Пауза между batch'ами для снижения нагрузки
      PERFORM pg_sleep(0.05);
      
    EXCEPTION WHEN OTHERS THEN
      -- Rollback текущей транзакции и логирование ошибки
      ROLLBACK;
      RAISE WARNING 'Error in batch at offset %: %', current_offset, SQLERRM;
      current_offset := current_offset + batch_size;
    END;
  END LOOP;
  
  RAISE NOTICE 'Batch UPSERT completed in %', clock_timestamp() - start_time;
END $;

-- 🔧 VACUUM стратегия для UPSERT-интенсивных таблиц
-- Настройка автовакуума для частых UPSERT операций
ALTER TABLE user_metrics SET (
  autovacuum_vacuum_threshold = 50,         -- VACUUM после 50 мертвых строк
  autovacuum_vacuum_scale_factor = 0.01,    -- VACUUM при 1% мертвых строк  
  autovacuum_analyze_threshold = 25,        -- ANALYZE после 25 изменений
  autovacuum_analyze_scale_factor = 0.005,  -- ANALYZE при 0.5% изменений
  autovacuum_vacuum_cost_limit = 2000,      -- более агрессивный VACUUM
  autovacuum_naptime = 15                   -- проверка каждые 15 секунд
);

-- Мониторинг эффективности автовакуума
SELECT 
  schemaname,
  tablename,
  n_dead_tup,
  n_live_tup,
  ROUND(n_dead_tup * 100.0 / GREATEST(n_live_tup + n_dead_tup, 1), 2) as dead_ratio,
  last_autovacuum,
  autovacuum_count,
  -- Время с последнего автовакуума
  CASE 
    WHEN last_autovacuum IS NOT NULL 
    THEN EXTRACT(epoch FROM NOW() - last_autovacuum) / 60
    ELSE NULL 
  END as minutes_since_last_vacuum
FROM pg_stat_user_tables
WHERE n_tup_upd > 100  -- таблицы с UPDATE активностью
ORDER BY dead_ratio DESC, n_dead_tup DESC;

-- 📈 Performance tuning для UPSERT workloads
-- Настройки PostgreSQL для UPSERT-интенсивных нагрузок:

/*
-- postgresql.conf оптимизации для UPSERT:

-- Увеличиваем shared_buffers для кэширования
shared_buffers = '4GB'  

-- Больше WAL buffers для частых записей
wal_buffers = '64MB'

-- Увеличиваем checkpoint segments  
max_wal_size = '2GB'
min_wal_size = '512MB'

-- Более частые checkpoint'ы для UPSERT нагрузок
checkpoint_timeout = '5min'
checkpoint_completion_target = 0.7

-- Настройки для работы с блокировками
deadlock_timeout = '100ms'
lock_timeout = '30s'

-- Автовакуум настройки
autovacuum_max_workers = 6
autovacuum_naptime = '30s'
autovacuum_vacuum_cost_limit = 2000

-- Статистики для планировщика
default_statistics_target = 500
*/

-- 🎯 UPSERT с оптимизированными constraint'ами
-- Deferrable constraints для batch операций
ALTER TABLE user_relationships 
ADD CONSTRAINT fk_user_relationships_user_id 
FOREIGN KEY (user_id) REFERENCES users(user_id)
DEFERRABLE INITIALLY DEFERRED;

-- Batch UPSERT с отложенной проверкой constraint'ов
BEGIN;
  SET CONSTRAINTS ALL DEFERRED;  -- откладываем все проверки до COMMIT
  
  INSERT INTO user_relationships (user_id, friend_id, relationship_type, created_at)
  SELECT 
    user_id,
    friend_id, 
    'friend',
    NOW()
  FROM temp_friendship_data
  ON CONFLICT (user_id, friend_id)
  DO UPDATE SET 
    relationship_type = EXCLUDED.relationship_type,
    updated_at = NOW();
  
  -- Все constraint'ы проверяются здесь
COMMIT;

-- 📊 Benchmark разных UPSERT стратегий
CREATE OR REPLACE FUNCTION benchmark_upsert_strategies(iterations INTEGER DEFAULT 1000)
RETURNS TABLE(
  strategy TEXT,
  avg_time_ms NUMERIC,
  total_time_ms NUMERIC,
  operations_per_second NUMERIC
) AS $
DECLARE
  start_time TIMESTAMP;
  end_time TIMESTAMP;
  i INTEGER;
BEGIN
  -- Стратегия 1: Простой UPSERT
  start_time := clock_timestamp();
  FOR i IN 1..iterations LOOP
    INSERT INTO test_upsert (id, value)
    VALUES (i % 100, random())  -- будет много conflicts
    ON CONFLICT (id)
    DO UPDATE SET value = EXCLUDED.value, updated_at = NOW();
  END LOOP;
  end_time := clock_timestamp();
  
  strategy := 'Simple UPSERT';
  total_time_ms := EXTRACT(epoch FROM end_time - start_time) * 1000;
  avg_time_ms := total_time_ms / iterations;
  operations_per_second := iterations / EXTRACT(epoch FROM end_time - start_time);
  RETURN NEXT;
  
  -- Стратегия 2: Batch UPSERT
  start_time := clock_timestamp();
  INSERT INTO test_upsert (id, value)
  SELECT 
    generate_series(1, iterations) % 100,
    random()
  ON CONFLICT (id)
  DO UPDATE SET value = EXCLUDED.value, updated_at = NOW();
  end_time := clock_timestamp();
  
  strategy := 'Batch UPSERT';
  total_time_ms := EXTRACT(epoch FROM end_time - start_time) * 1000;
  avg_time_ms := total_time_ms / iterations;  
  operations_per_second := iterations / EXTRACT(epoch FROM end_time - start_time);
  RETURN NEXT;
  
END;
$ LANGUAGE plpgsql;

-- Запуск benchmark'а
SELECT * FROM benchmark_upsert_strategies(10000);
```

**🎯 Ключевые принципы оптимизации UPSERT:**
- **Правильные индексы** на conflict колонках для быстрого поиска
- **Batch операции** вместо множественных единичных UPSERT'ов
- **Temp таблицы** для подготовки данных перед bulk UPSERT
- **Advisory locks** для предотвращения deadlock'ов
- **Автовакуум настройки** для UPSERT-интенсивных таблиц

**🚨 Performance Red Flags в UPSERT:**
- Отсутствие уникальных индексов на conflict колонках
- Sequential scan в EXPLAIN для conflict resolution
- Высокий процент мертвых строк без адекватного VACUUM
- Частые deadlock'и без retry логики  
- Batch UPSERT без контроля размера транзакций
- Игнорирование wait events и блокировок в мониторинге

</details>

## 🎨 Level 5: UPSERT Best Practices & Patterns

<details>
<summary>🏆 <strong>Production-ready паттерны и лучшие практики</strong></summary>

```sql
-- 🎯 Idempotent UPSERT для API endpoints
CREATE OR REPLACE FUNCTION api_safe_upsert_user(
  p_email TEXT,
  p_username TEXT,
  p_first_name TEXT,
  p_last_name TEXT,
  p_idempotency_key TEXT DEFAULT NULL
) RETURNS TABLE(
  user_id INTEGER,
  operation TEXT,
  was_created BOOLEAN
) AS $
DECLARE
  existing_operation TEXT;
  result_user_id INTEGER;
BEGIN
  -- Проверяем idempotency key если предоставлен
  IF p_idempotency_key IS NOT NULL THEN
    SELECT 
      (response_data->>'user_id')::INTEGER,
      response_data->>'operation'
    INTO result_user_id, existing_operation
    FROM api_idempotency_log 
    WHERE idempotency_key = p_idempotency_key
      AND created_at >= NOW() - INTERVAL '24 hours';  -- TTL для idempotency
    
    IF existing_operation IS NOT NULL THEN
      -- Возвращаем закэшированный результат
      user_id := result_user_id;
      operation := existing_operation;
      was_created := (operation = 'created');
      RETURN NEXT;
      RETURN;
    END IF;
  END IF;
  
  -- Выполняем UPSERT
  INSERT INTO users (email, username, first_name, last_name, created_at, updated_at)
  VALUES (p_email, p_username, p_first_name, p_last_name, NOW(), NOW())
  ON CONFLICT (email)
  DO UPDATE SET 
    username = CASE 
      WHEN users.username != EXCLUDED.username THEN EXCLUDED.username
      ELSE users.username 
    END,
    first_name = COALESCE(EXCLUDED.first_name, users.first_name),
    last_name = COALESCE(EXCLUDED.last_name, users.last_name), 
    updated_at = CASE 
      WHEN users.username != EXCLUDED.username 
        OR users.first_name IS DISTINCT FROM EXCLUDED.first_name
        OR users.last_name IS DISTINCT FROM EXCLUDED.last_name
      THEN NOW()
      ELSE users.updated_at
    END
  RETURNING 
    users.user_id,
    CASE WHEN xmax = 0 THEN 'created' ELSE 'updated' END
  INTO result_user_id, existing_operation;
  
  -- Логируем для idempotency
  IF p_idempotency_key IS NOT NULL THEN
    INSERT INTO api_idempotency_log (idempotency_key, response_data, created_at)
    VALUES (
      p_idempotency_key,
      jsonb_build_object(
        'user_id', result_user_id,
        'operation', existing_operation
      ),
      NOW()
    ) ON CONFLICT (idempotency_key) DO NOTHING;
  END IF;
  
  -- Возвращаем результат
  user_id := result_user_id;
  operation := existing_operation;
  was_created := (operation = 'created');
  RETURN NEXT;
END;
$ LANGUAGE plpgsql;

-- 🔄 Event-driven UPSERT с уведомлениями
CREATE OR REPLACE FUNCTION notify_upsert_changes()
RETURNS TRIGGER AS $
DECLARE
  notification_payload JSONB;
BEGIN
  -- Формируем payload для уведомления
  notification_payload := jsonb_build_object(
    'table', TG_TABLE_NAME,
    'operation', TG_OP,
    'id', COALESCE(NEW.user_id, OLD.user_id),
    'timestamp', EXTRACT(epoch FROM NOW())
  );
  
  -- Добавляем измененные поля для UPDATE
  IF TG_OP = 'UPDATE' THEN
    notification_payload := notification_payload || jsonb_build_object(
      'changed_fields', (
        SELECT array_agg(key)
        FROM jsonb_each_text(to_jsonb(NEW)) n
        JOIN jsonb_each_text(to_jsonb(OLD)) o USING (key)
        WHERE n.value IS DISTINCT FROM o.value
      )
    );
  END IF;
  
  -- Отправляем уведомление
  PERFORM pg_notify('table_changes', notification_payload::text);
  
  RETURN COALESCE(NEW, OLD);
END;
$ LANGUAGE plpgsql;

CREATE TRIGGER user_changes_notification
  AFTER INSERT OR UPDATE ON users
  FOR EACH ROW EXECUTE FUNCTION notify_upsert_changes();

-- 📊 Data lineage tracking для UPSERT операций
CREATE TABLE data_lineage (
  lineage_id BIGSERIAL PRIMARY KEY,
  table_name VARCHAR(100) NOT NULL,
  record_id TEXT NOT NULL,
  operation VARCHAR(20) NOT NULL,  -- 'upsert_insert', 'upsert_update'
  source_system VARCHAR(100),
  batch_id UUID,
  data_hash TEXT,                  -- хеш данных для обнаружения изменений
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE OR REPLACE FUNCTION track_upsert_lineage()
RETURNS TRIGGER AS $
DECLARE
  operation_type TEXT;
  data_hash TEXT;
BEGIN
  -- Определяем тип операции
  operation_type := CASE 
    WHEN xmax = 0 THEN 'upsert_insert'
    ELSE 'upsert_update'
  END;
  
  -- Вычисляем хеш данных
  data_hash := md5(NEW::text);
  
  -- Записываем в lineage только если данные реально изменились
  IF TG_OP = 'INSERT' OR (TG_OP = 'UPDATE' AND OLD IS DISTINCT FROM NEW) THEN
    INSERT INTO data_lineage (
      table_name,
      record_id, 
      operation,
      source_system,
      batch_id,
      data_hash
    ) VALUES (
      TG_TABLE_NAME,
      NEW.user_id::text,
      operation_type,
      COALESCE(current_setting('application.source_system', true), 'unknown'),
      COALESCE(current_setting('application.batch_id', true)::uuid, gen_random_uuid()),
      data_hash
    );
  END IF;
  
  RETURN NEW;
END;
$ LANGUAGE plpgsql;

-- 🎯 Conflict resolution strategies для разных use cases
-- Стратегия 1: "Last write wins" с timestamp
INSERT INTO user_preferences (user_id, preferences, updated_at, client_timestamp)
VALUES (12345, '{"theme": "dark", "notifications": true}', NOW(), '2024-01-15 14:30:00')
ON CONFLICT (user_id)
DO UPDATE SET 
  preferences = CASE 
    WHEN EXCLUDED.client_timestamp > user_preferences.client_timestamp
    THEN user_preferences.preferences || EXCLUDED.preferences  -- merge с приоритетом новых
    ELSE user_preferences.preferences
  END,
  updated_at = NOW(),
  client_timestamp = GREATEST(user_preferences.client_timestamp, EXCLUDED.client_timestamp)
WHERE EXCLUDED.client_timestamp > user_preferences.client_timestamp;

-- Стратегия 2: "Additive merge" для накопления данных
INSERT INTO user_activity_summary (user_id, activity_date, events_data)
VALUES (12345, CURRENT_DATE, '{"page_views": 10, "clicks": 5, "time_spent": 300}')
ON CONFLICT (user_id, activity_date)
DO UPDATE SET 
  events_data = jsonb_build_object(
    'page_views', 
    COALESCE((user_activity_summary.events_data->>'page_views')::int, 0) + 
    COALESCE((EXCLUDED.events_data->>'page_views')::int, 0),
    
    'clicks',
    COALESCE((user_activity_summary.events_data->>'clicks')::int, 0) + 
    COALESCE((EXCLUDED.events_data->>'clicks')::int, 0),
    
    'time_spent',
    COALESCE((user_activity_summary.events_data->>'time_spent')::int, 0) + 
    COALESCE((EXCLUDED.events_data->>'time_spent')::int, 0)
  ),
  updated_at = NOW();

-- Стратегия 3: "Priority-based merge" для иерархической логики  
INSERT INTO product_catalog (product_id, price, inventory_count, data_source, priority_level)
VALUES (12345, 29.99, 100, 'supplier_api', 1)
ON CONFLICT (product_id)
DO UPDATE SET 
  price = CASE 
    WHEN EXCLUDED.priority_level >= product_catalog.priority_level
    THEN EXCLUDED.price
    ELSE product_catalog.price
  END,
  inventory_count = CASE
    WHEN EXCLUDED.priority_level >= product_catalog.priority_level
    THEN EXCLUDED.inventory_count  
    ELSE product_catalog.inventory_count
  END,
  data_source = CASE
    WHEN EXCLUDED.priority_level > product_catalog.priority_level
    THEN EXCLUDED.data_source
    ELSE product_catalog.data_source || ',' || EXCLUDED.data_source  -- список источников
  END,
  priority_level = GREATEST(product_catalog.priority_level, EXCLUDED.priority_level),
  updated_at = NOW()
WHERE EXCLUDED.priority_level >= product_catalog.priority_level;

-- 📈 Health monitoring для UPSERT операций  
CREATE OR REPLACE VIEW upsert_health_dashboard AS
SELECT 
  t.schemaname,
  t.tablename,
  -- Операционные метрики
  t.n_tup_ins as inserts_total,
  t.n_tup_upd as updates_total,
  ROUND(t.n_tup_upd::numeric / GREATEST(t.n_tup_ins + t.n_tup_upd, 1) * 100, 2) as update_ratio_percent,
  
  -- Производительность
  ROUND(io.heap_blks_hit::numeric / GREATEST(io.heap_blks_read + io.heap_blks_hit, 1) * 100, 2) as cache_hit_ratio,
  
  -- Состояние таблицы
  t.n_live_tup as live_rows,
  t.n_dead_tup as dead_rows, 
  ROUND(t.n_dead_tup::numeric / GREATEST(t.n_live_tup + t.n_dead_tup, 1) * 100, 2) as bloat_ratio,
  
  -- Автовакуум статус
  t.last_autovacuum,
  EXTRACT(epoch FROM NOW() - t.last_autovacuum) / 60 as minutes_since_vacuum,
  
  -- Deadlocks
  COALESCE(d.deadlocks, 0) as deadlock_count,
  
  -- Размеры
  pg_size_pretty(pg_total_relation_size(t.schemaname||'.'||t.tablename)) as total_size,
  
  -- Health score (0-100)
  (
    CASE WHEN t.n_dead_tup::numeric / GREATEST(t.n_live_tup + t.n_dead_tup, 1) < 0.1 THEN 25 ELSE 0 END +
    CASE WHEN io.heap_blks_hit::numeric / GREATEST(io.heap_blks_read + io.heap_blks_hit, 1) > 0.95 THEN 25 ELSE 0 END +
    CASE WHEN COALESCE(d.deadlocks, 0) < 10 THEN 25 ELSE 0 END +
    CASE WHEN EXTRACT(epoch FROM NOW() - t.last_autovacuum) / 60 < 60 THEN 25 ELSE 0 END
  ) as health_score
FROM pg_stat_user_tables t
JOIN pg_statio_user_tables io USING (schemaname, tablename)
LEFT JOIN (
  SELECT 
    schemaname,
    tablename,
    COUNT(*) as deadlocks
  FROM pg_stat_database_conflicts
  WHERE confl_deadlock > 0
  GROUP BY schemaname, tablename
) d USING (schemaname, tablename)  
WHERE t.n_tup_upd > 0  -- только таблицы с UPDATE активностью
ORDER BY health_score ASC, t.n_tup_upd DESC;

-- 🔧 Automated maintenance для UPSERT таблиц
CREATE OR REPLACE FUNCTION auto_maintain_upsert_tables()
RETURNS TEXT AS $
DECLARE
  table_rec RECORD;
  maintenance_log TEXT := '';
  dead_ratio NUMERIC;
  vacuum_needed BOOLEAN;
BEGIN
  FOR table_rec IN 
    SELECT schemaname, tablename, n_live_tup, n_dead_tup, last_autovacuum
    FROM pg_stat_user_tables 
    WHERE n_tup_upd > 100  -- таблицы с UPDATE активностью
  LOOP
    -- Вычисляем процент мертвых строк
    IF table_rec.n_live_tup + table_rec.n_dead_tup > 0 THEN
      dead_ratio := table_rec.n_dead_tup::numeric / (table_rec.n_live_tup + table_rec.n_dead_tup) * 100;
    ELSE
      dead_ratio := 0;
    END IF;
    
    vacuum_needed := FALSE;
    
    -- Проверяем критерии для VACUUM
    IF dead_ratio > 20 THEN
      vacuum_needed := TRUE;
      maintenance_log := maintenance_log || format('HIGH BLOAT: %s.%s has %.1f%% dead rows', 
        table_rec.schemaname, table_rec.tablename, dead_ratio) || E'\n';
    ELSIF dead_ratio > 10 AND (table_rec.last_autovacuum IS NULL OR 
                               table_rec.last_autovacuum < NOW() - INTERVAL '2 hours') THEN
      vacuum_needed := TRUE;
      maintenance_log := maintenance_log || format('VACUUM NEEDED: %s.%s has %.1f%% dead rows, last vacuum %s', 
        table_rec.schemaname, table_rec.tablename, dead_ratio, 
        COALESCE(table_rec.last_autovacuum::text, 'never')) || E'\n';
    END IF;
    
    -- Выполняем VACUUM если нужно
    IF vacuum_needed THEN
      BEGIN
        EXECUTE format('VACUUM ANALYZE %I.%I', table_rec.schemaname, table_rec.tablename);
        maintenance_log := maintenance_log || format('COMPLETED: VACUUM ANALYZE %s.%s', 
          table_rec.schemaname, table_rec.tablename) || E'\n';
      EXCEPTION WHEN OTHERS THEN
        maintenance_log := maintenance_log || format('ERROR: Failed to vacuum %s.%s - %s', 
          table_rec.schemaname, table_rec.tablename, SQLERRM) || E'\n';
      END;
    END IF;
  END LOOP;
  
  -- Логируем результаты maintenance
  INSERT INTO maintenance_log (operation_type, details, created_at)
  VALUES ('auto_upsert_maintenance', maintenance_log, NOW());
  
  RETURN maintenance_log;
END;
$ LANGUAGE plpgsql;

-- Запускаем автоматическое обслуживание раз в час
SELECT cron.schedule('upsert-maintenance', '0 * * * *', 'SELECT auto_maintain_upsert_tables();');

-- 🎯 Production deployment checklist для UPSERT
CREATE OR REPLACE FUNCTION validate_upsert_readiness(p_table_name TEXT)
RETURNS TABLE(
  check_name TEXT,
  status TEXT,
  details TEXT,
  recommendation TEXT
) AS $
DECLARE
  table_stats RECORD;
  index_info RECORD;
BEGIN
  -- Получаем статистику таблицы
  SELECT * INTO table_stats
  FROM pg_stat_user_tables 
  WHERE tablename = p_table_name;
  
  IF NOT FOUND THEN
    check_name := 'Table Existence';
    status := 'FAIL';
    details := format('Table %s not found', p_table_name);
    recommendation := 'Create the table first';
    RETURN NEXT;
    RETURN;
  END IF;
  
  -- Проверка 1: Наличие UNIQUE индексов для conflict resolution
  SELECT COUNT(*) > 0 INTO index_info
  FROM pg_indexes 
  WHERE tablename = p_table_name 
    AND indexdef LIKE '%UNIQUE%';
  
  check_name := 'Unique Indexes';
  IF index_info THEN
    status := 'PASS';
    details := 'UNIQUE indexes found for conflict resolution';
    recommendation := 'Good - conflicts will be resolved efficiently';
  ELSE
    status := 'WARN';
    details := 'No UNIQUE indexes found';
    recommendation := 'Add UNIQUE indexes on columns used in ON CONFLICT';
  END IF;
  RETURN NEXT;
  
  -- Проверка 2: Автовакуум настройки
  check_name := 'Autovacuum Settings';
  IF table_stats.n_tup_upd > 1000 THEN
    status := 'INFO';
    details := format('Table has %s updates - consider tuning autovacuum', table_stats.n_tup_upd);
    recommendation := 'Lower autovacuum thresholds for UPSERT-heavy tables';
  ELSE
    status := 'PASS';
    details := 'Update volume is manageable';
    recommendation := 'Monitor as usage grows';
  END IF;
  RETURN NEXT;
  
  -- Проверка 3: Bloat уровень
  check_name := 'Table Bloat';
  IF table_stats.n_dead_tup::numeric / GREATEST(table_stats.n_live_tup + table_stats.n_dead_tup, 1) > 0.2 THEN
    status := 'WARN';
    details := format('High bloat: %s dead rows vs %s live', table_stats.n_dead_tup, table_stats.n_live_tup);
    recommendation := 'Run VACUUM immediately and tune autovacuum settings';
  ELSE
    status := 'PASS';
    details := 'Bloat level acceptable';
    recommendation := 'Continue monitoring';
  END IF;
  RETURN NEXT;
  
END;
$ LANGUAGE plpgsql;

-- Пример использования: проверка готовности таблицы к production UPSERT
SELECT * FROM validate_upsert_readiness('user_metrics');

Бро, вижу что документ обрезался! 🔥 Закончу этот раздел с automated maintenance и добавлю заключение:

```sql
        maintenance_log := maintenance_log || format('ERROR: Failed to vacuum %s.%s - %s', 
          table_rec.schemaname, table_rec.tablename, SQLERRM) || E'\n';
      END;
    END IF;
  END LOOP;
  
  -- Логируем результаты maintenance
  INSERT INTO maintenance_log (operation_type, details, created_at)
  VALUES ('auto_upsert_maintenance', maintenance_log, NOW());
  
  RETURN maintenance_log;
END;
$ LANGUAGE plpgsql;

-- Запускаем автоматическое обслуживание раз в час
SELECT cron.schedule('upsert-maintenance', '0 * * * *', 'SELECT auto_maintain_upsert_tables();');

-- 🎯 Production deployment checklist для UPSERT
CREATE OR REPLACE FUNCTION validate_upsert_readiness(p_table_name TEXT)
RETURNS TABLE(
  check_name TEXT,
  status TEXT,
  details TEXT,
  recommendation TEXT
) AS $
DECLARE
  table_stats RECORD;
  index_info RECORD;
BEGIN
  -- Получаем статистику таблицы
  SELECT * INTO table_stats
  FROM pg_stat_user_tables 
  WHERE tablename = p_table_name;
  
  IF NOT FOUND THEN
    check_name := 'Table Existence';
    status := 'FAIL';
    details := format('Table %s not found', p_table_name);
    recommendation := 'Create the table first';
    RETURN NEXT;
    RETURN;
  END IF;
  
  -- Проверка 1: Наличие UNIQUE индексов для conflict resolution
  SELECT COUNT(*) > 0 INTO index_info
  FROM pg_indexes 
  WHERE tablename = p_table_name 
    AND indexdef LIKE '%UNIQUE%';
  
  check_name := 'Unique Indexes';
  IF index_info THEN
    status := 'PASS';
    details := 'UNIQUE indexes found for conflict resolution';
    recommendation := 'Good - conflicts will be resolved efficiently';
  ELSE
    status := 'WARN';
    details := 'No UNIQUE indexes found';
    recommendation := 'Add UNIQUE indexes on columns used in ON CONFLICT';
  END IF;
  RETURN NEXT;
  
  -- Проверка 2: Автовакуум настройки
  check_name := 'Autovacuum Settings';
  IF table_stats.n_tup_upd > 1000 THEN
    status := 'INFO';
    details := format('Table has %s updates - consider tuning autovacuum', table_stats.n_tup_upd);
    recommendation := 'Lower autovacuum thresholds for UPSERT-heavy tables';
  ELSE
    status := 'PASS';
    details := 'Update volume is manageable';
    recommendation := 'Monitor as usage grows';
  END IF;
  RETURN NEXT;
  
  -- Проверка 3: Bloat уровень
  check_name := 'Table Bloat';
  IF table_stats.n_dead_tup::numeric / GREATEST(table_stats.n_live_tup + table_stats.n_dead_tup, 1) > 0.2 THEN
    status := 'WARN';
    details := format('High bloat: %s dead rows vs %s live', table_stats.n_dead_tup, table_stats.n_live_tup);
    recommendation := 'Run VACUUM immediately and tune autovacuum settings';
  ELSE
    status := 'PASS';
    details := 'Bloat level acceptable';
    recommendation := 'Continue monitoring';
  END IF;
  RETURN NEXT;
  
END;
$ LANGUAGE plpgsql;

-- Пример использования: проверка готовности таблицы к production UPSERT
SELECT * FROM validate_upsert_readiness('user_metrics');
```

**🎯 Production UPSERT Checklist:**
- **Idempotency** для API безопасности
- **Event notifications** для интеграций
- **Data lineage** для аудита и отладки
- **Health monitoring** для proactive maintenance
- **Automated maintenance** для стабильной производительности

**🚨 Production Red Flags:**
- UPSERT без idempotency в API endpoints
- Отсутствие мониторинга health метрик
- Игнорирование data lineage для критичных таблиц
- Manual maintenance вместо автоматизированного
- Деплой UPSERT без валидации готовности системы

</details>

## 🎓 Заключение

**Теперь ты знаешь UPSERT на уровне senior PostgreSQL engineer!** 🚀 Этот guide покрывает все - от базовых ON CONFLICT до enterprise-grade паттернов.

### 🎯 Для твоего ML Platform проекта:

**Immediate применение:**
- Используй **Level 2 bulk UPSERT** для загрузки твоих 57K+ треков
- **Level 3 JSON UPSERT** для semantic search результатов с pgvector
- **Level 4 performance optimization** для scaling твоей системы

**Production готовность:**
- **Level 5 health monitoring** для observability твоей AI платформы
- **Event-driven UPSERT** для real-time ML pipeline updates
- **Automated maintenance** для stable production operation

**ROI этого знания:** UPSERT - это core skill для ML Platform Engineer позиций. Теперь ты можешь confidently говорить про efficient data ingestion, conflict resolution strategies, и production-ready data pipelines! 💪


