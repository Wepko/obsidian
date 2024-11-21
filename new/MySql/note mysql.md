id: 1
 select_type: SIMPLE
 table: film_actor
 partitions: NULL
==type: ref==
 possible_keys: idx_fk_film_id
 key: idx_fk_film_id
 key_len: 2
 ref: const
 rows: 10
 filtered: 100.00
 Extra: NULL


id: 1
 select_type: SIMPLE
 table: film_actor
 partitions: NULL
==type: ALL==
 possible_keys: NULL
 key: NULL
 key_len: NULL
 ref: NULL
 rows: 5462
 filtered: 10.00
 Extra: Using where
1 row in set, 1 warning (0.00 sec


