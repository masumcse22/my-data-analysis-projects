# 🎨 SQL Masterpieces: An Analysis of Famous Paintings

This project delves into a comprehensive dataset of famous paintings, artists, and museums. Using SQL, we explore the relationships between these entities, uncover interesting facts, and derive key insights from the data.

## 🎯 Project Objective

The primary goal is to query a relational database to answer 22 specific questions, demonstrating proficiency in SQL for data analysis, including joins, aggregations, subqueries, and window functions.

---

## 🗃️ Dataset Overview

The analysis is performed on a relational database consisting of eight interconnected tables.

| Icon | Table Name     | Description                                                           |
| :--- | :------------- | :-------------------------------------------------------------------- |
| 🧑‍🎨   | `artist`       | Contains biographical information about the artists.                  |
| 🖼️   | `canvas_size`  | Provides details about different canvas dimensions.                   |
| 🔗   | `image_link`   | Stores URLs to the images of the paintings.                           |
| 🏛️   | `museum`       | Holds information about museums, including their location.            |
| 🕒   | `museum_hours` | Contains the opening and closing times for museums on different days. |
| 🎭   | `subject`      | Describes the subject matter or genre of the paintings.               |
| 🖌️   | `work`         | The central table, containing details about each individual painting. |
| 💰   | `product_size` | Lists the prices for different sizes of prints for each painting.     |

---

## ❓ SQL Queries & Analysis

Here are the 22 analytical queries performed on the dataset, along with their results.

### 1. Which paintings are not displayed in any museum?

_This query identifies artworks that are not associated with a museum._

```sql
SELECT *
FROM work
WHERE museum_id IS NULL;
```

**Insight:** There are **10,223 paintings** in the dataset that are not currently displayed in any museum.

---

### 2. Are there any museums without paintings?

_This query checks for museums that have no artworks listed in the `work` table._

```sql
SELECT m.*
FROM museum m
LEFT JOIN work w ON m.museum_id = w.museum_id
WHERE w.museum_id IS NULL;
```

**Insight:** All museums in the dataset have at least one painting associated with them.

---

### 3. How many paintings have a sale price higher than their regular price?

_This query counts the number of products with a sale price exceeding the regular price._

```sql
SELECT COUNT(*)
FROM product_size
WHERE sale_price > regular_price;
```

**Insight:** There are **zero paintings** with a sale price greater than their regular price.

---

### 4. Which paintings have a sale price less than 50% of their regular price?

_This query identifies paintings with a significant discount._

```sql
SELECT COUNT(DISTINCT work_id)
FROM product_size
WHERE sale_price < (regular_price * 0.5);
```

**Insight:** There are **5 paintings** whose asking price is less than half of their regular price.

---

### 5. Which canvas size costs the most?

_This query finds the canvas with the highest sale price._

```sql
SELECT
    cs.label,
    ps.sale_price
FROM product_size ps
JOIN canvas_size cs ON cs.size_id = ps.size_id
ORDER BY ps.sale_price DESC
LIMIT 1;
```

**Result:**
| Label | Sale Price |
| :--- | :--- |
| 48" x 96" (122 cm x 244 cm) | 1115 |

---

### 6. How are duplicate records deleted?

_These queries demonstrate how to remove duplicate entries from various tables._

```sql
-- For MySQL, using a window function in a subquery to identify duplicates
DELETE FROM work
WHERE work_id IN (
  SELECT work_id FROM (
    SELECT work_id, ROW_NUMBER() OVER (PARTITION BY work_id ORDER BY work_id) as rn
    FROM work
  ) x WHERE rn > 1
);

-- Similar logic applies to other tables, partitioning by their unique composite keys.
DELETE FROM product_size
WHERE (work_id, size_id) IN (
  SELECT work_id, size_id FROM (
    SELECT work_id, size_id, ROW_NUMBER() OVER (PARTITION BY work_id, size_id ORDER BY work_id) as rn
    FROM product_size
  ) x WHERE rn > 1
);
```

**Insight:** Duplicate records were successfully identified and removed from the `work`, `product_size`, `subject`, and `image_link` tables to ensure data integrity.

---

### 7. Which museums have invalid city information?

_This query finds museums where the city name appears to be a postal code._

```sql
-- Note: REGEXP is MySQL syntax. PostgreSQL uses '~'.
SELECT *
FROM museum
WHERE city REGEXP '^[0-9]';
```

**Insight:** Six museums were identified with numeric, invalid city names, indicating data quality issues that need to be addressed.

---

### 8. How is an invalid entry removed from `museum_hours`?

_This query identifies and removes duplicate entries for a museum's daily hours._

```sql
-- This query deletes rows where the combination of museum_id and day is duplicated.
DELETE FROM museum_hours
WHERE (museum_id, day) IN (
  SELECT museum_id, day FROM (
    SELECT museum_id, day, ROW_NUMBER() OVER (PARTITION BY museum_id, day ORDER BY museum_id) as rn
    FROM museum_hours
  ) x WHERE rn > 1
);
```

**Insight:** A duplicate entry in the `museum_hours` table was identified and removed.

---

### 9. What are the top 10 most famous painting subjects?

_This query ranks painting subjects by their frequency._

```sql
SELECT
    s.subject,
    COUNT(*) AS no_of_paintings
FROM work w
JOIN subject s ON s.work_id = w.work_id
GROUP BY s.subject
ORDER BY no_of_paintings DESC
LIMIT 10;
```

**Result:**
| Subject | No. of Paintings |
| :--- | :--- |
| Portraits | 1070 |
| Nude | 525 |
| Landscape Art | 495 |
| Rivers/Lakes | 480 |
| Flowers | 457 |
| Still-Life | 395 |
| Seascapes | 326 |
| Marine Art/Maritime | 268 |
| Horses | 265 |
| Abstract/Modern Art | 575 |

---

### 10. Which museums are open on both Sunday and Monday?

_This query identifies museums that operate on both Sunday and Monday._

```sql
SELECT m.name, m.city
FROM museum m
WHERE EXISTS (SELECT 1 FROM museum_hours WHERE museum_id = m.museum_id AND day = 'Sunday')
  AND EXISTS (SELECT 1 FROM museum_hours WHERE museum_id = m.museum_id AND day = 'Monday');
```

**Insight:** The query successfully identifies all museums that are confirmed to be open on both of these days.

---

### 11. How many museums are open every day?

_This query counts the number of museums open 7 days a week._

```sql
SELECT COUNT(*)
FROM (
    SELECT museum_id
    FROM museum_hours
    GROUP BY museum_id
    HAVING COUNT(DISTINCT day) = 7
) x;
```

**Insight:** There are **18 museums** that are open every day of the week.

---

### 12. What are the top 5 most popular museums?

_Popularity is defined by the number of paintings hosted._

```sql
SELECT
    m.name,
    COUNT(w.work_id) AS painting_count
FROM work w
JOIN museum m ON w.museum_id = m.museum_id
GROUP BY m.name
ORDER BY painting_count DESC
LIMIT 5;
```

**Result:**
| Name | Painting Count |
| :--- | :--- |
| The Metropolitan Museum of Art | 939 |
| Rijksmuseum | 452 |
| National Gallery | 423 |
| National Gallery of Art | 375 |
| The Barnes Foundation | 350 |

---

### 13. Who are the top 5 most popular artists?

_Popularity is defined by the number of paintings created._

```sql
SELECT
    a.full_name,
    COUNT(w.work_id) AS no_of_paintings
FROM work w
JOIN artist a ON w.artist_id = a.artist_id
GROUP BY a.full_name
ORDER BY no_of_paintings DESC
LIMIT 5;
```

**Result:**
| Full Name | No. of Paintings |
| :--- | :--- |
| Pierre-Auguste Renoir | 469 |
| Claude Monet | 378 |
| Vincent van Gogh | 308 |
| Maurice Utrillo | 253 |
| Albert Marquet | 233 |

---

### 14. What are the 3 least popular canvas sizes?

_Popularity is based on the number of paintings using a specific canvas size._

```sql
SELECT
    c.label,
    COUNT(w.work_id) AS work_count
FROM work w
JOIN product_size p ON w.work_id = p.work_id
JOIN canvas_size c ON c.size_id = p.size_id
GROUP BY c.label
ORDER BY work_count ASC
LIMIT 3;
```

**Result:**
| Label | Work Count |
| :--- | :--- |
| 37" x 30" (94 cm x 76 cm) | 1 |
| 32" x 18" (81 cm x 46 cm) | 1 |
| 46" x 35" (117 cm x 89 cm) | 1 |

---

### 15. Which museum is open for the longest on a single day?

_This query calculates the maximum duration a museum is open._

```sql
SELECT
    m.name,
    mh.day,
    TIMEDIFF(STR_TO_DATE(mh.close, '%h:%i:%p'), STR_TO_DATE(mh.open, '%h:%i:%p')) AS duration
FROM museum_hours mh
JOIN museum m ON m.museum_id = mh.museum_id
ORDER BY duration DESC
LIMIT 1;
```

**Insight:** The **Musée du Louvre** has the longest opening hours on a single day, staying open for **12 hours**.

---

### 16. Which museum has the most paintings of the most popular style?

_This query is a two-step analysis to find a concentration of popular art._

```sql
WITH popular_style AS (
    SELECT style
    FROM work
    GROUP BY style
    ORDER BY COUNT(1) DESC
    LIMIT 1
)
SELECT
    m.name,
    COUNT(1) AS painting_count
FROM work w
JOIN museum m ON m.museum_id = w.museum_id
WHERE w.style = (SELECT style FROM popular_style)
GROUP BY m.name
ORDER BY painting_count DESC
LIMIT 1;
```

**Insight:** **The Metropolitan Museum of Art** has the most paintings (**244**) in the most popular style, **Impressionism**.

---

### 17. Which artists have paintings displayed in multiple countries?

_This query identifies artists with an international presence._

```sql
SELECT
    a.full_name,
    COUNT(DISTINCT m.country) AS country_count
FROM work w
JOIN artist a ON a.artist_id = w.artist_id
JOIN museum m ON m.museum_id = w.museum_id
GROUP BY a.full_name
HAVING country_count > 1
ORDER BY country_count DESC;
```

**Insight:** **Vincent van Gogh** is the most international artist, with paintings displayed in **8 countries**.

---

### 18. Which country and city have the most museums?

_This query identifies the top locations for museums._

```sql
WITH country_ranking AS (
    SELECT country, RANK() OVER (ORDER BY COUNT(*) DESC) as rnk
    FROM museum GROUP BY country
),
city_ranking AS (
    SELECT city, RANK() OVER (ORDER BY COUNT(*) DESC) as rnk
    FROM museum GROUP BY city
)
SELECT
    (SELECT GROUP_CONCAT(country) FROM country_ranking WHERE rnk = 1) AS top_countries,
    (SELECT GROUP_CONCAT(city) FROM city_ranking WHERE rnk = 1) AS top_cities;
```

**Insight:** The **USA** has the most museums. **London, New York, Paris, and Washington** are tied for the cities with the most museums.

---

### 19. Where are the most and least expensive paintings located?

_This query finds the details of the highest and lowest-priced artworks._

```sql
(SELECT
    a.full_name AS artist,
    ps.sale_price,
    w.name AS painting,
    m.name AS museum
FROM product_size ps
JOIN work w ON ps.work_id = w.work_id
JOIN artist a ON w.artist_id = a.artist_id
JOIN museum m ON w.museum_id = m.museum_id
ORDER BY ps.sale_price DESC
LIMIT 1)
UNION ALL
(SELECT
    a.full_name,
    ps.sale_price,
    w.name,
    m.name
FROM product_size ps
JOIN work w ON ps.work_id = w.work_id
JOIN artist a ON w.artist_id = a.artist_id
JOIN museum m ON w.museum_id = w.museum_id
ORDER BY ps.sale_price ASC
LIMIT 1);
```

**Result:**

- **Most Expensive:** "Fortuna" by **Peter Paul Rubens** at **The Prado Museum** (Sale Price: 1115).
- **Least Expensive:** "Portrait of Madame Labille-Guiard and Her Pupils" by **Adélaïde Labille-Guiard** at **The Metropolitan Museum of Art** (Sale Price: 10).

---

### 20. Which country has the 5th highest number of paintings?

_This query ranks countries by their total number of paintings._

```sql
SELECT m.country, COUNT(*) AS no_of_paintings
FROM work w
JOIN museum m ON w.museum_id = w.museum_id
GROUP BY m.country
ORDER BY no_of_paintings DESC
LIMIT 1 OFFSET 4;
```

**Insight:** **Spain** ranks 5th with **196 paintings**.

---

### 21. What are the 3 most and 3 least popular painting styles?

_This query identifies the extremes in style popularity._

```sql
(SELECT style, 'Most Popular' as popularity FROM work WHERE style IS NOT NULL GROUP BY style ORDER BY COUNT(1) DESC LIMIT 3)
UNION ALL
(SELECT style, 'Least Popular' as popularity FROM work WHERE style IS NOT NULL GROUP BY style ORDER BY COUNT(1) ASC LIMIT 3);
```

**Result:**
| Style | Popularity |
| :--- | :--- |
| Impressionism | Most Popular |
| Post-Impressionism | Most Popular |
| Realism | Most Popular |
| Avant-Garde | Least Popular |
| Art Nouveau | Least Popular |
| Japanese Art | Least Popular |

---

### 22. Which artist has the most portraits outside the USA?

_This query identifies the leading non-US portrait artist._

```sql
SELECT
    a.full_name,
    a.nationality,
    COUNT(*) AS no_of_paintings
FROM work w
JOIN artist a ON a.artist_id = w.artist_id
JOIN subject s ON s.work_id = w.work_id
JOIN museum m ON m.museum_id = w.museum_id
WHERE s.subject = 'Portraits' AND m.country != 'USA'
GROUP BY a.full_name, a.nationality
ORDER BY no_of_paintings DESC
LIMIT 1;
```

**Insight:** The Dutch artists **Jan Willem Pieneman** and **Vincent van Gogh** are tied for the most portraits outside the USA, with **14** each.
