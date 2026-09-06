# DATOS-DE-PELICULAS-Y-SERIES-DE-NETLIX-ANALIZADOS-POR-SQL.
DATOS DE PELICULAS Y SERIES DE NETLIX ANALIZADOS POR SQL.

Resumen

Este proyecto implica un análisis exhaustivo de los datos de películas y series de Netflix utilizando SQL. El objetivo es extraer información valiosa y responder a diversas preguntas empresariales basadas en el conjunto de datos. El siguiente README ofrece un relato detallado de los objetivos del proyecto, problemas de negocio, soluciones, hallazgos y conclusiones.

Objetivos

Analiza la distribución de los tipos de contenido (películas vs series de televisión).
Identifica las valoraciones más comunes para películas y series de televisión.
Lista y analiza el contenido según años de lanzamiento, países y duraciones.
Explora y categoriza el contenido según criterios y palabras clave específicas.

Conjunto de datos

Los datos de este proyecto provienen del conjunto de datos de Kaggle:

Enlace al conjunto de datos: Conjunto de datos de películas
Esquema
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);
Problemas y soluciones empresariales
1. Contar el número de películas frente a series de televisión
SELECT 
    type,
    COUNT(*)
FROM netflix
GROUP BY 1;
Objetivo: Determina la distribución de los tipos de contenido en Netflix.

2. Encontrar la valoración más común para películas y series de televisión
WITH RatingCounts AS (
    SELECT 
        type,
        rating,
        COUNT(*) AS rating_count
    FROM netflix
    GROUP BY type, rating
),
RankedRatings AS (
    SELECT 
        type,
        rating,
        rating_count,
        RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rank
    FROM RatingCounts
)
SELECT 
    type,
    rating AS most_frequent_rating
FROM RankedRatings
WHERE rank = 1;
Objetivo: Identifica la valoración que aparece con más frecuencia para cada tipo de contenido.

3. Listar todas las películas estrenadas en un año específico (por ejemplo, 2020)
SELECT * 
FROM netflix
WHERE release_year = 2020;
Objetivo: Recupera todas las películas estrenadas en un año específico.

4. Encuentra los 5 países con más contenido en Netflix
SELECT * 
FROM
(
    SELECT 
        UNNEST(STRING_TO_ARRAY(country, ',')) AS country,
        COUNT(*) AS total_content
    FROM netflix
    GROUP BY 1
) AS t1
WHERE country IS NOT NULL
ORDER BY total_content DESC
LIMIT 5;
Objetivo: Identifica los 5 países con el mayor número de elementos de contenido.

5. Identificar la película más larga
SELECT 
    *
FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC;
Objetivo: Encuentra la película con la duración más larga.

6. Encontrar contenido añadido en los últimos 5 años
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
Objetivo: Recuperar contenido añadido a Netflix en los últimos 5 años.

7. Encuentra todas las películas/series del director 'Rajiv Chilaka'
SELECT *
FROM (
    SELECT 
        *,
        UNNEST(STRING_TO_ARRAY(director, ',')) AS director_name
    FROM netflix
) AS t
WHERE director_name = 'Rajiv Chilaka';
Objetivo: Lista de todo el contenido dirigido por 'Rajiv Chilaka'.

8. Listar todas las series de televisión con más de 5 temporadas
SELECT *
FROM netflix
WHERE type = 'TV Show'
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;
Objetivo: Identifica series de televisión con más de 5 temporadas.

9. Contar el número de elementos de contenido en cada género
SELECT 
    UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre,
    COUNT(*) AS total_content
FROM netflix
GROUP BY 1;
Objetivo: Cuenta el número de elementos de contenido de cada género.

10. Encuentra cada año y el número medio de contenido lanzado en India en Netflix.
¡Volver al top 5 del año con mayor promedio de contenido lanzado!

SELECT 
    country,
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        COUNT(show_id)::numeric /
        (SELECT COUNT(show_id) FROM netflix WHERE country = 'India')::numeric * 100, 2
    ) AS avg_release
FROM netflix
WHERE country = 'India'
GROUP BY country, release_year
ORDER BY avg_release DESC
LIMIT 5;
Objetivo: Calcula y clasifica los años según el número medio de publicaciones de contenido en India.

11. Listar todas las películas que son documentales
SELECT * 
FROM netflix
WHERE listed_in LIKE '%Documentaries';
Objetivo: Recupera todas las películas clasificadas como documentales.

12. Encontrar todo el contenido sin director
SELECT * 
FROM netflix
WHERE director IS NULL;
Objetivo: Incluye contenido que no tenga director.

13. Descubre cuántas películas ha aparecido el actor 'Salman Khan' en los últimos 10 años
SELECT * 
FROM netflix
WHERE casts LIKE '%Salman Khan%'
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
Objetivo: Cuenta el número de películas protagonizadas por 'Salman Khan' en los últimos 10 años.

14. Encuentra los 10 mejores actores que han aparecido en el mayor número de películas producidas en la India
SELECT 
    UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor,
    COUNT(*)
FROM netflix
WHERE country = 'India'
GROUP BY actor
ORDER BY COUNT(*) DESC
LIMIT 10;
Objetivo: Identifica a los 10 actores con más apariciones en películas producidas en India.

15. Categorizar el contenido en función de la presencia de palabras clave 'matar' y 'violencia'
SELECT 
    category,
    COUNT(*) AS content_count
FROM (
    SELECT 
        CASE 
            WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix
) AS categorized_content
GROUP BY category;
Objetivo: Categoriza el contenido como 'Malo' si contiene 'matar' o 'violencia' y 'Bueno' en caso contrario. Cuenta el número de elementos en cada categoría.

Hallazgos y conclusión
Distribución del contenido: El conjunto de datos contiene una amplia variedad de películas y series de televisión con distintas clasificaciones y géneros.
Valoraciones comunes: Las información sobre las valoraciones más comunes proporcionan una idea del público objetivo del contenido.
Perspectivas geográficas: Los principales países y la media de publicaciones de contenido en India destacan la distribución regional de contenido.
Categorización de contenido: Categorizar el contenido en función de palabras clave específicas ayuda a entender la naturaleza del contenido disponible en Netflix.
Este análisis ofrece una visión completa del contenido de Netflix y puede ayudar a informar la estrategia y la toma de decisiones de contenido.
