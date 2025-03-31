# The repository
This repo holds PowerQuery "templates", i.e. power query files that might need a minor change for it to work for a specific use case. Typical example: API templates that need a web path specified to avoid dynamic data source issues.The files have extension is *.pt* (PQ Templates).

Note that the snippets need some minor tweaking for your specific environment. For example: API calls using the Web.Contents function must have the domain hard coded, otherwise auto-refresh will not work.  So you need to hardcode their own domain in before the function can be used. 

Since patterns require some modification, these should not be linked directly from github.

# Repository structure
As most of the functions are API related, the repo is split by application type where possible, like "Jira","Sharepoint",etc.  

# Using Templates
I tried to keep the format as cosistent as possible for templates and changes to be made are identifiable via text in angle brackets or chevrons. 

```
(SourceTable as table, Domains as text, Projects as text)=> 
let
  #"Get Keys" = Table.ExpandRecordColumn(
		  Table.ExpandListColumn(
		    Table.ExpandRecordColumn(
		      Table.AddColumn(SourceTable, "keys", each 
            if Record.Field(_,Domains) = "<your domain>" 
            then Json.Document(
              Web.Contents(
                "<your domain>",
                  [
                    RelativePath = "rest/api/3/search/",
                    Query = [
                      jql="project="&Record.Field(_,Projects),
                      maxResults = "10000" 
                    ]
                  ]
              )
            )
            else null
          ), 
		      "keys", {"issues"}
		    ), "issues"
		  ), 
		  "issues", {"key"}
	  )
in #"Get Keys"
```

In this case, **"&lt;your domain&gt;"** needs to be replaced with the string (e.g. with **"https://xyz.atlassian.net"**) of your JIRA domain omit the forward slash at the end of your domain, otherwise the function will return nothing. 

While a reasonable question would be to ask why **Record.Field(_,Domains)** is not added to the Web.Contents area as this works in PowerBI desktop, there is a reason for it being implemented as it is. 

In the function above, if the domain does not match the string, the field will be populated with null. This can be changed to suit of course.

Essentially, a published PowerBi report cannot easily verify a dynamic domain, meaning you end up with an "dynamic datasource" error. To mitigate this, you can use a workaround via an if statement to check if **Record.Field(_,Domains)** matches a certain string and if it does, use the string as the domain. 

_**Note:** All other parts of the Web.Contents function (RelativePath and Query's) do allow parameters and dynamic content to be used, only the main domain does not._

To add more domains (in this case), extend the if statement as follows:

```
(SourceTable as table, Domains as text, Projects as text)=> 
let
  #"Get Keys" = Table.ExpandRecordColumn(
		  Table.ExpandListColumn(
		    Table.ExpandRecordColumn(
		      Table.AddColumn(SourceTable, "keys", each 
            if Record.Field(_,Domains) = "<your domain>" 
            then Json.Document(
              Web.Contents(
                "<your domain>",
                  [
                    RelativePath = "rest/api/3/search/",
                    Query = [
                      jql="project="&Record.Field(_,Projects),
                      maxResults = "10000" 
                    ]
                  ]
              )
            ) else


            if Record.Field(_,Domains) = "<your second domain>" 
            then Json.Document(
              Web.Contents(
                "<your second domain>",
                  [
                    RelativePath = "rest/api/3/search/",
                    Query = [
                      jql="project="&Record.Field(_,Projects),
                      maxResults = "10000" 
                    ]
                  ]
              )
            )

            ...

            else null
          ), 
		      "keys", {"issues"}
		    ), "issues"
		  ), 
		  "issues", {"key"}
	  )
in #"Get Keys"
```
