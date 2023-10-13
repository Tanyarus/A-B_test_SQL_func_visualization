# Задание 2. SQL
2.1 Очень усердные ученики.

2.1.1 Условие

Образовательные курсы состоят из различных уроков, каждый из которых состоит из нескольких маленьких заданий. Каждое такое маленькое задание называется "горошиной".

Назовём очень усердным учеником того пользователя, который хотя бы раз за текущий месяц правильно решил 20 горошин.

### структура данных:

#### default.peas:

st_id - ID ученика

timest - время решения карточки

correct - правильно ли решена горошина

subject - дисциплина, в которой находится горошина


    SELECT
      st_id AS diligent_students,
      SUM(correct) AS sum_correct
    FROM
      default.peas
    WHERE
      correct = 1
      AND CAST(timest AS DATE) BETWEEN '2021-10-01'
      AND '2021-11-01'
    GROUP BY
      diligent_students
    HAVING
      sum_correct >= 20
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
# 2.1.2 Задача

Необходимо написать оптимальный запрос, который даст информацию о количестве очень усердных студентов.

NB! Под усердным студентом мы понимаем студента, который правильно решил 20 задач за текущий месяц.

    SELECT
      COUNT(st_id) AS diligent_students
    FROM
      (
        SELECT
          st_id,
          SUM(correct) AS sum_correct
        FROM
          default.peas
        WHERE
          correct = 1
          AND CAST(timest AS DATE) BETWEEN '2021-10-01'
          AND '2021-11-01'
        GROUP BY
          st_id
        HAVING
          sum_correct >= 20
      )
  
   
  
  
# 2.2.2
Образовательная платформа предлагает пройти студентам курсы по модели trial: студент может решить бесплатно лишь 30 горошин в день. Для     неограниченного количества заданий в определенной дисциплине студенту необходимо приобрести полный доступ. Команда провела эксперимент, где был протестирован новый экран оплаты.

Необходимо в одном запросе выгрузить следующую информацию о группах пользователей:

ARPU 
ARPAU 
CR в покупку 
СR активного пользователя в покупку 
CR пользователя из активности по математике (subject = ’math’) в покупку курса по математике
ARPU считается относительно всех пользователей, попавших в группы.

Активным считается пользователь, за все время решивший больше 10 задач правильно в любых дисциплинах.

Активным по математике считается пользователь, за все время решивший 2 или больше задач правильно по математике.

### структура данных:

#### default.studs:

st_id - ID ученика

test_grp - метка ученика в данном эксперементе

#### default.final_project_check:

st_id - ID ученика

sale_time - время покупки

money - Цена, по которой приобрели данный курс

subject - Дисциплина, на которую приобрели полный доступ


     SELECT
      l.test_grp,
      COUNT(DISTINCT l.st_id) FILTER (
        WHERE
          all_subj_money > 0
      ) / COUNT(DISTINCT l.st_id) AS CR,
      COUNT(DISTINCT l.st_id) FILTER (
        WHERE
          all_subj_money > 0
      ) / COUNT(DISTINCT l.st_id) FILTER (
        WHERE
          r.all_sum_correct > 10
      ) AS CR_active,
      COUNT(DISTINCT l.st_id) FILTER (
        WHERE
          l.money_math > 0
      ) / COUNT(DISTINCT l.st_id) FILTER (
        WHERE
            r.sum_correct_math >= 2
      ) AS CR_math,
      SUM(l.all_subj_money) / COUNT(DISTINCT l.st_id) AS ARPU,
      SUM(l.all_subj_money) / (
        COUNT(DISTINCT l.st_id) FILTER (
          WHERE
            all_subj_money > 0
        )
      ) AS ARPPU 
    FROM
      (
        SELECT
          l.st_id,
          l.test_grp,
          SUM(r.money) FILTER (
            WHERE
              paid_subject = 'Math'
          ) AS money_math,
          SUM(r.money) AS all_subj_money
        FROM
          (
            SELECT
              st_id,
              test_grp
            FROM
              default.studs
          ) AS l
          LEFT JOIN (
            SELECT
              st_id,
              money,
              subject AS paid_subject
            FROM
              default.final_project_check
          ) AS r ON l.st_id = r.st_id
        GROUP BY
          l.st_id,
          l.test_grp
      ) AS l
      LEFT JOIN (
        SELECT
          st_id,
          SUM(correct) AS all_sum_correct,
          SUM(correct) FILTER (
            WHERE
              subject = 'Math'
          ) AS sum_correct_math
        FROM
          default.peas
        GROUP BY
          st_id
      ) AS r ON l.st_id = r.st_id
    GROUP BY
      l.test_grp
  
  
 
