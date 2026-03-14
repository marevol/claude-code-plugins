---
name: maven-source
description: >
  Retrieve and display Java source code from Maven local repository (~/.m2/repository/).
  Use when the user wants to look up Java library source code, find a class implementation,
  or browse source files from Maven dependencies.
user-invocable: true
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
---

# Maven Source Viewer

Retrieve and display Java source code from the Maven local repository (`~/.m2/repository/`).

## Instructions

1. **Parse user input**
   Determine the input type from the user's query:
   - **Maven coordinates** (`groupId:artifactId:version`, e.g. `org.junit:junit:4.13.2`): Build the path directly
   - **Artifact name** (`artifactId` or `groupId:artifactId`, e.g. `junit` or `junit:junit`): Search for matching directories
   - **Class name** (simple or fully-qualified, e.g. `Assert` or `org.junit.Assert`): Search inside sources JARs
   - If additional arguments are provided (e.g. `/maven-source junit Assert`), treat the first as the artifact hint and the second as the class to find within it

2. **Locate the artifact**
   Based on the input type:
   - **Maven coordinates**: Navigate directly to `~/.m2/repository/{groupId with dots replaced by /}/{artifactId}/{version}/`
   - **Artifact name**: Run `find ~/.m2/repository -maxdepth 4 -type d -name "{artifactId}"` to find candidate directories
   - **Class name**: If fully-qualified (contains dots and last segment is capitalized), convert package to path and search with `find ~/.m2/repository -maxdepth 8 -name "*-sources.jar" | head -20` then `jar tf {jar} | grep "{ClassName}.java$"`

3. **Select version**
   - If a specific version was provided, use it directly
   - If multiple versions exist, list them and recommend the latest one
   - Let the user choose if they want a different version

4. **Verify sources JAR exists**
   - Check for `{artifactId}-{version}-sources.jar` in the artifact directory
   - If not found, inform the user and suggest running:
     ```
     mvn dependency:sources -Dartifact=groupId:artifactId:version
     ```
   - Also check if the regular JAR exists to confirm the artifact is present

5. **List or search source files**
   - Run `jar tf {sources.jar} | grep '\.java$'` to list all Java source files
   - If the user requested a specific class, filter for matching filenames
   - If multiple matches exist, show the list and let the user choose

6. **Display source code**
   - Extract and display the requested file using `unzip -p {sources.jar} {path/to/File.java}`
   - Use `unzip -p` to write directly to stdout without creating temporary files
   - If the file is very large, mention that to the user and display it in full

7. **Handle errors gracefully**
   - **`~/.m2/repository` does not exist**: Inform the user that no Maven local repository was found
   - **Artifact not found**: Suggest checking the artifact name or running `mvn dependency:resolve`
   - **sources JAR not found**: Suggest `mvn dependency:sources` with the correct coordinates
   - **Class not found in JAR**: List available classes in the same package or suggest alternatives
