# JavaFX Launch Configuration Guide

## Overview
This README explains how to configure `launch.json` for JavaFX projects in VS Code, what caused the error in your workspace, and how to fix it. It also provides detailed step-by-step guidance for all common JavaFX project scenarios.

## What happened in this workspace
Your project root is `D:\Project CSE 2100`, which contains spaces. That is fine, but the VS Code launch configuration must handle paths with spaces correctly.

The launch failure happened because the current `.vscode/launch.json` had two bad values:
- `cwd` was not set to `${workspaceFolder}`
- `vmArgs` used an unquoted module path and hardcoded folder name, which broke when the path included spaces

Because of that, Java read part of the path as a class name and failed with:
- `Could not find or load main class CSE`

## What you must change in `launch.json`
Every JavaFX launch configuration should include:
- `mainClass`: the fully qualified name of your Java application's main class
- `cwd`: `${workspaceFolder}` to ensure the program runs from the project root
- `vmArgs`: the JavaFX module path and required JavaFX modules, properly quoted if the path contains spaces

### Example correct `launch.json`
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "java",
            "name": "Launch JavaFX App",
            "request": "launch",
            "mainClass": "App",
            "cwd": "${workspaceFolder}",
            "vmArgs": "--module-path \"${workspaceFolder}/lib\" --add-modules javafx.controls,javafx.fxml"
        }
    ]
}
```

## Detailed explanations for each field
- `type`: must be `java` for the VS Code Java extension.
- `name`: any friendly name for the launch profile.
- `request`: use `launch` to start the application.
- `mainClass`: the entry point class name, including package if present.
- `cwd`: `${workspaceFolder}` makes sure Java runs inside your project root.
- `vmArgs`: the JavaFX SDK path and modules, with quotes around the path.

## All common JavaFX scenarios

### Scenario 1: Simple project with `App.java` in the root of `src`
If your class is `src/App.java` and not inside any package:
- `mainClass`: `App`
- `vmArgs`: `--module-path "${workspaceFolder}/lib" --add-modules javafx.controls,javafx.fxml`

### Scenario 2: Project uses a package
If `App.java` starts with `package com.example;`, use the fully qualified class:
```json
"mainClass": "com.example.App"
```
No other change is required.

### Scenario 3: Modular project with `module-info.java`
If your project contains `src/module-info.java`, verify that it has:
```java
module your.module.name {
    requires javafx.controls;
    requires javafx.fxml;
    exports com.example;
}
```
Then keep the same launch config:
```json
"vmArgs": "--module-path \"${workspaceFolder}/lib\" --add-modules javafx.controls,javafx.fxml"
```

### Scenario 4: Non-modular project without `module-info.java`
Same launch configuration applies. JavaFX is still loaded using `--module-path` and `--add-modules`.

### Scenario 5: Maven or Gradle JavaFX project
If you use Maven/Gradle, the Java extension often discovers the classpath automatically.
Your `launch.json` can be minimal:
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "java",
            "name": "Launch JavaFX App",
            "request": "launch",
            "mainClass": "com.example.App"
        }
    ]
}
```
If JavaFX still fails, add `vmArgs` as shown earlier and make sure dependencies are correctly declared in `pom.xml` or `build.gradle`.

### Scenario 6: JavaFX SDK installed outside the workspace
If your JavaFX jars are not inside `${workspaceFolder}/lib`, use an absolute path in `vmArgs`:
```json
"vmArgs": "--module-path \"C:/javafx-sdk-26.0.1/lib\" --add-modules javafx.controls,javafx.fxml"
```
Use forward slashes or escaped backslashes with Windows paths, and always quote the module-path if it contains spaces.

### Scenario 7: Running from a different output folder
If your compiled classes are emitted to `out` or another folder, VS Code usually handles this automatically. If you need to specify it manually, use the Java project settings rather than `launch.json`.

## What to change for each common failure

### 1. `Could not find or load main class CSE`
Change `cwd` to `${workspaceFolder}` and quote the module path:
```json
"cwd": "${workspaceFolder}",
"vmArgs": "--module-path \"${workspaceFolder}/lib\" --add-modules javafx.controls,javafx.fxml"
```

### 2. `Error: JavaFX runtime components are missing`
Add or fix `vmArgs` with the JavaFX SDK path and modules.

### 3. `java.lang.ClassNotFoundException: com.example.App`
Use the full package name in `mainClass` or confirm that the compiled class is in the correct output folder.

### 4. `WARNING: A restricted method in java.lang.System has been called`
This warning appears when JavaFX native access is restricted in JDK 17+ environments.
To suppress it, add:
```json
"vmArgs": "--module-path \"${workspaceFolder}/lib\" --add-modules javafx.controls,javafx.fxml --enable-native-access=javafx.graphics"
```

## Working with path spaces and quoting
Paths containing spaces are the most common source of launch issues.
- Always use `${workspaceFolder}` instead of a hardcoded directory name.
- Always put the module path in quotes when the path contains spaces.
- Avoid using unquoted Windows paths like `D:\Project CSE 2100\lib` directly in `vmArgs`.

Example correct quoting:
```json
"vmArgs": "--module-path \"${workspaceFolder}/lib\" --add-modules javafx.controls,javafx.fxml"
```

### Why quoting matters
When Java sees an unquoted path with spaces, it treats the path portions after each space as separate arguments. In your case, the workspace name `Project CSE 2100` was split, causing Java to look for a class named `CSE`.

## Full example for several common launch.json cases

### Case A: Basic JavaFX app in default package
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "java",
            "name": "Launch JavaFX App",
            "request": "launch",
            "mainClass": "App",
            "cwd": "${workspaceFolder}",
            "vmArgs": "--module-path \"${workspaceFolder}/lib\" --add-modules javafx.controls,javafx.fxml"
        }
    ]
}
```

### Case B: JavaFX app in a package
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "java",
            "name": "Launch JavaFX App",
            "request": "launch",
            "mainClass": "com.example.App",
            "cwd": "${workspaceFolder}",
            "vmArgs": "--module-path \"${workspaceFolder}/lib\" --add-modules javafx.controls,javafx.fxml"
        }
    ]
}
```

### Case C: Modular JavaFX app
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "java",
            "name": "Launch JavaFX App",
            "request": "launch",
            "mainClass": "com.example.App",
            "cwd": "${workspaceFolder}",
            "vmArgs": "--module-path \"${workspaceFolder}/lib\" --add-modules javafx.controls,javafx.fxml"
        }
    ]
}
```

In this case, ensure `module-info.java` contains:
```java
module com.example {
    requires javafx.controls;
    requires javafx.fxml;
    exports com.example;
}
```

### Case D: Maven/Gradle JavaFX app
If your build file manages JavaFX, a simple launch config is usually enough:
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "java",
            "name": "Launch JavaFX App",
            "request": "launch",
            "mainClass": "com.example.App"
        }
    ]
}
```

If launch still fails, add the same `vmArgs` section and verify your dependency declarations.

## Commercial readiness checklist
1. `launch.json` must use `${workspaceFolder}` for `cwd`.
2. Module paths must be quoted if they include spaces.
3. `mainClass` must match the exact class name and package.
4. Use a build tool (Gradle/Maven) for dependency management.
5. Keep JavaFX SDK or libraries in a dedicated folder such as `lib`.
6. Test on both your local machine and a clean build environment.
7. Package the app with `jpackage` or another installer for distribution.

## Final recommendation
The main fix in your case was:
- Use `${workspaceFolder}` in `cwd`
- Quote the JavaFX module path in `vmArgs`
- Point `mainClass` to the actual application class

This README now includes the exact changes you should make for every major JavaFX launch scenario, plus the commercial guidance needed for consistent run behavior.
`