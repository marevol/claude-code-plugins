---
name: java-source-code
description: >
  Use to fetch and display Java source code for a class that is NOT part of the current project (i.e. comes from a third-party dependency).
  TRIGGER when: user asks to show, read, or trace into a Java class/method that is not defined in the project's own sources, references a third-party class like "show me RestTemplate source" or "how does ObjectMapper work internally", or wants to browse a dependency's sources JAR.
  DO NOT TRIGGER when: the target class exists in the current project's source tree (use Read/Grep instead).
user-invocable: true
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
---

# Java Dependency Source Code Viewer

Resolve and display Java source code for classes that live in third-party dependencies, not in the current project.

The skill prefers the Maven local repository (`~/.m2/repository/`) and falls back to Gradle's cache (`~/.gradle/caches/modules-2/files-2.1/`) when needed. If sources are missing locally, it instructs the user how to download them via the project's build tool.

## Instructions

1. **Confirm the class is NOT in the project**
   - Run `grep -r "class {ClassName}" --include="*.java" .` (or use Glob on `**/{ClassName}.java`) at the repo root
   - If a match exists in the project, STOP and tell the user to read it directly (this skill is only for dependencies)

2. **Parse the user's input**
   Decide what was given:
   - **Maven coordinates** (`groupId:artifactId:version`, e.g. `org.springframework:spring-web:6.1.0`): use them directly
   - **Artifact hint** (`artifactId` or `groupId:artifactId`, e.g. `spring-web`): search the local cache
   - **Class name** (simple `RestTemplate` or fully-qualified `org.springframework.web.client.RestTemplate`): search inside `*-sources.jar` files
   - If two arguments are passed (e.g. `spring-web RestTemplate`), treat the first as the artifact hint and the second as the class

3. **Locate the artifact in the local cache**
   Try Maven first, then Gradle:
   - **Maven**: `~/.m2/repository/{groupId-with-dots-as-slashes}/{artifactId}/{version}/`
     - Search by artifactId: `find ~/.m2/repository -maxdepth 8 -type d -name "{artifactId}"`
   - **Gradle**: `~/.gradle/caches/modules-2/files-2.1/{groupId}/{artifactId}/{version}/`
     - Search by artifactId: `find ~/.gradle/caches/modules-2/files-2.1 -maxdepth 4 -type d -name "{artifactId}"` (sources live in a hash-named subdirectory)
   - **Class-name search across all cached sources JARs** (when only a class name is given):
     ```
     find ~/.m2/repository ~/.gradle/caches/modules-2/files-2.1 -name "*-sources.jar" 2>/dev/null \
       | while read jar; do jar tf "$jar" 2>/dev/null | grep -q "/{ClassName}\.java$" && echo "$jar"; done
     ```

4. **Pick a version**
   - If the user supplied a version, use it
   - Otherwise list candidates and recommend the latest (semver-sorted)
   - When the project has a build file (`pom.xml`, `build.gradle`, `build.gradle.kts`), prefer the version declared there

5. **Verify the sources JAR is present**
   - Look for `{artifactId}-{version}-sources.jar` next to the regular JAR
   - If missing, instruct the user how to download it:
     - **Maven project**: `mvn dependency:sources -Dartifact={groupId}:{artifactId}:{version}`
     - **Gradle project**: add `idea { module { downloadSources = true } }` or run `./gradlew dependencies --refresh-dependencies` after enabling sources in the IDE plugin
   - If the regular JAR is also missing, the dependency itself was never resolved — suggest `mvn dependency:resolve` or `./gradlew build`

6. **List or filter source files inside the JAR**
   - `jar tf {sources.jar} | grep '\.java$'` to enumerate
   - When a class name was requested, filter with `grep "/{ClassName}\.java$"`
   - On multiple matches, show the list and let the user choose

7. **Display the source**
   - Stream directly from the JAR without extracting:
     ```
     unzip -p {sources.jar} {path/to/File.java}
     ```
   - Warn the user if the file is very large (>1500 lines) before printing

8. **Handle errors**
   - Neither `~/.m2/repository` nor `~/.gradle/caches` exists → no local cache; ask the user to build the project first
   - Artifact not found in either cache → suggest running the project's build, or supply explicit Maven coordinates
   - Sources JAR missing → use the download commands in step 5
   - Class not found in the JAR → list classes in the same package as alternatives
