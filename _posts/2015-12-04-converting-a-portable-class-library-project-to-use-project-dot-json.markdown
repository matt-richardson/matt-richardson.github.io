---
layout: post
title: "Converting a Portable Class Library project to use project.json"
date: 2015-12-04 09:01:00 +0100
comments: true
published: true
categories: ["blog", "archives"]
tags: ["On Software"]
---

I've been helping setting up a whole bunch of builds in [TeamCity](http://jetbrains.com/teamcity) for a client, moving them over from TFS. As the build log is, err, not that easy to see in TFS, the developers have not been paying too much attention to the warnings that the build is throwing.

To help drive best practice, we've been pushing towards fixing these warnings, and then enabling a TeamCity failure condition to prevent them occuring again. One of these was the dreaded `MSB3277: Found conflicts between different versions of the same dependent assembly that could not be resolved.  These reference conflicts are listed in the build log when log verbosity is set to detailed.`

After investigating, I found that MSBuild was getting confused as one of the dependencies ([stateless](https://github.com/nblumhardt/stateless)) was a PCL library, and therefore referencing `System.Core, Version=2.0.5.0`, whereas the rest of the code was referencing `System.Core, Version=4.0.0.0`. Now, version 2.0.5.0 of the reference assemblies is an interesting one, as its not a real assembly - its just there for the compiler to use, and uses type forwarding to pass it onto the correct assembly.

This all compiles and runs successfully, but it still shows that pesky error during compilation. So, being a good open source citizen, I logged [an issue](https://github.com/nblumhardt/stateless/issues/37), and hoped for the best. But as this is the real world, the maintaner was a bit too busy to take a look and suggested I send a pull request, and added the suggestion that it would be worth using the new `project.json` format that ASP.NET 5 / Visual Studio 2015 introduced. And as such things are want to be, it took a tad more effort than expected.

<!-- more -->

The easy part was deleting the `.nuspec` file, the `.csproj` files, and creating `project.json` files, using just a basic one for most projects:

```
{
  "dependencies": {
    "Stateless": ""
  },
  "frameworks": {
    "net40": { }
  }
}
```

Some needed to be strong named (which meant I needed to upgrade from beta5 to RC1), but that was simple:

```
{
  "compilationOptions": {
    "keyFile": "Stateless.Tests.snk"
  },
  "dependencies": {
    "NUnit": "2.6.4",
    "Stateless": ""
  },
  "frameworks": {
    "net40": { }
  }
} 
``` 

Even the main dll was relatively simple:

```
{
  "title": "Stateless",
  "description": "Create state machines and lightweight state machine-based workflows directly in .NET code",
  "authors": [ "Nicholas Blumhardt and Contributors" ],
  "owners": [ "" ],
  "language": "en-US",
  "licenseUrl": "http://www.apache.org/licenses/LICENSE-2.0",
  "projectUrl": "https://github.com/nblumhardt/stateless",
  "iconUrl": "https://code.google.com/p/stateless/logo",
  "compilationOptions": {
    "keyFile": "Stateless.snk"
  },
  "dependencies": {
  },
  "frameworks": {
    ".NETPortable,Version=v4.0,Profile=Profile136": {
      "compilationOptions": { "define": [ "PORTABLE" ] },
      "frameworkAssemblies": {
        "mscorlib": "",
        "System": "",
        "System.Core": ""
      }
    },
    "net35": {
      "compilationOptions": { "define": [ "NET35" ] },
      "frameworkAssemblies": {
        "mscorlib": "",
        "System": "",
        "System.Core": ""
      }
    },
    "net40": {
      "compilationOptions": { "define": [ "NET40" ] },
      "frameworkAssemblies": {
        "mscorlib": "",
        "System": "",
        "System.Core": ""
      }
    }
  }
}
```

(Note that a lot of the information that used to be in a nuspec file is now in project.json)

This all built and created the `.nupkg` file as expected. The real challenge was around successfully creating the Portable Class Library (PCL) version, as the old build did. Specifying the appropriate `.NETPortable,Version=v4.0,Profile=Profile136` worked at compile time, and created the expected structue in the `nupkg` file, but opening it up in [DotPeek](http://jetbrains.com/dotpeek) showed that it was compiling as .net 4.6. What the?

![.net 4.0 PCL ends up targeting .net v4.6?](/assets/img/portable-class-libraries-in-aspnet5-incorrect-framework.png)

After banging my head against the wall many times, I eventually stumbled across [a reference](http://stackoverflow.com/a/16254204/779192) that mentioned you now need to use the `TargetFramework` assembly attribute to specify this, eg:

```
#if PORTABLE
[assembly: TargetFramework(".NETPortable,Version=v4.0,Profile=Profile136", FrameworkDisplayName = ".NET Portable Subset")]
#endif
```
  
and now it magically compiles against the correct framework:

![targeting correct framework](/assets/img/portable-class-libraries-in-aspnet5-correct-framework.png)

For those following along at home, feel free to check out the resulting [pull request](https://github.com/nblumhardt/stateless/pull/41). Got a [build failure](https://ci.appveyor.com/project/NicholasBlumhardt/stateless/build/2.5.33) at the moment, but hopefully that's just AppVeyor configuration.

Overall, I'm really liking the multi-targeting in aspnet5 - just needs the tooling to catch up a bit.

Hope this tidbit helps someone!
