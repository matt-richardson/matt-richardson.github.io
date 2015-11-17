---
layout: post
title: "Continuous Integration with Octopus Deploy Step Templates"
date: 2015-11-17 07:45:00 +0100
comments: true
published: false
categories: ["blog", "archives"]
tags: ["On Software"]
---

[Octopus Deploy](http://octopus.com) is a great tool in the .net space for deploying applications and managing the lifecycle of the deployments across different environments.

One of the really nice features is the exensibility around step templates (the basic building block of a deployment process) - enabling you to write ([and share](https://library.octopusdeploy.com/)) your own steps in PowerShell, C# or Bash.

Unfortunately, it appears that the common approach of writing, testing and deploying step templates is missing a trick or two from the hard lessons learned around testing and Continuous Integration (CI). Step templates are written locally, tested manually, and then copied and pasted into Octopus Deploy. Editing involves editing in the Octopus Deploy UI or copying and pasting back to a local script. To publish them to the community site, these templates are exported and copy and pasted into a [git repo](https://github.com/octopusdeploy/library). A few too many manual steps in there for my liking.

While working with a major UK ecommerce site on an Octopus Deploy rollout, I've implemented a CI pipeline for step templates that addresses most of these issues. This post walks through the CI process we've implemented.

<!-- more -->

First off, we need to store the step templates into a source code repo. We've gone for a `*.StepTemplate.ps1` filename format (always a fan of being explicit), and we have a `*.StepTemplate.Tests.ps1` that contains our [Pester](https://github.com/Pester/Pester) tests.

Ideally, we would use the same format that Octopus imports and exports, but unfortunately, while it's very friendly for computers, it's not great for humans. So, we've gone for a format that is easy for humans to read and modify, but requires some transformation (compilation?) before it gets imported.

For the sake of this post, we will work with a short and sweet community contributed template, [SQL - Test Connection String](https://library.octopusdeploy.com/#!/step-template/actiontemplate-sql-test-connection-string), which looks like:

```
#Create SQL Connection
$con = new-object "System.data.sqlclient.SQLconnection"
Write-Host "Opening SQL connection to $ConnectionString"

$con.ConnectionString=("$ConnectionString")
try {
    $con.Open()
    Write-Host "Successfully opened connection to the database"
}
catch {
    $error[0]
    exit 1
}
finally{
    Write-Host "Closing SQL connection"
    $con.Close()
    $con.Dispose()
    Write-Host "Connection closed."
}
```

Now, this script, while perfectly functional, and easy to test in isolation, doesn't really confirm to the ideals of a CI pipeline. If we do a small refactor, we can make it easily testable:

```
#sql-test-connection-string.steptemplate.ps1
function Test-ConnectionString {
  param (
    [string]$connectionString
  )
  $con = new-object "System.data.sqlclient.SQLconnection"
  Write-Host "Opening SQL connection to $connectionString"
  
  $con.ConnectionString = $connectionString
  try {
      $con.Open()
      Write-Host "Successfully opened connection to the database"
  }
  catch {
      $error[0]
      return $false
  }
  finally {
      Write-Host "Closing SQL connection"
      $con.Close()
      $con.Dispose()
      Write-Host "Connection closed."
  }
  return $true
}

if ($OctopusParameters) {
  $result = Test-ConnectionString ("$ConnectionString")
  if ($result) { 
    exit 0 
  }
  exit 1
}
```
Moving the code into a function allows us to write some Pester tests for different scenarios:

```
#sql-test-connection-string.steptemplate.tests.ps1

$sut = "sql-test-connection-string.steptemplate.ps1"
. "$PSScriptRoot\$sut"

describe "when connecting to a valid database" {
  context "with a valid connection string" {
    it "should succeed and return true" {
      $result = Test-ConnectionString "server=(local)\sqlexpress;database=master;integrated security=sspi;"
      $result | should be $true
    }
  }

  context "with an invalid connection string" {
    it "should return false" {
      $scriptBlock = { Test-ConnectionString "not a connection string" }
      $scriptBlock | Should Throw
    }
  }
}
## more tests here
```

Now that we've got some tests running, we now have some level of confidence that our code (or more likely, any future modification to it) is all okay, and we can upload to Octopus. But, obviously, we dont want to do that as a manual, easily forgotten step. So we need to add some ability to convert into the json format that Octopus uses for import/export and upload it.

In the json that Octopus expects, there are 4 important parts that change:
* the script body (obviously)
* the name
* the description
* the parameters

If we add these as parameters to the top of the script, we can parse them in our build script:
```
$StepTemplateName = 'SQL - Test Connection'
$StepTemplateDescription = 'Tests a SQL Server connection string by attempting to connect to the database.'
$StepTemplateParameters = 
@(
    @{
        "Name" = "ConnectionString";
        "Label" = "Connection string";
        "HelpText" = "The connection string. For example:\n\n> Data Source=MyServer;Initial Catalog=MyDatabase;User Id=admin; Password=supersecretpassword;";
        "DefaultValue" = $null;
        "DisplaySettings" = @{ }
    }
)
  
# script body goes here
```

From here, we can easily generate the json we need to import into Octopus, and even compare this against the current version so that we dont upload a new version that hasn't changed.

One of the benefits of having all of this run as a single build is that we can run some automated tests against the step template for common issues, and to ensure they conform to our expectations, eg:
* check all the required metadata parameters are supplied
* check that all params are used in the script body
* check that tests exist
* etc

And finally, in our build script, we can check to see if we are running on the build server, and if so, we can upload to Octopus using the api. (We always want to have a build script that is runnable locally, as well as on the server, with minimal differences.)

To summarise the build script:
  * for each step template
   * run template specific pester tests (in -passthru mode, saving the output to an xml file)
   * run generic tests (again, in -passthru mode, saving the output to an xml file)
   * extract metadata from the template
   * create json
   * download the current step template from the server version
   * upload if current version is different to server version
   
With a little modification, this also works for script modules, albeit with less metadata, and a slightly different compare and upload process.

Using [TeamCity](https://jetbrains.com/teamcity) for the CI server, we can easily call our `build.ps1` script, and use the [Xml Reporting build feature](https://confluence.jetbrains.com/display/TCD9/XML+Report+Processing) to import the test results, and even use [build messages](https://confluence.jetbrains.com/display/TCD9/Build+Script+Interaction+with+TeamCity) to group our build output nicely.

At some point, it would be nice to integrate some automated publishing to GitHub, though that would require some kind of mechanism to ensure only publically useful scripts get published (as opposed to scripts that are very custom to this project). It would also need something to ensure that people editing the scripts are aware that their changes will be visible to the world. Another point to consider is how this could tie into the contribution process for the Octopus Deploy community library.

A final thought to ponder is how this could tie into Octopus Deploy itself, making CI easier, but without constraining people who don't want to use it. [Let me know](https://twitter.com/squire_matt) your thoughts! 