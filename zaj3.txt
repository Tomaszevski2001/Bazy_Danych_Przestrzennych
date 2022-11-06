
-- cw 1
-- Znajd budynki, które zosta³y wybudowane
-- lub wyremontowane na przestrz.geomeni r.geomoku (zmianapomiêdzy 2018 a 2019).

CREATE VIEW BuildingsFromTask1 AS
SELECT b19.* FROM t2018_kar_buildings AS b18, t2019_kar_buildings AS b19
WHERE ST_Equals(b18.geom, b19.geom) = false AND
	b19.polygon_id = b18.polygon_id;

-- cw 2
-- Znajd ile nowych POI pojawi³o siê w promieniu 500 m od wyremontowanych lub wybudowanych budynków,
-- które znalezione zosta³y w zadaniu 1. Policz je wg ich kategorii.

SELECT * FROM T2019_KAR_POI_TABLE;

SELECT COUNT(DISTINCT(Poi19.poi_id)) FROM
	BuildingsFromTask1 AS TFT1,
	T2019_KAR_POI_TABLE AS Poi19
WHERE
	Poi19.gid NOT IN (
		SELECT DISTINCT(Poi19.gid) FROM
			T2019_KAR_POI_TABLE AS Poi19,
			T2018_KAR_POI_TABLE AS Poi18
		WHERE 
		ST_Equals(Poi19.geom, Poi18.geom)
	)
	AND
	ST_DWithin(Poi19.geom, TFT1.geom, 500);
	
	
-- cw 3
CREATE TABLE t2019_kar_streets_Berlin_Cassini AS(
	SELECT gid, link_id, st_name, ref_in_id, nref_in_id,
	func_class, speed_cat, fr_speed_l, to_speed_l, dir_travel, 
	ST_Transform(geom, 3068)
	FROM t2019_kar_streets
)

SELECT Nor.geom, Tr.st_transform FROM t2019_kar_streets_Berlin_Cassini AS Tr, t2019_kar_streets AS Nor;


-- cw 4
-- Stwórz tabelę o nazwie ‘input_points’ i dodaj do niej dwa rekordy o geometrii punktowej.
-- Użyj następujących współrzędnych:
-- X		Y
-- 8.36093  49.03174
-- 8.39876  49.00644
-- Przyjmij układ współrzędnych GPS.

CREATE TABLE input_points (
	p_id INT PRIMARY KEY,
	geom GEOMETRY(POINT, 4326)
);


INSERT INTO input_points (p_id, geom)
VALUES 
	(1, ST_GeomFromText('POINT(8.36093 49.03174)', 4326)),
	(2, ST_GeomFromText('POINT(8.39876 49.00644)', 4326));
		

-- 5) Zaktualizuj dane w tabeli ‘input_points’ tak, aby punkty te były w układzie współrzędnych
-- DHDN.Berlin/Cassini. Wyświetl współrzędne za pomocą funkcji ST_AsText().

UPDATE input_points SET geom = ST_Transform(ST_SetSRID(geom, 4326), 3068);
ALTER TABLE input_points ALTER COLUMN geom TYPE geometry(POINT, 3068);

SELECT ST_AsText(geom) FROM input_points;


-- 6) Znajdź wszystkie skrzyżowania, które znajdują się w odległości 200 m od linii zbudowanej
-- z punktów w tabeli ‘input_points’. Wykorzystaj tabelę T2019_STREET_NODE.
-- Dokonajreprojekcji geometrii, aby była zgodna z resztą tabel.

SELECT * FROM T2019_KAR_STREET_NODE AS node
WHERE
ST_DWithin(ST_Transform(node.geom, 3068), (SELECT ST_MakeLine(pi.geom) FROM input_points AS pi), 200)
AND node.intersect = 'Y';


-- 7) Policz jak wiele sklepów sportowych (‘Sporting Goods Store’ - tabela POIs)
-- znajduje sięw odległości 300 m od parków (LAND_USE_A).

SELECT COUNT(DISTINCT(Spot.geom)) FROM T2019_KAR_LAND_USE_A AS Land, T2019_KAR_POI_TABLE AS Spot
WHERE 
Spot.type = 'Sporting Goods Store'
AND ST_DWithin(Land.geom, Spot.geom, 300) 
AND Land.type = 'Park (City/County)';


-- 8) Znajdź punkty przecięcia torów kolejowych (RAILWAYS) z ciekami (WATER_LINES). 
-- Zapiszznalezioną geometrię do osobnej tabeli o nazwie ‘T2019_KAR_BRIDGES’.

CREATE TABLE T2019_KAR_BRIDGES AS
(
	SELECT DISTINCT(ST_Intersection(r.geom, w.geom))
	FROM t2019_kar_railways AS r,t2019_kar_water_lines AS w
)

SELECT * FROM T2019_KAR_BRIDGES;




	