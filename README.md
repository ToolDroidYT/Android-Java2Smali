# Java2Smali

A comprehensive Android library for converting Java source code to Smali assembly language. Java2Smali provides a complete pipeline for transforming Java files into human-readable Smali code through compilation, dexing, and disassembly.

I made this for my other project `ByteHaven` and decided to distribute it for others to use because I couldnt find any other libararies like this that works on Android, and I made it easy to use.

## Overview

Java2Smali orchestrates the complete transformation pipeline:

1. **Compilation**: Java source files → .class files (using javac)
2. **Dexing**: .class files → .dex files (using D8 or R8)
3. **Disassembly**: .dex files → .smali files (using baksmali)

## Features

- **Dual Dexing Options**: Support for both D8 (fast) and R8 (with optimization and obfuscation)
- **Classpath Management**: Separate boot classpath (android.jar) and user classpath support
- **ProGuard Integration**: Configurable ProGuard rules for R8 optimization
- **Status Callbacks**: Real-time status updates for UI integration
- **Partial Pipeline Execution**: Run individual stages (compile-only, compile+dex, etc.)
- **Multidex Support**: Automatic handling of multiple DEX files
- **Thread-Safe**: Asynchronous execution with proper concurrency controls

## Requirements

- Android API Level 26 or higher
- Android Gradle Plugin 7.0+
- Java 8 or higher

## Installation

### Gradle

Add the dependency to your module's `build.gradle`:

```gradle
dependencies {
        implementation("io.github.tooldroidyt:java2smali:1.1.1")
}
```

## Quick Start

### Step-by-Step

**1. Instantiate Java2Smali**

```java
// Instantiate Java2Smali
Java2Smali java2Smali = new Java2Smali(this);
```

**2. Adding Java Sources**

```java
// Add individual Java source files
java2Smali.setSourceFiles(new File("path/to/Main.java"), new File("path/to/Example.java"));
// Or provide a source directory (recursively discovers .java files)
java2Smali.setSourceFiles(new File("path/to/src"));

// Set output directory for generated Smali files
java2Smali.setOutputFolder(new File("/output"));

// Set temporary working directory for intermediate build artifacts
java2Smali.setTempFolder(new File("/.build"));
```

**3. Adding Dependencies**

Files you might need (boot classes):
  - [android.jar](https://github.com/ToolDroidYT/Android-Java2Smali/releases/download/1.1.1/android.jar): needed when you use Android java api (eg. android.app.Activity)
  - [core-lambda-stubs.jar](https://github.com/ToolDroidYT/Android-Java2Smali/releases/download/1.1.1/core-lambda-stubs.jar): needed when the source you're gonna compile contains lambda syntax


```java
// Boot classpath (e.g., android.jar): used for compilation only; not packaged
java2Smali.setBootClassPath(new File("android.jar"), new File("core-lambda-stubs.jar"));
// Alternatively, provide a directory containing framework jars
java2Smali.setBootClassPath(new File("path/to/folder/bootclasses"));

// Application classpath (e.g., mylibrary.jar): included in dex/smali output
java2Smali.setClassPath(new File("mylibrary.jar"), new File("morelibraries.jar"));
// Alternatively, provide a directory containing dependency jars
java2Smali.setClassPath(new File("path/to/folder/libraries"));
```

**4. R8 Configuration**

```java
// Optional: custom ProGuard/R8 rules file
File proguardFile = new File("proguard-rules.pro");

java2Smali.useR8(
    true,  // enable R8 (use D8 when false)
    true,  // release mode (enables optimizations)
    true,  // allow code minification/shrinking
    proguardFile // ProGuard/R8 rules file (null to use defaults)
);
```

**5. Listeners**

```java
// Listen to pipeline status updates (e.g., STARTING, COMPILING, DEXING)
java2Smali.setOnStatusListener(new OnStatusCallback() {
    @Override
    public void onStatus(Status status, String message) {
        Log.i("TAG", message);
    }
});

// Handle errors during compilation/dexing/disassembly
java2Smali.setOnErrorListener(new OnErrorCallback() {
    @Override
    public void onError(Exception exception) {
        // TODO: Handle error (e.g., show a message or retry)
    }
});

// Handle successful completion and use the output folder
java2Smali.setOnFinishListener(new OnFinishCallback() {
    @Override
    public void onFinish(File outputFolder) {
        // TODO: Use outputFolder (e.g., display path or open folder)
    }
});

// Run the full pipeline
java2Smali.start();
```

### Full Example

```java
// Instantiate Java2Smali
Java2Smali java2Smali = new Java2Smali(this);

// Add individual Java source files
java2Smali.setSourceFiles(new File("path/to/Main.java"), new File("path/to/Example.java"));
// Or provide a source directory (recursively discovers .java files)
java2Smali.setSourceFiles(new File("path/to/src"));

// Set output directory for generated Smali files
java2Smali.setOutputFolder(new File("/output"));

// Set temporary working directory for intermediate build artifacts
java2Smali.setTempFolder(new File("/.build"));

// Boot classpath (e.g., android.jar): used for compilation only; not packaged
java2Smali.setBootClassPath(new File("android.jar"), new File("core-lambda-stubs.jar"));
// Alternatively, provide a directory containing framework jars
java2Smali.setBootClassPath(new File("path/to/folder/bootclasses"));

// Application classpath (e.g., mylibrary.jar): included in dex/smali output
java2Smali.setClassPath(new File("mylibrary.jar"), new File("morelibraries.jar"));
// Alternatively, provide a directory containing dependency jars
java2Smali.setClassPath(new File("path/to/folder/libraries"));


// Optional: custom ProGuard/R8 rules file
File proguardFile = new File("proguard-rules.pro");

java2Smali.useR8(
    true,  // enable R8 (use D8 when false)
    true,  // release mode (enables optimizations)
    true,  // allow code minification/shrinking
    proguardFile // ProGuard/R8 rules file (null to use defaults)
);

// Listen to pipeline status updates (e.g., STARTING, COMPILING, DEXING)
java2Smali.setOnStatusListener(new OnStatusCallback() {
    @Override
    public void onStatus(Status status, String message) {
        Log.i("TAG", message);
    }
});

// Handle errors during compilation/dexing/disassembly
java2Smali.setOnErrorListener(new OnErrorCallback() {
    @Override
    public void onError(Exception exception) {
        // TODO: Handle error (e.g., show a message or retry)
    }
});

// Handle successful completion and use the output folder
java2Smali.setOnFinishListener(new OnFinishCallback() {
    @Override
    public void onFinish(File outputFolder) {
        // TODO: Use outputFolder (e.g., display path or open folder)
    }
});

// Run the full pipeline
java2Smali.start();
```

## Configuration

### Boot Classpath vs Classpath

**Boot Classpath** - Framework libraries used as references (not included in output):
- `android.jar`
- `core-lambda-stubs.jar`
- Other Android framework JARs

**Classpath** - Application dependencies (included in output):
- Your library JARs
- Third-party dependencies (Gson, OkHttp, etc.)
- Custom libraries

### Annotation Warnings

When using libraries like Gson that reference optional annotation dependencies (Error Prone, JSR-305, etc.), Java2Smali automatically suppresses warnings. __To disable__:

```java
java2smali.setIgnoreAnnotationWarnings(false);
```

This adds the following ProGuard rules automatically:
```proguard
-dontwarn com.google.errorprone.annotations.**
-dontwarn javax.annotation.**
-dontwarn org.checkerframework.**
-dontwarn org.codehaus.mojo.animal_sniffer.**
-dontwarn edu.umd.cs.findbugs.annotations.**
```

## API Reference

### Constructors

#### `Java2Smali(Context context)`
Creates a compiler instance with minimal configuration. Use setter methods to configure.

<!-- #### `Java2Smali(Context context, File[] sourceFiles, File outputFolder, File[] jarFiles)`
Creates a fully configured compiler instance. -->

### Configuration Methods

| Method | Description |
|--------|-------------|
| `setSourceFiles(File... source)` | Sets Java source files or directories to compile |
| `setOutputFolder(File outputFolder)` | Sets the output directory for smali files |
| `setBootClassPath(File... jars)` | Sets boot classpath (framework JARs) |
| `setClassPath(File... jars)` | Sets user classpath (dependency JARs) |
| `setTempFolder(File tempFolder)` | Sets temporary directory for build artifacts |
| `useR8(boolean useR8, boolean release, boolean allowMinification, File proguardRules)` | Configures R8 dexing options |
| `setIgnoreAnnotationWarnings(boolean ignore)` | Controls annotation warning suppression |

### Callback Methods

| Method | Description |
|--------|-------------|
| `setOnStatusListener(OnStatusCallback callback)` | Sets callback for status updates |
| `setOnFinishListener(OnFinishCallback callback)` | Sets callback for successful completion |
| `setOnErrorListener(OnErrorCallback callback)` | Sets callback for error handling |

### Execution Methods

| Method | Description |
|--------|-------------|
| `start()` | Runs complete pipeline: Compile → Dex → Disassemble → Cleanup |
| `startUntilCompiling()` | Runs: Compile only |
| `startUntilDexing()` | Runs: Compile → Dex |
| `startUntilBackSmali()` | Runs: Compile → Dex → Disassemble (no cleanup) |

### Status Enum Values

```java
public enum Status {
    STARTING,   // Pipeline initialization
    COMPILING,  // Java → .class
    DEXING,     // .class → .dex
    BAKSMALI,   // .dex → .smali
    CLEANING,   // Removing temporary files
    FINISHED,   // Complete success
    ERROR       // Fatal error occurred
}
```

## Troubleshooting

### "No Java compiler available"

Ensure you have javac available. On Android, the library includes an embedded javac.

### "Missing class com.google.errorprone.annotations"

Enable annotation warning suppression:
```java
compiler.setIgnoreAnnotationWarnings(true);
```

### "Compilation failed to complete" with R8

Check your ProGuard rules. Try disabling minification:
```java
compiler.useR8(true, true, false, null);
```

### No .class files produced

Verify your source files are valid Java and the paths are correct:
```java
File[] sources = compiler.getSourceFiles();
Log.d(TAG, "Found " + sources.length + " source files");
```

## Architecture

```
┌─────────────────┐
│  Java Sources   │
│   (.java)       │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│     javac       │
│  (Compilation)  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Class Files    │
│   (.class)      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   D8 or R8      │
│   (Dexing)      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   DEX Files     │
│   (.dex)        │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│    baksmali     │
│ (Disassembly)   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Smali Files    │
│   (.smali)      │
└─────────────────┘
```

## Dependencies

- **Android Tools R8**: DEX compilation and optimization
- **Smali/Baksmali**: DEX disassembly to Smali format
- **DexLib2**: DEX file parsing and manipulation

## License

```
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

<!-- ## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request -->

## Support

For issues, questions, or suggestions:
- Open an issue on [GitHub](https://github.com/ToolDroidYT/Android-Java2Smali)
- Contact: [notlifetechlolv2@gmail.com](mailto:notlifetechlolv2@gmail.com)
- Telegram: [ByteHaven Pro](https://t.me/ByteHavenPro)

## Acknowledgments

- Inspired by [Java2Smali by Belluxx](https://github.com/Belluxx/Java2Smali)
- Android Open Source Project for R8/D8
- [JesusFreke for Smali/Baksmali](https://github.com/JesusFreke/smali)
- All contributors to this project
