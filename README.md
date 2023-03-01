# Resolved Dependencies Problem

This is a minimal reproduction of a failure in the test suite for Spring Boot's Gradle plugin.
The problem only occurs when running with `--configuration-cache` and with Gradle 8.

In Spring Boot's Gradle plugin we have [some code][1] that maintains a mapping of a dependency's `File` back to its GAV coordinates and a flag for whether or not it was a project dependency.
This information about resolved dependencies is [populated using an after resolve hook][2].
When run on Gradle 8.0.1 using `--configuration-cache`, information about dependencies that are on the runtime classpath goes missing.

The `build.gradle` file in the root of this repository contains a simplified and reduced approximation of the code in Spring Boot.
It should reproduce the behavior described above.

First, run `./gradlew customWar` and the build should pass:

```
$ ./gradlew customWar

BUILD SUCCESSFUL in 929ms
4 actionable tasks: 4 executed
```

Then run `./gradlew clean customWar --configuration-cache` and the build should fail:

```
$ ./gradlew clean customWar --configuration-cache
Configuration cache is an incubating feature.
Reusing configuration cache.
> Task :customWar FAILED

FAILURE: Build failed with an exception.

* Where:
Build file '[…]/resolved-dependencies-problem/build.gradle' line: 22

* What went wrong:
Execution failed for task ':customWar'.
> Could not find […]/resolved-dependencies-problem/three/build/libs/three.jar

* Try:
> Run with --stacktrace option to get the stack trace.
> Run with --info or --debug option to get more log output.
> Run with --scan to get full insights.

* Get more help at https://help.gradle.org

BUILD FAILED in 691ms
8 actionable tasks: 8 executed
Configuration cache entry reused.
```

We have [very similar code in Spring Boot for jar archives][3] and it's not affected by the problem.
This reproduction also demonstrates this behavior:

```
$ ./gradlew customJar

BUILD SUCCESSFUL in 1s
4 actionable tasks: 4 executed
```

```
$ ./gradlew clean customJar --configuration-cache
Configuration cache is an incubating feature.
Calculating task graph as no configuration cache is available for tasks: clean customJar

BUILD SUCCESSFUL in 708ms
8 actionable tasks: 8 executed
Configuration cache entry stored.
```

It continues to work when a configuration cache entry is available for re-use:

```
$ ./gradlew clean customJar --configuration-cache
Configuration cache is an incubating feature.
Reusing configuration cache.

BUILD SUCCESSFUL in 637ms
8 actionable tasks: 8 executed
Configuration cache entry reused.
```


[1]: https://github.com/spring-projects/spring-boot/blob/2.7.x/spring-boot-project/spring-boot-tools/spring-boot-gradle-plugin/src/main/java/org/springframework/boot/gradle/tasks/bundling/ResolvedDependencies.java
[2]: https://github.com/spring-projects/spring-boot/blob/29a16a6428fab572c00646bd8812d84bb25fd0cf/spring-boot-project/spring-boot-tools/spring-boot-gradle-plugin/src/main/java/org/springframework/boot/gradle/tasks/bundling/BootWar.java#L83-L90
[3]: https://github.com/spring-projects/spring-boot/blob/29a16a6428fab572c00646bd8812d84bb25fd0cf/spring-boot-project/spring-boot-tools/spring-boot-gradle-plugin/src/main/java/org/springframework/boot/gradle/tasks/bundling/BootJar.java#L83-L90