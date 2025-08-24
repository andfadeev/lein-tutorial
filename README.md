# Getting Started with Leiningen: A Beginner’s Guide

If you are learning Clojure you will pretty soon need a tool to manage your projects. 
I would suggest to stick to Leiningen for a while to simplify your learning. In the future it's still useful to explore Clojure CLI (LINK HERE)
and `deps.edn`. It's getting more popular due to support from language developers and it's high chance that you will encounter it in a real project
But as a beginner definitely stick with Lein for a while. 

## History

Once clojure was released it didn't have any tooling support for buildking projects and managing dependencies, so I was basically do it yourself. 

I've checked GitHub history of Leinengen project and found the first commit from 1st November 2009 (https://github.com/technomancy/leiningen/commit/88b53602f744584cb434346d868629100448ff59)

And it quickly gained traction, so when I joined Clojure community (around 12 years ago) lein was already a de-facto standard. 

There was also a competitor project Boot at that time, but frankly speaking I never understood why we needed that and today we can say that that project is dead. 

So it a nutshell what do we need from the tool:
- dependencies management (you add somewhere a new java or clojure dependency and the tool downloads it and makes it available on the classpath)
- building project (we want some kind of a Jar as the output, so we can package it as a DOcker image and deploy our service)
- running tests 
- running project (it's usually less common, for local development you'd probably start REPL and run you app from it, for production, probably the DOcker iamge with Java and Jar inside, although I've seen cases when projects where deploy with DOcker image containing Lein and starting the app without precompiling Java)
- running REPL

Leiningen ticks all the boxes without any custom configuration, so let's get started and see it in action. 

## Installation

Lein is available in multiple package managers, so that probably should be a prefered way to install it: https://wiki.leiningen.org/Packaging
If not it could be installed with a script. 

I'm using `mise` to manage my dependencies, so I'm installing lein through plugin, also you need to install Java on your machine:

```bash 
mise use -g java@21

java --version
openjdk 21.0.1 2023-10-17
OpenJDK Runtime Environment (build 21.0.1+12-29)
OpenJDK 64-Bit Server VM (build 21.0.1+12-29, mixed mode, sharing)
``` 

```bash 
mise plugins install lein https://github.com/miorimmax/asdf-lein.git
mise use -g lein@latest

lein version
Leiningen 2.11.2 on Java 21.0.1 OpenJDK 64-Bit Server VM
```

Now we are ready to get started!

## Creating new project 

Now as you have Lein install, to start a new Clojure project is a easy as running a simple command:
```shell 
lein new app lein-tutorial
```

So 'lein new' is a command to generate the project, `app` is the template name to use, it will create a project for a new service and it used most of the time.
Note that there are third-party templates available, or you can create your own template if you want. 

```shell 
lein new 

Subtasks available:
template   A meta-template for 'lein new' templates.
plugin     A leiningen plugin project template.
default    A general project template for libraries.
app        An application project template.
```

Finally we used `lein-tutorial` as the project name, so once the command is finished we can move to a newly created folder and explore the structure:
```shell 
cd lein-tutorial/

tree

├── CHANGELOG.md
├── LICENSE
├── README.md
├── doc
│   └── intro.md
├── project.clj
├── resources
├── src
│   └── lein_tutorial
│       └── core.clj
└── test
    └── lein_tutorial
        └── core_test.clj

7 directories, 7 files
```

## Exploring the `project.clj` file

The `project.clj` file is the main entry point to manage your application, let's see what we have by default:
```clojure
(defproject lein-tutorial "0.1.0-SNAPSHOT"
  :description "FIXME: write description"
  :url "http://example.com/FIXME" 
  :license {:name "EPL-2.0 OR GPL-2.0-or-later WITH Classpath-exception-2.0"
            :url "https://www.eclipse.org/legal/epl-2.0/"}
  :dependencies [[org.clojure/clojure "1.11.1"]]
  :main ^:skip-aot lein-tutorial.core
  :target-path "target/%s"
  :profiles {:uberjar {:aot :all
                       :jvm-opts ["-Dclojure.compiler.direct-linking=true"]}})
```

Here we see some placeholders that we can change like `description`, `url` and `licence`, but the main section we should pay attention to is the `:dependencies` array.
Now we only have Clojure dependency itself, so let's learn how to add additional libraries to our project. 

There is no CLI command to do it for you, but honestly I prefer it that way, you just copy dependency string and paste it into `project.clj`. 
Most sources like MVN repository or Clojars will have the proper Lein dependency string available, but the structure is simple:
```clojure 
[org.name/package-name "0.1.0-version"]
```

For example, HikariCP from MVNRepository: https://mvnrepository.com/artifact/com.zaxxer/HikariCP/7.0.2 will be:
``` 
[com.zaxxer/HikariCP "7.0.2"]
```

Or if something is pure Clojure library you'll most likely find it on Clojars, 




lein run vs lein trampoline run


## Plugins 
There are tons of plugins available for different tasks, and for a long time it was a go-to way to add linters and formatters to the project. 
Nowadays, most of the tools are available as native binaries, due to the popularity of the Babashka project and GraalVM.
So using plugins in Lein now is less common and you should still probably know about that option. Let's say we 
want to add popular formatter `cljfmt`: https://github.com/weavejester/cljfmt

We need to add this line into our `project.clj` or if you already have some plugins to extend that array with a new entry:
```clojure 
:plugins [[dev.weavejester/lein-cljfmt "0.13.1"]]
```

Now we have access to additional commands, for example to check the formatting:
```bash 
lein cljfmt check
All source files formatted correctly
```

## Conclusion

I think Lein still has its own place in the Clojure ecosystem and I personally enjoy using it a lot. 
For beginners, I definitely suggest start with Lein and then explore tool.deps and Clojure CLI later. 
Don't feel like you are using something deprecated, it's doing its job and doing it well - and definitely 
provides all the features for building your Clojure application.

At the end I'd like to share a sample `project.clj` file with lot's properties, I usually use it as a reference: https://codeberg.org/leiningen/leiningen/src/branch/stable/sample.project.clj

There is also an official tutorial that worth reading: https://leiningen.org/tutorial.html


