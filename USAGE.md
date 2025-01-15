# Using the data in Multi-CAST datasets


## refind

Multi-CAST corpora often include annotations according to the *refind* specification. This allows
associating glossed words in the corpus with referent metadata. Retrieving glosses and referent metadata
requires querying multiple tables in a dataset and, thus, is best done using the SQLite database version of
a dataset.

Below is a query retrieving all words (and glosses) associated with referents classified as `hum` in the Bora
corpus (v2311):

```sql
SELECT e.cid, e.word, e.gloss, r.cldf_name, r.cldf_description
FROM
(SELECT word.cid, word.word as word, gloss.word as gloss, refind.word as refind
FROM (
    SELECT row_number() over (order by cid) as rn, word, cid
    FROM (
        WITH split(word, clause, cid) AS (
            SELECT '', f.cldf_analyzedWord || X'09', f.cldf_id
            FROM ExampleTable AS f
            UNION ALL
            SELECT
                substr(clause, 0, instr(clause, X'09')),
                substr(clause, instr(clause, X'09') + 1),
                cid
            FROM split WHERE clause != ''
        )
        SELECT word, cid FROM split where word != '')
) AS word,
(
    SELECT row_number() over (order by cid) as rn, word, cid
    FROM (
        WITH split(word, clause, cid) AS (
            SELECT '', f.cldf_gloss || X'09', f.cldf_id
            FROM ExampleTable AS f
            UNION ALL
            SELECT
                substr(clause, 0, instr(clause, X'09')),
                substr(clause, instr(clause, X'09') + 1),
                cid
            FROM split WHERE clause != ''
        )
        SELECT word, cid FROM split where word != '')
) AS gloss,
(
    SELECT row_number() over (order by cid) as rn, word, cid
    FROM (
        WITH split(word, clause, cid) AS (
            SELECT '', f.refind || X'09', f.cldf_id
            FROM ExampleTable AS f
            UNION ALL
            SELECT
                substr(clause, 0, instr(clause, X'09')),
                substr(clause, instr(clause, X'09') + 1),
                cid
            FROM split WHERE clause != ''
        )
        SELECT word, cid FROM split where word != '')
) AS refind
WHERE word.rn = gloss.rn AND word.cid = gloss.cid AND word.rn = refind.rn AND word.cid = refind.cid
ORDER BY word.cid, word.rn) as e
JOIN `referents.csv` as r on e.refind = r.cldf_id
WHERE r.class = 'hum';
```
                       
