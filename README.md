# how-to-make-digirunner-opensource-launch-standalone

How to make [digirunner opensource](https://github.com/TPIsoftwareOSPO/digiRunner-Open-Source) launch on the host system without container

## Prerequisite

- [java 21 installed](https://www.azul.com/downloads/?package=jdk#zulu)


## Step 1: Download latest release build from digiRunner Opensource Repository

download `digiRunner-Open-Source-<release tag>-h2-build.zip` file from [release](https://github.com/TPIsoftwareOSPO/digiRunner-Open-Source/releases)


_At the time of writing this article, only version test-release-action-11 was available for download. The examples below use this version._

```
wget https://github.com/TPIsoftwareOSPO/digiRunner-Open-Source/releases/download/test-release-action-11/digiRunner-Open-Source-test-release-action-11-h2-build.zip
```

## Step 2: Unzip digiRunner compress file

unzip digiRunner compress file, the file structure should be like:

```
digiRunner-Open-Source-test-release-action-11-h2-build/
├─ lib/
├─ keys/
├─ libsext/
├─ digirunner.jar
```

## Step 3: Download Task-Compose Portable Executable

- download the latest portable version of **task-compose** from the [releases page](https://github.com/vulcanshen-tpi/task-compose/releases). 

- Be sure to pick the one that matches your operating system. The filenames for these versions will start with `task-compose-portable-`.

_At the time of writing this article, only version v1.0.1 was available for download. The examples below use this version._

```
wget https://github.com/vulcanshen-tpi/task-compose/releases/download/v1.0.1/task-compose-portable_1.0.1_darwin_arm64.tar.gz
```

## Step 4: Unzip Task-Compose compress file

unzip task-compose compress file, the file structure should be like:

```
task-compose-portable_1.0.1_darwin_arm64/
├─ task-compose-portable
├─ README.md
├─ LICENSE
```

## Step 5: Copy Executable

copy `task-compose-portable(.exe)` file to digiRunner unzip directory, the file structure will be like:

```
...-h2-build/
├─ lib/
├─ keys/
├─ libsext/
├─ digirunner.jar
├─ task-compose-portable(.exe) <-- paste to here
```

## Step 6: Write Your Own `task-compose.yaml` file

create `task-compose.yaml` in digiRunner unzip directory

content of `task-compose.yaml`
```yaml
tasks:
  - name: digirunner-opensource
    executable: java
    base_dir: .
    args:
      - -jar
      - digirunner.jar
```

so currently the file structure should be:

```
...-h2-build/
├─ lib/
├─ keys/
├─ libsext/
├─ digirunner.jar
├─ task-compose-portable(.exe)
├─ task-compose.yaml
```

then double click(execute) the `task-compose-portable(.exe)`

the digiRunner Opensource startup process should begin

---

# Example Configuration

### Example 1

This example demonstrates how to launch the application without any prior Java installation, ensuring a clean, portable startup.

**Step 1** Download zulu jre 21 **zip** package

[Azul Zulu](https://www.azul.com/downloads/?package=jdk#zulu)

**Step 2** Unzip jre

```
zulu21.42.19-ca-jre21.0.7-macosx_aarch64/
├─ zulu-21.jre/
├─ bin/
├─ conf/
├─ ...
```

**Step 3** Move jre directory to h2-build/ directory

```
...-h2-build/
├─ lib/
├─ keys/
├─ libsext/
├─ digirunner.jar
├─ task-compose-portable(.exe)
├─ task-compose.yaml
├─ zulu21.42.19-ca-jre21.0.7-macosx_aarch64/
```

task-compose.yaml

```yaml
tasks:
  - name: digirunner-opensource
    executable: ./zulu21.42.19-ca-jre21.0.7-macosx_aarch64/bin/java # <-- change this line
    base_dir: .
    args:
      - -jar
      - digirunner.jar
```

This setup will use the downloaded JRE to launch digiRunner, eliminating the need to install Java on your system beforehand.

### Example 2

This is a configuration for a custom service that sets the port number, specifies the H2 file-based database, and automatically initializes the database.

```yaml
tasks:
  - name: digirunner-opensource
    executable: java
    base_dir: .
    args:
      - -jar
      - digirunner.jar
      - --server.port=31080 # Change to your preferred port.
      - --spring.datasource.url=jdbc:h2:./dgrdb;NON_KEYWORDS=VALUE;Mode=MySQL # Set your preferred JDBC URL.
      - --spring.sql.init.mode=always # Set to "always" for initial database setup; "never" otherwise.
```

### Example 3

Because Example 2 requires you to manually change the spring.sql.init.mode parameter to `never` upon restart, this example automates the detection of the database file and adjusts the parameter accordingly.

```yaml
tasks:
  - name: db-detect
    executable: bash
    base_dir: .
    args:
      - -c
      - |
        rm -f digirunner.args
        cat << EOF > digirunner.args
        -Dserver.port=31080
        -Dspring.datasource.url=jdbc:h2:./db/dgrdb;NON_KEYWORDS=VALUE;Mode=MySQL
        EOF
        ls ./db/*.db &> /dev/null && echo '-Dspring.sql.init.mode=never' >> digirunner.args || echo '-Dspring.sql.init.mode=always' >> digirunner.args;
    healthcheck:
      frequency:
        delay: 1s
      command:
        scripts:
          - ls
          - digirunner.args
  - name: digirunner-opensource
    executable: java
    base_dir: .
    depends_on:
      - db-detect
    args:
      - -cp
      - digirunner.jar
      - "@digirunner.args"
      - org.springframework.boot.loader.launch.PropertiesLauncher
```


For more settings, please refer to the [digiRunner-Open-Source](https://github.com/TPIsoftwareOSPO/digiRunner-Open-Source) Repository