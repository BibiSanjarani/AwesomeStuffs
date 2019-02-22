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
### So how do we use this to find out what are the most popularly used npm packages with unpatched/patched vunerabilities?

We want to use the basic framework of what we've done earlier but just add in a few more metrics
- See how many times a package is listed as a dependency for projects.
- **Filter by keywords that indicate a vunerability in a commit message.**
- Filter by NPM packages.
- Aggregate data by the package name.
- Order the data in decending order and limit how many results we want to see (100).

### What's the easiest way to do this?

#### Pick the right source. 

Earlier, I said this...

"Libraries.io has tables that fit the many to one relationship for dependencies. Getting this from github's database would require more work to represent this before we have actionable data to use. Since we are looking for the top 100 most popular npm packages, where popularity is determined by the 'how **many** times a package is listed as **a dependency**.' "

... But we can no longer just use `libraries.io` as a sole source as it doesn't contain commit messages or it would be exaustive to get them. So we need to find a common field in both DBs to do this. The most obvious field to do a join on, would be the name of the repository.

Fields we want from Libraries.io:

- The name of a dependency for a project => `dependency_project_name `

- The count of how many records the dependency of a project turns up. This isn't a field in the table so we have to run SQL function (COUNT) on the `dependency_project_name ` . We can set the alias for the count as anything  => `COUNT(dependency_project_name) AS amount`

- Name of the package manifest platform, so we can filter by `npm` => `manifest_platform`
- The name of the repository => `repository_name_with_owner`

Fields we want from Github Sample tables `bigquery-public-data.github_repos.sample_commits` :

- The name of a repository to join on => `repo_name`
- The commit messages =>  `message`

What keywords do we want to search for?

Essentially we want vocabulary that indicates a vunerability, ie. the word 'vunerability', without attepting to look for a certain type of vunerability and without including ordinary bug fixes. So the word "fix" or "patch" are too generic and will yeild a lot of results.
The additional words I have chosen to use are "escape" and "remote" as these indicate potential places that might be exploitable. I.e escaping characters or remote feeds for distributed sites or CDN.

    ```#Get package name and count how may times it turns up in DB
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
        100```

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
