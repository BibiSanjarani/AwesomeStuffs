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

## Methodology

### What do we need to do so we can get the top 100?

- See how many times a package is listed as a dependency for projects.
- Filter by NPM packages.
- Aggregate data by the package name.
- Order the data in decending order and limit how many results we want to see (100).

### What's the easiest way to do this?

#### Pick the right source. 
Libraries.io has tables that fit the many to one relationship for dependencies. Getting this from github's database would require more work to represent this before we have actionable data to use. Since we are looking for the top 100 most popular npm packages, where popularity is determined by the *how **many** times a package is listed as **a dependency** * 

I have chosen to use the table  `bigquery-public-data.libraries_io.repository_dependencies` to do this.

Fields we want:


- The name of a dependency for a project => `dependency_project_name `

- The count of how many records the dependency of a project turns up. This isn't a field in the table so we have to run SQL function (COUNT) on the `dependency_project_name ` . We can set the alias for the count as anything => `COUNT(dependency_project_name) AS amount`

- Name of the package manifest platform, so we can filter by `npm` => `manifest_platform`

### Full query

    ```#Get package name and count how may times it turns up in DB
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
    
    #Only get the top 100
      LIMIT
        100```
        
   

## Results
  
  Top 5. See the rest => [Click me](./results-20190221-214858.json)
 
       
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
       
# Top 100 NPM dependencies with patched or unpatched vunerabilities

    #Get package name and count how may times it turns up in DB
      SELECT
     dependency_project_name,
     COUNT(dependency_project_name) AS amount
    
    #From sample commits   
      FROM
        `bigquery-public-data.github_repos.sample_commits` commits
    
    #Make repo names equal the same from both Libraries.io and Github DB's 
     INNER JOIN
        `bigquery-public-data.libraries_io.repository_dependencies` repo_dependencies
      ON
        commits.repo_name = repo_dependencies.repository_name_with_owner
    #Conditions: 1) only NPM packages, 2)Commit message contains one or more of key words 
      WHERE
        (repo_dependencies.manifest_platform LIKE "npm")
        AND ((commits.message LIKE "%vunerability%")
          OR (commits.message LIKE "%escape%")
          OR (commits.message LIKE "%remote%"))
    
    #Aggregate results by popular npm package name  
      GROUP BY
        dependency_project_name
    #Order by count - Decending 
      ORDER BY
        amount DESC
    #Only get top 100  
      LIMIT
        100
## Results

Top 5. See the rest => [Click me](./results-20190222-010623.json)

        {
          "dependency_project_name": "vscode-nls",
          "amount": "1258"
        },
        {
          "dependency_project_name": "@types/node",
          "amount": "1036"
        },
        {
          "dependency_project_name": "object-assign",
          "amount": "517"
        },
        {
          "dependency_project_name": "semver",
          "amount": "475"
        },
        {
          "dependency_project_name": "rimraf",
          "amount": "467"
        }
