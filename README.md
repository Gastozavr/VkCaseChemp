
# Модель CommerceScore
$$
\text{CommerceScore} =
W_{\text{Relevance}} \times
W_{\text{Quality}} \times
P_{\text{Health}} \times
W_{\text{Engagement}} \times
B_{\text{Trend}}
$$

Учитывает: 
* релевантность товара автору,
* качество автора,
* его качество оформления контента и отношение аудитории,
* реальное вовлечение,
* актуальность, сложность и конкурентность категории.

---
## 1. $W_{\text{Relevance}}$ (вес релевантности товара автору)

$$
W_{\text{Relevance}} =
(k_C \cdot \text{Match}_{\text{Cat}}) +
(k_F \cdot \text{Match}_{\text{Format}})
$$


**Параметры:**

### 1.1 MatchCat (совпадение категории автора и товара)

Берутся топ-популярные категории автора, разделённые на три группы:

* первая треть → 1.0
* вторая → 0.7
* третья → 0.5

### 1.2 MatchFormat (совпадение формата автора и формата категории товара)

$$
\text{Match}_{\text{Format}} =
1 - \left|\text{Предпочтение категории} - \text{Фокус автора}\right|
$$

Где:

$$
\text{Pref}_{\text{category}} =
\frac{
\text{AvgClipsReach}
}{
\text{AvgClipsReach} + \text{AvgPostsReach}
}$$

$$
\text{Focus}_{\text{author}} =
\frac{
\text{clipsReach30d}
}{
\text{clipsReach30d} + \text{postsReach30d} + \epsilon
}
$$

$\text{Pref}_{\text{category}}$ — «Предпочтение категории»,  

$\text{Focus}_{\text{author}}$ — «Фокус автора».

---

## 2. $W_{\text{Quality}}$ (вес качества автора)

$$
W_{\text{Quality}} =
(k_S \cdot \text{Score}_{\text{Segment}}) +
(k_O \cdot \text{Score}_{\text{Original}}) +
(k_E \cdot \text{Score}_{\text{Expert}})
$$


### ScoreSegment (сегмент автора по качеству аудитории)

* 1 → 1.0
* 2 → 0.8
* 3 → 0.6
* 4 → 0.4
* 5 → 0.2
* 6 → 0.1

### ScoreOriginal (тип автора: оригинальный создатель или агрегатор)

* creator → 1.0
* aggregator → 0.7
* not_creator_or_aggregator → 0.3

### ScoreExpert (уровень экспертности автора)

* prof → 1.0
* pugc → 0.7
* ugc → 0.4

---

##$ 3. $P_{\text{Health}}$ (коэффициент качествf оформления контента автором)

Поскольку у автора есть **категориальная** оценка качества контента (`superlike`, `like`, `unlike`, `zhest`), а не распределение реакций, вводится фиксированный вес:


$$
P_{\text{Health}} = \text{Score}_{\text{Health}}
$$



| Статус    | ScoreHealth | Логика                   |
| --------- | ----------- | ------------------------ |
| superlike | 1.20        | бонус за высшее качество |
| like      | 1.00        | нейтрально               |
| unlike    | 0.50        | значимый штраф           |
| zhest     | 0.10        | максимальный штраф       |

---

## 4. $W_{\text{Engagement}}$ (вес реальной вовлечённости)

$$
W_{\text{Engagement}} =
1 + k_{\text{ER}} \cdot \text{ER}_{\text{Author}}
$$

$$
\text{ER}_{\text{Author}} =
\frac{
\sum (\text{Likes} + \text{Comments} + \text{Reposts})
}{
\sum \text{Reach} + \epsilon
}
$$

Суммирование ведётся по постам и клипам за 30 дней.

---

## 5. $B_{\text{Trend}}$ (коэффициент трендовости и сложности категории)

Компонент разделён на три части: актуальность просмотров, сложность категории и новизна/низкая конкуренция.

$$
B_{\text{Trend}} =
(\beta_V \cdot \text{Score}_{\text{Views}}) +
(\beta_D \cdot \text{Score}_{\text{Difficulty}}) +
(\beta_N \cdot \text{Bonus}_{\text{Novelty}})
$$

Где 
$$
\beta_V + \beta_D + \beta_N = 1
$$
---

## 5.1 $\text{Score}_{\text{Views}}$ (актуальность автора по просмотрам)

$$
\text{Score}_{\text{Views}} =
\frac{
\text{postsViews30d} + \text{clipsViews30d}
}{
\text{avgCategoryViews} + \epsilon
}
$$




Если автор выше среднего по просмотрам — множитель > 1.

---

## 5.2 $\text{Score}_{\text{Difficulty}}$ (сложность категории)

Основана на среднем ER в категории товара.

$$
\text{Score}_{\text{Difficulty}} =
\left(
\frac{
ER_{\text{GlobalAvg}}
}{
ER_{\text{AvgCategory}} + \epsilon
}
\right)^{k_{\text{difficulty}}}
$$


Если ER в категории низкий, категория считается трудной — множитель растёт (>1).

---

## 5.3 $\text{Bonus}_{\text{Novelty}}$ (бонус за новизну / низкую конкуренцию)

Конкуренция аппроксимируется плотностью охвата — средним reach на автора.

$$
\text{Bonus}_{\text{Novelty}} =
\left(
\frac{
\text{Средний Reach на Автора}_{\text{Global}}
}{
\text{Средний Reach на Автора}_{\text{Category}} + \epsilon
}
\right)^{k_{\text{novelty}}}
$$

Если reach на одного автора в категории низкий, ниша менее занята — множитель > 1.

