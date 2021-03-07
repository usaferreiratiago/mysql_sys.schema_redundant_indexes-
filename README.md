# mysql_sys.schema_redundant_indexes-

SELECT 
    `sys`.`redundant_keys`.`table_schema` AS `table_schema`,
    `sys`.`redundant_keys`.`table_name` AS `table_name`,
    `sys`.`redundant_keys`.`index_name` AS `redundant_index_name`,
    `sys`.`redundant_keys`.`index_columns` AS `redundant_index_columns`,
    `sys`.`redundant_keys`.`non_unique` AS `redundant_index_non_unique`,
    `sys`.`dominant_keys`.`index_name` AS `dominant_index_name`,
    `sys`.`dominant_keys`.`index_columns` AS `dominant_index_columns`,
    `sys`.`dominant_keys`.`non_unique` AS `dominant_index_non_unique`,
    IF(((0 <> `sys`.`redundant_keys`.`subpart_exists`)
            OR (0 <> `sys`.`dominant_keys`.`subpart_exists`)),
        1,
        0) AS `subpart_exists`,
    CONCAT('ALTER TABLE `',
            `sys`.`redundant_keys`.`table_schema`,
            '`.`',
            `sys`.`redundant_keys`.`table_name`,
            '` DROP INDEX `',
            `sys`.`redundant_keys`.`index_name`,
            '`') AS `sql_drop_index`
FROM
    (`sys`.`x$schema_flattened_keys` `redundant_keys`
    JOIN `sys`.`x$schema_flattened_keys` `dominant_keys` ON (((`sys`.`redundant_keys`.`table_schema` = `sys`.`dominant_keys`.`table_schema`)
        AND (`sys`.`redundant_keys`.`table_name` = `sys`.`dominant_keys`.`table_name`))))
WHERE
    ((`sys`.`redundant_keys`.`index_name` <> `sys`.`dominant_keys`.`index_name`)
        AND (((`sys`.`redundant_keys`.`index_columns` = `sys`.`dominant_keys`.`index_columns`)
        AND ((`sys`.`redundant_keys`.`non_unique` > `sys`.`dominant_keys`.`non_unique`)
        OR ((`sys`.`redundant_keys`.`non_unique` = `sys`.`dominant_keys`.`non_unique`)
        AND (IF((`sys`.`redundant_keys`.`index_name` = 'PRIMARY'),
        '',
        `sys`.`redundant_keys`.`index_name`) > IF((`sys`.`dominant_keys`.`index_name` = 'PRIMARY'),
        '',
        `sys`.`dominant_keys`.`index_name`)))))
        OR ((LOCATE(CONCAT(`sys`.`redundant_keys`.`index_columns`,
                    ','),
            `sys`.`dominant_keys`.`index_columns`) = 1)
        AND (`sys`.`redundant_keys`.`non_unique` = 1))
        OR ((LOCATE(CONCAT(`sys`.`dominant_keys`.`index_columns`,
                    ','),
            `sys`.`redundant_keys`.`index_columns`) = 1)
        AND (`sys`.`dominant_keys`.`non_unique` = 0))))
