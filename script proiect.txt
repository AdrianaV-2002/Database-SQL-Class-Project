---creare tabele

CREATE TABLE autori (
    id_autor NUMBER PRIMARY KEY,
    nume_autor VARCHAR2(100) NOT NULL
);

CREATE TABLE categorii (
    id_categorie NUMBER PRIMARY KEY,
    nume_categorie VARCHAR2(50) NOT NULL
);

CREATE TABLE carti (
    id_carte NUMBER PRIMARY KEY,
    titlu VARCHAR2(200) NOT NULL,
    id_autor NUMBER,
    id_categorie NUMBER,
    CONSTRAINT fk_autor FOREIGN KEY (id_autor) REFERENCES autori(id_autor),
    CONSTRAINT fk_categorie FOREIGN KEY (id_categorie) REFERENCES categorii(id_categorie)
);

CREATE TABLE cititori (
    id_cititor NUMBER PRIMARY KEY,
    nume VARCHAR2(100) NOT NULL,
    email VARCHAR2(100) UNIQUE
);

CREATE TABLE imprumuturi (
    id_imprumut NUMBER PRIMARY KEY,
    id_carte NUMBER,
    id_cititor NUMBER,
    data_imprumut DATE,
    data_returnare DATE,
    CONSTRAINT fk_carte FOREIGN KEY (id_carte) REFERENCES carti(id_carte),
    CONSTRAINT fk_cititor FOREIGN KEY (id_cititor) REFERENCES cititori(id_cititor) ON DELETE CASCADE; -- permite stergerea cititorului si a inprumuturilor care depind de acesta
);



--adaugare coloana
ALTER TABLE cititori ADD telefon VARCHAR2(20);

--redenumire
RENAME carti TO carti_biblioteca;

--verificarea crearii tabelelor
SELECT table_name FROM user_tables;

--verificarea constrangerilor existente ale unui tabel
SELECT constraint_name, constraint_type, table_name
FROM user_constraints
WHERE table_name = 'CARTI_BIBLIOTECA';


--Alte obiecte: view sinonim, index


CREATE VIEW vw_imprumuturi_active AS
SELECT c.titlu, ci.nume, i.data_imprumut
FROM imprumuturi i
JOIN carti_biblioteca c ON i.id_carte = c.id_carte
JOIN cititori ci ON i.id_cititor = ci.id_cititor
WHERE i.data_returnare IS NULL;

CREATE INDEX idx_titlu ON carti_biblioteca(titlu);

CREATE SYNONYM carti FOR carti_biblioteca;

SELECT object_name, object_type FROM user_objects;



--Prelucrarea datelor:

-- Inserare date pentru AUTORI
INSERT ALL
    INTO autori (id_autor, nume_autor) VALUES (1, 'Mihai Eminescu')
    INTO autori (id_autor, nume_autor) VALUES (2, 'Ion Creangă')
    INTO autori (id_autor, nume_autor) VALUES (3, 'Gabriel Garcia Marquez')
    INTO autori (id_autor, nume_autor) VALUES (4, 'J.K. Rowling')
    INTO autori (id_autor, nume_autor) VALUES (5, 'George Orwell')
SELECT * FROM dual;

-- categorii
INSERT ALL
    INTO categorii (id_categorie, nume_categorie) VALUES (1, 'Literatura Română')
    INTO categorii (id_categorie, nume_categorie) VALUES (2, 'Literatura Străină')
    INTO categorii (id_categorie, nume_categorie) VALUES (3, 'Fantezie')
    INTO categorii (id_categorie, nume_categorie) VALUES (4, 'Science Fiction')
    INTO categorii (id_categorie, nume_categorie) VALUES (5, 'Poezie')
SELECT * FROM dual;

-- cititori
INSERT ALL
    INTO cititori (id_cititor, nume, email, telefon) VALUES (1, 'Ion Popescu', 'ion.popescu@gmail.com', '0722333444')
    INTO cititori (id_cititor, nume, email, telefon) VALUES (2, 'Maria Ionescu', 'maria.ionescu@gmail.com', '0733555666')
    INTO cititori (id_cititor, nume, email, telefon) VALUES (3, 'Andrei Vasilescu', 'andrei.vasilescu@gmail.com', '0744666777')
    INTO cititori (id_cititor, nume, email, telefon) VALUES (4, 'Elena Georgescu', 'elena.georgescu@gmail.com', '0755777888')
SELECT * FROM dual;

-- carti_biblioteca
INSERT ALL
    INTO carti_biblioteca (id_carte, titlu, id_autor, id_categorie) VALUES (1, 'Poezii', 1, 5)
    INTO carti_biblioteca (id_carte, titlu, id_autor, id_categorie) VALUES (2, 'Amintiri din copilarie', 2, 1)
    INTO carti_biblioteca (id_carte, titlu, id_autor, id_categorie) VALUES (3, 'Un veac de singurătate', 3, 2)
    INTO carti_biblioteca (id_carte, titlu, id_autor, id_categorie) VALUES (4, 'Harry Potter și Piatra Filozofală', 4, 3)
    INTO carti_biblioteca (id_carte, titlu, id_autor, id_categorie) VALUES (5, '1984', 5, 4)
    INTO carti_biblioteca (id_carte, titlu, id_autor, id_categorie) VALUES (6, 'Capra cu trei iezi', 2, 1)
    
SELECT * FROM dual;

--imprumuturi

INSERT ALL
    INTO imprumuturi (id_imprumut, id_carte, id_cititor, data_imprumut, data_returnare) VALUES (1, 1, 1, DATE '2026-01-01', DATE '2026-01-10')
    INTO imprumuturi (id_imprumut, id_carte, id_cititor, data_imprumut, data_returnare) VALUES (2, 2, 2, DATE '2026-01-05', NULL)
    INTO imprumuturi (id_imprumut, id_carte, id_cititor, data_imprumut, data_returnare) VALUES (3, 3, 3, DATE '2025-12-20', DATE '2025-12-30')
    INTO imprumuturi (id_imprumut, id_carte, id_cititor, data_imprumut, data_returnare) VALUES (4, 4, 1, DATE '2026-01-08', NULL)
    INTO imprumuturi (id_imprumut, id_carte, id_cititor, data_imprumut, data_returnare) VALUES (5, 5, 4, DATE '2025-12-25', DATE '2026-01-05')
    INTO imprumuturi (id_imprumut, id_carte, id_cititor, data_imprumut, data_returnare) VALUES (6, 4, 4, DATE '2026-01-08', NULL)
SELECT * FROM dual;




--folosim sinonimul "carti" asignat pentru "carti_biblioteca"
UPDATE carti
SET titlu = 'Poezii Complete'
WHERE id_carte = 1;


---stergere

DELETE FROM cititori WHERE id_cititor = 1; -- stergere cititor si inprumuturile corespunzatoare lui

UPDATE imprumuturi -- actualizare data returnare carti
SET data_returnare =  DATE '2026-02-15'
WHERE id_imprumut = 2;



-- aceasta comanda actualizeaza un rand daca exista deja sau introduce unul nou daca nu exista
MERGE INTO carti_biblioteca c
USING (SELECT 1 id_carte, 'Poezii Editate' titlu FROM dual) s
ON (c.id_carte = s.id_carte)
WHEN MATCHED THEN
  UPDATE SET c.titlu = s.titlu
WHEN NOT MATCHED THEN
  INSERT (id_carte, titlu) VALUES (s.id_carte, s.titlu);


--Integrari complexe

--afisarea cartilor neimprumutate
SELECT c.titlu
FROM carti_biblioteca c
LEFT JOIN imprumuturi i ON c.id_carte = i.id_carte
WHERE i.id_imprumut IS NULL;


-- selectare cititori care au imprumuturi nereturnate
SELECT DISTINCT ci.nume
FROM cititori ci
JOIN imprumuturi i ON ci.id_cititor = i.id_cititor
WHERE i.data_returnare IS NULL;


-- selectarea cartilor imprumutate in ultimele 30 de zile
SELECT c.titlu, i.data_imprumut
FROM carti_biblioteca c
JOIN imprumuturi i ON c.id_carte = i.id_carte
WHERE i.data_imprumut >= SYSDATE - 30;


-- numarul de carti dintr-o categorie

SELECT cat.nume_categorie, COUNT(c.id_carte) nr_carti
FROM categorii cat
JOIN carti_biblioteca c ON cat.id_categorie = c.id_categorie
GROUP BY cat.nume_categorie
HAVING COUNT(c.id_carte) > 0;
