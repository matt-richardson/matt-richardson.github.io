---
layout: post
title: "Continuous Integration with Octopus Deploy Step Templates"
date: 2016-04-12 12:34:00 +0100
comments: true
published: true
categories: ["blog", "archives"]
tags: ["On Software"]
---

[Octopus Deploy](http://octopus.com) is a great tool in the .net space for deploying applications and managing the lifecycle of the deployments across different environments.

One of the really nice features is the extensibility around step templates (the basic building block of a deployment process) - enabling you to write ([and share](https://library.octopusdeploy.com/)) your own steps in PowerShell, C# or Bash.

It appears that the common approach of writing, testing and deploying step templates is missing some of the good practices that came from the hard lessons learned around testing and [Continuous Integration](http://www.martinfowler.com/articles/continuousIntegration.html) (CI). Step templates are written locally, tested manually, and then copied and pasted into Octopus Deploy. Updates involves editing in the Octopus Deploy UI or copying and pasting back to a local script. To publish them to the community site, these templates are exported into a git repo and a Pull Request raised against the main [library repo](https://github.com/octopusdeploy/library). All up, quite a few manual steps in there.

While working with [ASOS](https://asos.com) (a major UK fashion site) on an Octopus Deploy rollout, we implemented a CI pipeline for step templates that addresses most of these issues. This post walks through the CI process we've implemented.

<!-- more -->

## Source Control

First off, we need to store the step templates into a source code repo. We've gone for a `*.StepTemplate.ps1` filename format (always a fan of being explicit), and we have a `*.StepTemplate.Tests.ps1` file that contains our [Pester](https://github.com/Pester/Pester) tests.

Ideally, we would use the same format that Octopus imports and exports, but unfortunately, while it's very friendly for computers, it's not great for humans. So, we've gone for a format that is easy for humans to read and modify, but requires some transformation (you could almost call it compilation?) before it gets imported.

## Show me the code

For the sake of this post, we will work with a short and sweet community contributed template, [SQL - Test Connection String](https://library.octopusdeploy.com/#!/step-template/actiontemplate-sql-test-connection-string), which looks like:

```powershell
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

Now, this script, while perfectly functional, and easy to test in isolation (ie, in Powershell ISE), doesn't really conform to the ideals of a CI pipeline. If we do a small refactor, and move the code into a function, we can make it easily testable:

```powershell
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

## Testing 

Moving the code into a function now allows us to write some Pester tests for different scenarios:

```powershell
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

## Script Modules

So far, we have just been talking about Step Templates. Script Modules are another extensibility point in Octopus, that allow us
to write common library functions. The same principles and automation applies to these. We went for a `*.StepTemplate.ps1` and 
`*.StepTemplate.Tests.ps1` approach for these files. 

## Speaking Octopus

Now that we've got some tests running, we now have some level of confidence that our code (or more likely, any future modification to it) is all okay, and we can upload to Octopus. But, following the principle of 'same result every time', we dont want to do that as a manual, easily forgotten step. So we need to add some ability to convert into the json format that Octopus uses for import/export and upload it.

In the json that Octopus expects, there are 4 important parts that change:

 - the script body
 - the name
 - the description
 - the parameters

For script modules, the first three parameters are the same, however they do not have any parameters. 
There are also differences in the way that the Octopus API treats them, but that is outside the scope of this post.

If we add these as parameters to the top of our Step Template script, eg:

```powershell
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

From here, we can extract them out (with a bit of Powershell AST voodoo) and easily generate the json we need to import into Octopus, and even compare this against the current version so that we dont upload a new version that hasn't changed.

## Automated tests

One of the benefits of having all of this run as a single build is that we can run some automated tests against the step template for common issues, and to ensure they conform to our expectations, eg:

 - check all the required metadata parameters are supplied
 - check that all params are used in the script body
 - check that tests exist
 - run static analysis tests using [PSScriptAnalyzer](https://github.com/PowerShell/PSScriptAnalyzer)
 - etc

And finally, in our build script, if we are running on the build server, we can upload to Octopus using the api. (It's always a good idea to make sure the script that the build server runs is also runnable locally, as well as on the server, with as few differences as possible.)

To summarise the process:

 - for each step template / script module
  - run template/module specific pester tests (in -passthru mode, saving the output to an xml file)
  - run generic tests (again, in -passthru mode, saving the output to an xml file)
  - extract metadata from the template / module
  - create json
  - download the current step template from the server version
  - upload if current version is different to server version
   
## Powershell modules

This began life as a plain old `build.ps1` file, that while it worked, it was growing unwieldy. With a stellar contribution from one of the Asos team, [Paul Marston](https://github.com/paulmarsy), this was converted into a proper powershell module that can be used both during local development and for an automated build process. Hopefully, we should be able to publish this onto the [powershell gallery](https://powershellgallery.com) soon.

One of the benefits of doing this as a powershell module was the ease of adding some helper cmdlets for local use. `New-StepTemplate`, `New-ScriptModule` and `New-ScriptValidationTest` cmdlets are available to make it super easy to do local development.

## Automatic test run & publish

Using [TeamCity](https://jetbrains.com/teamcity) for the build server, once we have downloaded and imported the module, we just call `Invoke-TeamCityCiUpload -uploadIfSuccessful`, and use the [Xml Reporting build feature](https://confluence.jetbrains.com/display/TCD9/XML+Report+Processing) to import the test results.

## No really, show me the code

While this has been in use internally at ASOS for quite a while, this has just been published on [github](https://github.com/ASOS/OctopusStepTemplateCi). Go check it out. Issues and PRs gratefully received.

## Where next?

At some point, it would be nice to integrate some automated publishing to GitHub, though that would require some kind of mechanism to ensure only publically useful scripts get published (as opposed to scripts that are very custom to ASOS). It would also need something to ensure that those editing the scripts are aware that their changes will be visible to the world. 

Another interesting challenge to consider is how this could tie into the contribution process for the Octopus Deploy community library.

While this code was written before the new [store step templates in packages](https://github.com/OctopusDeploy/Issues/issues/2189) functionality was introduced in Octopus 3.3.0, it looks very interesting. Definitely on the list to investigate.

A final thought to ponder is how this could tie into Octopus Deploy itself, making CI easier, but without constraining people who don't want to use it. [Let me know](https://twitter.com/squire_matt) your thoughts! 
