  # DATA Preview 
  # SELECT * FROM `bigquery-public-data.github_repos.sample_contents` LIMIT 10
WITH
  lines_of_code AS (
  SELECT
    SPLIT(content,'\n') AS line, # BREAK code INTO an ARRAY OF individual line sample_repo_name,
    sample_path
  FROM  `bigquery-public-data.github_repos.sample_contents`),

  # Preview progress 
  # SELECT * FROM  lines_of_code LIMIT  10 
  
  flattened_code AS (
  SELECT
    sample_repo_name,
    sample_path,
    flattened_line
  FROM
    lines_of_code, UNNEST(line) AS flattened_line),
  # Preview progress 
  # SELECT  * FROM  flattened_code LIMIT  10 
  
  extract_first_charater AS (
  SELECT
    SUBSTR(flattened_line, 1, 1) AS first_character,
    flattened_line,
    sample_path,
    sample_repo_name
  FROM
    flattened_code),
  # Preview progress 
  #SELECT  * FROM  extract_first_charater LIMIT  10 
  
  tab_and_space AS (
  SELECT
    first_character,
  IF
    (REGEXP_CONTAINS(first_character, '\t'),1,0) AS tab_count,
  IF
    (REGEXP_CONTAINS(first_character, ' '),1,0) AS space_count,
    flattened_line,
    sample_path,
    sample_repo_name
  FROM
    extract_first_charater
  WHERE
    REGEXP_CONTAINS(first_character, '[ \t]')),
  # Preview progress 
  # SELECT  * FROM  tab_and_space LIMIT  10 
  
  # aggregation OF charaters count
  initial_aggregation AS (
  SELECT 
    COUNT(flattened_line) AS lines, 
    SUM(tab_count) AS tab_count, 
    SUM(space_count) AS space_count, 
  IF
    (SUM(tab_count) > SUM(space_count), 1,0) AS tab_wins,
  IF
    (SUM(tab_count) < SUM(space_count), 1,0) AS space_wins, 
    sample_path, 
    sample_repo_name,
    REGEXP_EXTRACT(sample_path, r'\.([^\.]*)$') AS extension
  FROM 
    tab_and_space
  GROUP BY 
    sample_path, 
    sample_repo_name
),
# Preview progress 
# SELECT  * FROM  initial_aggregation LIMIT 100 

# Final table
tab_and_space_for_each_extensions AS (
  SELECT 
    extension, 
    COUNT(extension) AS files_count, 
    SUM(lines) AS lines, 
    SUM(tab_count) AS tab_count, 
    SUM(space_count) AS space_count,
    SUM(tab_wins) AS tab_winners, 
    SUM(space_wins) AS space_winners, 
    SUM(tab_wins) / (SUM(space_wins) + 1) AS ratio
  FROM 
    initial_aggregation
  GROUP BY 
    extension
  ORDER BY 
    files_count DESC
)

# FInal formated table
SELECT 
  extension, 
  FORMAT("%'d", files_count) AS files, 
  FORMAT("%'d", lines) AS lines,
  FORMAT("%'d", tab_count) AS tab_count, 
  FORMAT("%'d", space_count) AS space_count,
  FORMAT("%'d", tab_winners) AS tab_winners, 
  FORMAT("%'d", space_winners) AS space_winners,
  ROUND(ratio, 3) AS ratio
FROM  
  tab_and_space_for_each_extensions 
