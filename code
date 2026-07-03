WITH base AS (                              -- створюю основу на яку потім будунакладати додаткові стовпці
  SELECT ses.date
     , sespar.country
     , acc.send_interval
     , acc.is_verified
     , acc.is_unsubscribed
     , acc.id                                  
FROM `data-analytics-mate.DA.account` acc
JOIN `DA.account_session` ases
ON acc.id = ases.account_id
JOIN `DA.session_params` sespar
ON ases.ga_session_id = sespar.ga_session_id
JOIN `DA.session` ses
ON ases.ga_session_id= ses.ga_session_id
),


account_met AS (                                   -- рахую метрику по аккаунтам
SELECT b.date
     , b.country
     , b.send_interval
     , b.is_verified
     , b.is_unsubscribed
     , COUNT(DISTINCT b.id) AS account_cnt
FROM base b
GROUP BY 1, 2, 3, 4, 5
),




email_met AS (                                     -- рахую метрику по імейлам
SELECT DATE_ADD(b.date, INTERVAL est.sent_date DAY) AS date
     , b.country
     , b.send_interval
     , b.is_verified
     , b.is_unsubscribed
     , COUNT(DISTINCT est.id_message) AS sent_msg
     , COUNT(DISTINCT epn.id_message) AS open_msg
     , COUNT(DISTINCT evt.id_message) AS visit_msg
FROM base b
JOIN`DA.email_sent` est
ON est.id_account = b.id
LEFT JOIN `DA.email_open` epn
ON est.id_message = epn.id_message
LEFT JOIN `DA.email_visit` evt
ON est.id_message = evt.id_message
GROUP BY 1,2,3,4,5),


union1 AS (                                           -- об'єдную дані
SELECT date
     , country
     , send_interval
     , is_verified
     , is_unsubscribed
     , account_cnt
     , 0 AS sent_msg
     , 0 AS open_msg
     , 0 AS visit_msg
FROM account_met


UNION ALL


SELECT date
     , country
     , send_interval
     , is_verified
     , is_unsubscribed
     , 0 AS account_cnt
     , sent_msg
     , open_msg
     , visit_msg
FROM email_met
),


final_mtrx AS (                                   -- сумую об'єднані дані аби уникнути нулів
     SELECT date
     , country
     , send_interval
     , is_verified
     , is_unsubscribed
     , SUM(account_cnt) AS account_cnt
     , SUM(sent_msg) AS sent_msg
     , SUM(open_msg) AS open_msg
     , SUM(visit_msg) AS visit_msg
FROM union1
GROUP BY 1,2,3,4,5
),


final0 AS (SELECT *                                   -- додаю віконні функції
     , SUM(account_cnt) OVER(PARTITION BY country) AS total_country_account_cnt
     , SUM(sent_msg) OVER(PARTITION BY country) AS total_country_sent_cnt
FROM final_mtrx),


final AS (                                             -- додаю ранги
     SELECT *
     , DENSE_RANK() OVER(ORDER BY total_country_account_cnt DESC) AS rank_total_country_account_cnt
     , DENSE_RANK() OVER(ORDER BY total_country_sent_cnt DESC) AS rank_total_country_sent_cnt
     FROM final0
)


SELECT *                                     -- вивожу фінальні результати з фільтруванням
FROM final
WHERE rank_total_country_account_cnt <= 10
     OR rank_total_country_sent_cnt <= 10
