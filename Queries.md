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

# Top 100 NPM packages
    #Get package name and count how may times it turns up in DB
      SELECT
     dependency_project_name,
     COUNT(dependency_project_name) AS amount
      FROM
        `bigquery-public-data.libraries_io.repository_dependencies`
      
    #Condition: Look for only NPM packages by filtering by manifest platform type
      WHERE
        manifest_platform LIKE "npm"
    
    #Aggregate results by package name
     GROUP BY
        dependency_project_name
    
    #Order by count - Decending
      ORDER BY
        amount DESC  
    
    #Only want the top 100
      LIMIT
        100
        
   
  ##Results
  
  Top 5. See the rest => link
 
       
        {
          "dependency_project_name": "mocha",
          "amount": "887231"
        },
        {
          "dependency_project_name": "express",
          "amount": "495152"
        },
        {
          "dependency_project_name": "debug",
          "amount": "411817"
        },
        {
          "dependency_project_name": "async",
          "amount": "400717"
        },
        {
          "dependency_project_name": "babel-core",
          "amount": "383659"
        }
       
