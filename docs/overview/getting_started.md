## Getting started

To get all neow3j features add the `io.neow3j:contract` project to your
dependencies. Because neow3j is split into multiple project modules you can also
depend on a subset of the functionality for example if you only require certain
utility methods.


### Releases

__Gradle__

```groovy
compile 'io.neow3j:contract:2.+'
```

__Maven__

```xml
<dependency>
    <groupId>io.neow3j</groupId>
    <artifactId>contract</artifactId>
    <version>[2.0.0,3.0.0)</version>
</dependency>
```

### Development Snapshots

If you would like to test `SNAPSHOT` versions, which preview the most recent
features but are unstable and not ready for production, use the following
repository.

__Gradle__

```groovy
repositories {
    maven {
        url 'http://oss.sonatype.org/content/repositories/snapshots'
    }
    mavenCentral()
}
```

```groovy
compile 'io.neow3j:contract:2.2.0-SNAPSHOT'
```

__Maven__

```xml
<repositories>
    <repository>
        <id>sonatype-snapshots</id>
        <name>OSS Sonatype Snapshots</name>
        <url>http://oss.sonatype.org/content/repositories/snapshots</url>
        <releases>
            <enabled>false</enabled>
        </releases>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
</repositories>
```

```xml
<dependency>
    <groupId>io.neow3j</groupId>
    <artifactId>contract</artifactId>
    <version>2.2.0-SNAPSHOT</version>
</dependency>
```