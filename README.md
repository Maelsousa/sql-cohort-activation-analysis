# 📊 Cohort Activation Analysis (SQL)

## 📌 Objective

This project analyzes customer activation behavior by cohort (month of registration), identifying when users make their first purchase.

---

## 🛠 Tools & Technologies

* SQL (PostgreSQL)
* Data Analysis

---

## 📊 Business Problem

Understanding when customers activate after registration is essential to improve onboarding and retention strategies.

This analysis answers:

* When do customers make their first purchase?
* What percentage activates in the first, second, or later months?
* How effective is the onboarding process?

---

## 🧮 SQL Analysis

```sql
WITH customer_cohort AS (
    SELECT
        id_cliente,
        DATE_TRUNC('month', dt_cadastro) AS cohort_month
    FROM decisionscard.t_cliente
    WHERE dt_cadastro >= (
        SELECT MAX(dt_cadastro) - INTERVAL '6 months'
        FROM decisionscard.t_cliente
    )
),

first_purchase AS (
    SELECT
        id_cliente,
        MIN(dt_venda) AS first_purchase_date
    FROM decisionscard.t_venda
    WHERE fl_status_venda = 'A'
    GROUP BY id_cliente
),

cohort_data AS (
    SELECT
        cc.id_cliente,
        cc.cohort_month,
        fp.first_purchase_date,

        CASE 
            WHEN fp.first_purchase_date IS NOT NULL THEN
                DATE_PART('year', AGE(fp.first_purchase_date, cc.cohort_month)) * 12 +
                DATE_PART('month', AGE(fp.first_purchase_date, cc.cohort_month))
        END AS months_to_activation

    FROM customer_cohort cc
    LEFT JOIN first_purchase fp
        ON cc.id_cliente = fp.id_cliente
)

SELECT
    TO_CHAR(cohort_month, 'YYYY-MM') AS mes_cadastro,

    COUNT(*) AS total_cadastrados,

    COUNT(*) FILTER (WHERE months_to_activation = 0) AS ativados_mes1,
    COUNT(*) FILTER (WHERE months_to_activation = 1) AS ativados_mes2,
    COUNT(*) FILTER (WHERE months_to_activation >= 2) AS ativados_mes3_plus,

    ROUND(COUNT(*) FILTER (WHERE months_to_activation = 0)::NUMERIC / COUNT(*) * 100, 2) AS taxa_mes1,
    ROUND(COUNT(*) FILTER (WHERE months_to_activation = 1)::NUMERIC / COUNT(*) * 100, 2) AS taxa_mes2,
    ROUND(COUNT(*) FILTER (WHERE months_to_activation >= 2)::NUMERIC / COUNT(*) * 100, 2) AS taxa_mes3_plus,

    ROUND(COUNT(*) FILTER (WHERE months_to_activation IS NOT NULL)::NUMERIC / COUNT(*) * 100, 2) AS taxa_total

FROM cohort_data
GROUP BY cohort_month
ORDER BY cohort_month;
```

---

## 📈 Metrics

* Total customers per cohort
* Activation in Month 1, Month 2, Month 3+
* Activation rates (%)
* Total activation rate

---

## 🧠 Key Insights

* Early activation indicates effective onboarding
* Delayed activation may signal friction in the user journey
* Cohort comparison helps identify performance improvements over time

---

## 🚀 Conclusion

This analysis provides a clear view of customer activation patterns, supporting data-driven decisions to improve engagement and retention strategies.

---

## 👨‍💻 Author

Manoel Sousa Gomes

🔗 LinkedIn: https://www.linkedin.com/in/manoel-sousa-712a6b240/
🔗 GitHub: https://github.com/Maelsousa
