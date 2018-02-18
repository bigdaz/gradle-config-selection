# gradle-config-selection
Provides a demonstration of configuration selection in Gradle.

Note that the `dependencyInsight` demos below use a hacked version of Gradle that lists _all_ selected configurations for 'variant name'.

Has a root project with 3 subprojects:
- `:sub1-vanilla`: defines 4 configurations without attributes
- `:sub2-variants`: defines 4 configurations with custom attributes
- `:sub3-java`: applies the `java` plugin, getting the relevant configurations and attributes

The root project defines the following dependency graph:
```
+--- project :subproject1-vanilla
+--- project :subproject2-variants
+--- project :subproject3-java
+--- org:pom1:1.0
|    +--- org:pom-from-pom:1.0
|    \--- org:ivy-from-pom:1.0
\--- org:ivy1:1.0
     +--- org:pom-from-ivy:1.0
     |    \--- org:pom-from-pom-from-ivy:1.0
     \--- org:ivy-from-ivy:1.0
```

#### Resolving without attributes
When resolving this graph using a configuration with no attributes. Things to note:

- `:subproject2-variants`: does not show up. Fails due to lack of disambiguating attributes in request. 
- `:subproject3-java`: 'runtimeElements' is chosen due to the `PreferJavaRuntimeVariant` disambiguation rule.
- `pom1` and `ivy1`: 'default' is used for direct project->external dependency
- `ivy-from-pom`: Standard mapping for a dependency declared in a POM file is `compile->compile,master;%->compile,runtime,master`
- `pom-from-pom`: Uses standard mapping as per `ivy-from-pom`, but leaves out 'compile' (included in 'runtime') and 'master' (empty)
- `ivy-from-ivy` and `pom-from-ivy`: 'master,compile' is defined in the `ivy.xml` file.
- `pom-from-pom-from-ivy`: Uses standard mapping for POM files, but shows the `compile->compile,master` portion. This is only possible because the Ivy file points to the compile configuration in `pom-from-ivy`


```
➜  config-selection (master) gradle dependencyInsight --configuration conf --dependency o

> Task :dependencyInsight 
project :subproject1-vanilla
   variant "default"
\--- conf

project :subproject3-java
   variant "runtimeElements" [
      org.gradle.usage = java-runtime-jars (not requested)
   ]
\--- conf

org:ivy-from-ivy:1.0
   variant "master+compile"
\--- org:ivy1:1.0
     \--- conf

org:ivy-from-pom:1.0
   variant "runtime+compile+master"
\--- org:pom1:1.0
     \--- conf

org:ivy1:1.0
   variant "default"
\--- conf

org:pom-from-ivy:1.0
   variant "master+compile"
\--- org:ivy1:1.0
     \--- conf

org:pom-from-pom:1.0
   variant "runtime"
\--- org:pom1:1.0
     \--- conf

org:pom-from-pom-from-ivy:1.0
   variant "compile"
\--- org:pom-from-ivy:1.0
     \--- org:ivy1:1.0
          \--- conf

org:pom1:1.0
   variant "default"
\--- conf
```

#### Resolving with custom attribute
When resolving this graph using a configuration a single 'quality' attribute. Things that are different from previous:

- `:subproject2-variants`: is now resolved due to disambiguating 'quality' attribute.

```
➜  config-selection (master) gradle dependencyInsight --configuration confWithAttribute --dependency o 

> Task :dependencyInsight 
project :subproject1-vanilla
   variant "default" [
      Requested attributes not found in the selected variant:
         quality = a
   ]
\--- confWithAttribute

project :subproject2-variants
   variant "default" [
      quality = a
   ]
\--- confWithAttribute

project :subproject3-java
   variant "runtimeElements" [
      org.gradle.usage = java-runtime-jars (not requested)
      Requested attributes not found in the selected variant:
         quality          = a
   ]
\--- confWithAttribute

org:ivy-from-ivy:1.0
   variant "master+compile" [
      Requested attributes not found in the selected variant:
         quality = a
   ]
\--- org:ivy1:1.0
     \--- confWithAttribute

org:ivy-from-pom:1.0
   variant "runtime+compile+master" [
      Requested attributes not found in the selected variant:
         quality = a
   ]
\--- org:pom1:1.0
     \--- confWithAttribute

org:ivy1:1.0
   variant "default" [
      Requested attributes not found in the selected variant:
         quality = a
   ]
\--- confWithAttribute

org:pom-from-ivy:1.0
   variant "master+compile" [
      Requested attributes not found in the selected variant:
         quality = a
   ]
\--- org:ivy1:1.0
     \--- confWithAttribute

org:pom-from-pom:1.0
   variant "runtime" [
      Requested attributes not found in the selected variant:
         quality = a
   ]
\--- org:pom1:1.0
     \--- confWithAttribute

org:pom-from-pom-from-ivy:1.0
   variant "compile" [
      Requested attributes not found in the selected variant:
         quality = a
   ]
\--- org:pom-from-ivy:1.0
     \--- org:ivy1:1.0
          \--- confWithAttribute

org:pom1:1.0
   variant "default" [
      Requested attributes not found in the selected variant:
         quality = a
   ]
\--- confWithAttribute
```

#### Resolving from a standard Java configuration
When resolving this graph using the `compileClasspath` configuration. Things that are different from previous:

- `:subproject3-java`: get `apiElements` instead of `runtimeElements`. The attribute select works!

```
➜  config-selection (master) gradle dependencyInsight --configuration compileClasspath --dependency o  

> Task :dependencyInsight 
project :subproject1-vanilla
   variant "default" [
      Requested attributes not found in the selected variant:
         org.gradle.usage = java-api
   ]
\--- compileClasspath

project :subproject2-variants
   variant "compile" [
      quality          = b (not requested)
      org.gradle.usage = java-api
   ]
\--- compileClasspath

project :subproject3-java
   variant "apiElements" [
      org.gradle.usage = java-api
   ]
\--- compileClasspath

org:ivy-from-ivy:1.0
   variant "master+compile" [
      Requested attributes not found in the selected variant:
         org.gradle.usage = java-api
   ]
\--- org:ivy1:1.0
     \--- compileClasspath

org:ivy-from-pom:1.0
   variant "runtime+compile+master" [
      Requested attributes not found in the selected variant:
         org.gradle.usage = java-api
   ]
\--- org:pom1:1.0
     \--- compileClasspath

org:ivy1:1.0
   variant "default" [
      Requested attributes not found in the selected variant:
         org.gradle.usage = java-api
   ]
\--- compileClasspath

org:pom-from-ivy:1.0
   variant "master+compile" [
      Requested attributes not found in the selected variant:
         org.gradle.usage = java-api
   ]
\--- org:ivy1:1.0
     \--- compileClasspath

org:pom-from-pom:1.0
   variant "runtime" [
      Requested attributes not found in the selected variant:
         org.gradle.usage = java-api
   ]
\--- org:pom1:1.0
     \--- compileClasspath

org:pom-from-pom-from-ivy:1.0
   variant "compile" [
      Requested attributes not found in the selected variant:
         org.gradle.usage = java-api
   ]
\--- org:pom-from-ivy:1.0
     \--- org:ivy1:1.0
          \--- compileClasspath

org:pom1:1.0
   variant "default" [
      Requested attributes not found in the selected variant:
         org.gradle.usage = java-api
   ]
\--- compileClasspath
```


