# Getting Started with Leiningen: A Beginner’s Guide

If you’re learning Clojure, you’ll soon need a tool to manage your projects.
At the beginning, I recommend sticking with Leiningen—it simplifies setup and lets you focus on learning the language itself.

Later on, it’s worth exploring the [Clojure CLI](https://clojure.org/guides/deps_and_cli)
and deps.edn. The CLI has been gaining popularity, partly because it’s supported directly by the Clojure core team,
so you’ll likely come across it in real-world projects. It also offers some neat features,
such as dynamic loading of libraries without restarting the REPL and support for git-based dependencies.

But as a beginner, starting with Leiningen will give you the smoothest experience.

## A bit of history

When Clojure was first released, there was no tooling support for building projects or managing dependencies—it was basically *do it yourself*.

I checked the GitHub history of the Leiningen project and found its very first commit from [November 1st, 2009](https://github.com/technomancy/leiningen/commit/88b53602f744584cb434346d868629100448ff59).

Leiningen quickly gained traction. By the time I joined the Clojure community (around 12 years ago), it had already become the de facto standard.

There was also a competing project called [Boot](https://github.com/boot-clj/boot) at that time. Honestly, I never quite understood the need for it, and today we can safely say the project is no longer active.

So, in a nutshell, what do we actually need from a build tool?

- Dependency management – add a new Java or Clojure library, and the tool downloads it and makes it available on the classpath.
- Building the project – produce a JAR file so we can package it into a Docker image and deploy our service.
- Running tests – execute unit and integration tests.
- Running the project – less common in practice. For local development, you’d typically start a REPL and run the app from there. For production, you’d normally deploy a Docker image containing Java and your JAR, though I’ve seen projects deployed with Leiningen inside the Docker image, starting the app without precompiling.
- Running a REPL – start an interactive session for development, in most cases it will be managed by your editor or IDE, so you don't need to directly use Leiningen for it.

Leiningen covers all of these out of the box, with no extra configuration required. So let’s dive in and see it in action!

## Installation


Leiningen is available through multiple package managers, which is usually the preferred way to install it: [https://wiki.leiningen.org/Packaging](https://wiki.leiningen.org/Packaging)

If that doesn’t work for you, Leiningen can also be installed via a script.

Personally, I use [`mise`](https://github.com/jdx/mise) to manage my dependencies. With it, I can install Leiningen via a plugin.  
You’ll also need Java installed on your machine:

```bash 
mise use -g java@21

java --version
openjdk 21.0.1 2023-10-17
OpenJDK Runtime Environment (build 21.0.1+12-29)
OpenJDK 64-Bit Server VM (build 21.0.1+12-29, mixed mode, sharing)
``` 
Now install Leiningen with `mise`:
```bash 
mise plugins install lein https://github.com/miorimmax/asdf-lein.git
mise use -g lein@latest

lein version
Leiningen 2.11.2 on Java 21.0.1 OpenJDK 64-Bit Server VM
```

That’s it — Leiningen is installed and ready to go!

## Creating a new project

Once you have Leiningen installed, starting a new Clojure project is as easy as running a single command:

```shell 
lein new app lein-tutorial
```

Here’s what each part means:

- `lein new` – the command to generate a new project.
- `app` – the template to use. This one creates a project for a standalone application or service, and it’s the most commonly used template.
- `lein-tutorial` – the name of the project.

Leiningen also supports third-party templates, and you can even create your own if you want more customization.
Once the command finishes, you’ll have a new project directory named `lein-tutorial`. Let’s move inside and explore the generated structure:

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

The `project.clj` file is the main configuration file for your project. It defines things like the project name, version, dependencies, build instructions, and more.  
Let’s take a look at what Leiningen generates by default:
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
Here we see some placeholders we can change, like `:description`, `:url`, and `:license`.  
But the most important section to pay attention to is the `:dependencies` vector.

Right now, it only contains the Clojure dependency itself. Let’s learn how to add additional libraries to our project.

Unlike some build tools, Leiningen doesn’t have a CLI command to add dependencies for you—and honestly, that’s a good thing. It keeps the process simple: just copy the dependency string and paste it into your `project.clj`.

You’ll usually find the correct string on either **Maven Central (MVNRepository)** or **Clojars**. The structure is always the same:

```clojure
[group-id/package-name "version"]
```

### Example 1: From Maven Central
If we want to add **HikariCP** (a popular JDBC connection pool), we can grab the dependency from [MVNRepository](https://mvnrepository.com/artifact/com.zaxxer/HikariCP/7.0.2):

```clojure
[com.zaxxer/HikariCP "7.0.2"]
```

### Example 2: From Clojars
For pure Clojure libraries, you’ll usually use [Clojars](https://clojars.org/).  
For instance, to add `next.jdbc` library:

```clojure
[com.github.seancorfield/next.jdbc "1.3.1048"]
```

Once added, Leiningen will automatically download these dependencies the next time you run your project or start a REPL.

## Most useful commands

Let’s go over some of the commands you’ll use most often in everyday Clojure development.

### Managing dependencies
Once you’ve added a new dependency to `project.clj`, it will automatically become part of your project.  
However, you may need to explicitly pull it down. Note that if you run other commands (like tests or running the REPL), Leiningen will automatically fetch any missing dependencies.

Still, it’s useful to know how to pull dependencies yourself:
```shell
lein deps
```

You’ll probably see some output about new dependencies being downloaded if they aren’t already in your local Maven .m2 cache folder.
Leiningen uses the same folder for caching, so you don’t have to download the same dependencies multiple times.

Another related command is useful for solving Java “dependency hell” situations. This is a bit more advanced, but good to know - if two dependencies require different versions of the same library, you can debug conflicts with:

```shell 
lein deps :tree
 [nrepl "1.0.0" :exclusions [[org.clojure/clojure]]]
 [org.clojure/clojure "1.11.1"]
   [org.clojure/core.specs.alpha "0.2.62"]
   [org.clojure/spec.alpha "0.3.218"]
 [org.nrepl/incomplete "0.1.0" :exclusions [[org.clojure/clojure]]]
```

Luckily for us, in our small project the dependency tree looks clean so far!

### Building the project

This is the bread-and-butter command, often used in CI/CD pipelines.  
The idea is to create a JAR file that contains everything needed to run your application.

At this point, it doesn’t matter that the code was written in Clojure—the output is just a normal Java JAR.  
All you need to run it is Java, which is why this JAR is often used as the basis for a Docker image built on a Java base image.

To build the project, run:
```shell 
lein uberjar
```

After running `tree` command you'll see a fresh JAR created in the target 
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
  []
  (println "Hello, World!"))
```

The JVM is known for its slow startup, and in this case, two nested JVMs are started—one for Leiningen itself and one for your application.
We can improve startup time by using the `trampoline` command:

```shell 
trampoline Run a task without nesting the project's JVM inside Leiningen's.
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

Leiningen includes the `clojure.test` runner by default, though you can configure it to use other test runners if needed.  
For starting out, the default setup is perfectly fine.

To run your tests, simply execute:

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

If you have a failing test in the project, try fixing it, then rerun the command to see the passing output.

## Plugins 
Leiningen has a rich ecosystem of plugins for different tasks.  
For a long time, plugins were the go-to way to add linters, formatters, and other development tools to your project.

Nowadays, many tools are available as standalone binaries, thanks to projects like Babashka and GraalVM.  
Still, it’s useful to know how to use plugins in Leiningen.

For example, let’s add the popular code formatter [`cljfmt`](https://github.com/weavejester/cljfmt).  
Add the following line to your `project.clj` inside the `:plugins` vector (or extend the existing vector if you already have plugins):

```clojure 
:plugins [[dev.weavejester/lein-cljfmt "0.13.1"]]
```

Now we have access to additional commands, for example to check the formatting:
```bash 
lein cljfmt check
All source files formatted correctly
```

## Conclusion

Leiningen still has a strong place in the Clojure ecosystem, and I personally enjoy using it a lot.  
For beginners, I definitely recommend starting with Leiningen, and then exploring `tool.deps` and the Clojure CLI later.

Don’t worry about using something “deprecated” — Leiningen does its job very well and provides all the features you need to build a Clojure application.

For reference, here’s a sample `project.clj` file with many properties and settings: [project.clj](https://codeberg.org/leiningen/leiningen/src/branch/stable/sample.project.clj)

There’s also an official tutorial that’s worth reading: [Leiningen Tutorial](https://leiningen.org/tutorial.html)
