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
├── lein-tutorial.iml
├── project.clj
├── resources
├── src
│   └── lein_tutorial
│       └── core.clj
└── test
    └── lein_tutorial
        └── core_test.clj

6 directories, 7 files
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

Or if something is pure Clojure library you'll most likely find it on Clojars, for example let's get the next.jdbc library from https://clojars.org/com.github.seancorfield/next.jdbc:

```clojure 
[com.github.seancorfield/next.jdbc "1.3.1048"]
```

## Most useful commands 

Let's learn most useful command for everyday. 

### Managing dependencies
Once you've added a new dependency to `project.clj` it's magically appear in your project. 
You'll need to use a command to pull it, although if you running any other commands (like test run), dependencies will be updated anyway. 

It's still useful to know how to pull dependencies yourself:
```shell
lein deps
```

You'll probably some output about new dependencies added, it happens if you haven't got them already locally in the Maven `.m2` cache folder. 
Lein will use same folder to cache them as well, so you don't need to download them multiple times. 

Other related command is useful solve the Java `dependency hell` situations, it's a bit more advanced but good to know anyway.
If you get a version conflic (like 2 dependencies are pulling different version of same third dependency) its usesul to debug. 

```shell 
lein deps :tree
 [nrepl "1.0.0" :exclusions [[org.clojure/clojure]]]
 [org.clojure/clojure "1.11.1"]
   [org.clojure/core.specs.alpha "0.2.62"]
   [org.clojure/spec.alpha "0.3.218"]
 [org.nrepl/incomplete "0.1.0" :exclusions [[org.clojure/clojure]]]
```

Luckely for us, in our tiny project the dependency tree looks good so far!

### Building project

This is the bread and butter command, used in most CI/CD pipelines, the idea is to create a JAR file that will contain everything we need to run our application. 
At this point we don't even care that the code was written in Clojure, it's a normal JAR now and we only need Java to run it (so usually that output JAR is used to build a Docker image from some Java base image).

To do so we need to run:
```shell 
lein uberjar
 

```

And after running `tree` command you'll see a fresh JAR created in the target 
folder `target/uberjar/lein-tutorial-0.1.0-SNAPSHOT-standalone.jar`:
```shell 
tree
├── target
│   └── uberjar
│       ├── classes
│       │   ├── META-INF
│       │   │   └── maven
│       │   │       └── lein-tutorial
│       │   │           └── lein-tutorial
│       │   │               └── pom.properties
│       │   └── lein_tutorial
│       │       ├── core$_main.class
│       │       ├── core$fn__173.class
│       │       ├── core$loading__6789__auto____171.class
│       │       ├── core.class
│       │       └── core__init.class
│       ├── lein-tutorial-0.1.0-SNAPSHOT-standalone.jar
│       ├── lein-tutorial-0.1.0-SNAPSHOT.jar
│       └── stale
│           └── leiningen.core.classpath.extract-native-dependencies
```

### Running project 

You can also run the project without building the JAR first, for example for local development:

```shell 
lein run
Hello, World!
```

It will basically execute the `-main` function from the entrypoint namespace defined in the `project.clj` config:
```clojure
(defn -main
  "I don't do a whole lot ... yet."
  [& args]
  (println "Hello, World!"))
```

JVM is known for the slow start but actually in this case there are 2 nested JVMs started,
one for our app and one for the Lein itself, but we can improve that by running the trampoline command:

```shell 
trampoline          Run a task without nesting the project's JVM inside Leiningen's.
```

This is the difference I'm getting on my machine:

```shell 
time lein run
Hello, World!
________________________________________________________
Executed in    1.69 secs    fish           external
   usr time    1.21 secs    0.00 micros    1.21 secs
   sys time    0.44 secs  406.00 micros    0.44 secs

time lein trampoline run
Hello, World!
________________________________________________________
Executed in    1.02 secs    fish           external
   usr time    1.18 secs    0.00 micros    1.18 secs
   sys time    0.33 secs  398.00 micros    0.33 secs
```

### Running tests

Lein includes the `clojure.test` runner by default (or you can also confire it to use some other available test runner), 
the default is good enough for the start. 

```shell 
lein test

lein test lein-tutorial.core-test

lein test :only lein-tutorial.core-test/a-test

FAIL in (a-test) (core_test.clj:7)
FIXME, I fail.
expected: (= 0 1)
  actual: (not (= 0 1))

Ran 1 tests containing 1 assertions.
1 failures, 0 errors.
Subprocess failed (exit code: 1)
```

We have one failing test in the project, you can figure out how to fix it, rerun the test and see a passing output. 


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


