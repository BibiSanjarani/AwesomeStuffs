   # Using BigQuery to find zipslip
   
   ### bigquery-public-data.libraries_io

#### SQL query
    #select fields from libraries.io.repositories_dependencies DB
    SELECT `bigquery-public-data.libraries_io.repository_dependencies`.repository_name_with_owner, `bigquery-public-data.libraries_io.repository_dependencies`.repository_id, manifest_filepath, dependency_project_name, dependency_requirements  
    FROM `bigquery-public-data.libraries_io.repository_dependencies`

    # make repository dependency name = name in projects dependency field so you can get the time last pushed displayed in table 
    INNER JOIN `bigquery-public-data.libraries_io.projects_with_repository_fields`
    ON `bigquery-public-data.libraries_io.repository_dependencies`.dependency_project_name = `bigquery-public-data.libraries_io.projects_with_repository_fields`.name
    
    #conditional clause for finding pom file in manifest path field and host so I can get all the git hub ones only and the zipslip #depedecy name
    WHERE (manifest_filepath LIKE "pom.xml" AND host_type LIKE "GitHub" 
    AND dependency_project_name LIKE "%zt-zip%") 

    #order ascending when the repo was last pushed*
    ORDER BY `bigquery-public-data.libraries_io.projects_with_repository_fields`.repository_last_pushed_timestamp ASC
    
    # set load so you dont download too much *
    LIMIT 1000
