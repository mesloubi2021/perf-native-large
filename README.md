# Large Native Performance Reproduction Project

Currently, this doesn't look like a very large project. But, it is. We
just have to take a few steps to generate the sources from scratch
while we are working out whatever inaccuracies there might have been
in our code generation scripts.

## Getting Started

Clone the repo:
```sh
$> git clone https://github.com/gradle/perf-native-large.git
$> cd perf-native-large
```

From here you need to run the generation script. It takes about 4 minutes to do it's work.
```sh
$> cd generator
$> ./gradlew run
```

What it's doing is populating 430 project directories in the root of
this repository and the corresponding `settings.gradle` file which
ties the projects together in one big happy multiproject build.

Specifically, it is parsing the
[components.txt](generator/components.txt) file and generating c++
source code, header files and references to prebuit native libraries
on your platform. For each othese projects and all their complicated
interdependencies.

Speaking of prebuilt libraries. We need to *trick* the top-level build
into thinking that there are some prebuilt binaries, so we can
satisfiy the needs of the generated `:project431` which has all ton of
`PrebuiltLibrary` entries.

Do do that run:
```sh
$> cd ../prebuilt/util
$> ./gradlew assemble
```

At this point, you should be able to pop back up to the top level and run the build.

```sh
$> cd ../
$> gradle assemble
```

*Note* I haven't added a top-level `build.gradle` file yet because I
 didn't want people to see it in the repo and think this project would
 behave just like any other one you've used before.

## Profiling with honest-profiler

There is a script [`prof.sh`](profiler/prof.sh) which automates running honest-profiler and creating a flamegraph of the execution.

These environment variables must be set to run the script:
- `JAVA_HOME` : path to the JDK
- `HP_HOME_DIR` : path to an installation of [honest-profiler](https://github.com/RichardWarburton/honest-profiler). This must be compiled from sources since it requires some changes that are in master branch.
- `FG_HOME_DIR` : path to an installation of [FlameGraph](https://github.com/brendangregg/FlameGraph). This is a clone of the [FlameGraph repository](https://github.com/brendangregg/FlameGraph).
- `GRADLE_BIN` : path to the gradle binary which will be used to execute the builds. 

Example command
`GRADLE_BIN=~/.sdkman/gradle/3.1-rc-1/bin/gradle ./profiler/prof.sh assemble`


## Profiling with gradle-profiler

Clone [gradle-profiler](https://github.com/adammurdoch/gradle-profiler) to some directory and build it [according to the instructions](https://github.com/adammurdoch/gradle-profiler/blob/master/README.md).

Example commands to do profiling
```
sdk u gradle 3.1-rc-1
gradle wrapper
echo 'org.gradle.jvmargs=-Xms8g -Xmx8g' >> gradle.properties
../gradle-profiler/build/install/gradle-profiler/bin/gradle-profiler --project-dir $PWD assemble
jmc -open profile.jfr
```

Make sure the project has a wrapper created with the version of Gradle that is being tested.