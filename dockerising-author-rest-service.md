Spotify has a great Maven plugin \([https://github.com/spotify/docker-maven-plugin](https://github.com/spotify/docker-maven-plugin%29\), which builds a docker image via Maven.

To use this plugin we are going to add the `docker.image.prefix` property to the`pom.xml`

```
<properties>
    ...
    <docker.image.prefix>joaovicente</docker.image.prefix>
</properties>
```

Add the plugin

```
<plugin>
        <groupId>com.spotify</groupId>
        <artifactId>docker-maven-plugin</artifactId>
        <version>0.4.11</version>
        <configuration>
                <imageName>${docker.image.prefix}/${project.artifactId}</imageName>
                <imageTags>
                        <imageTag>${project.version}</imageTag>
                        <imageTag>latest</imageTag>
                </imageTags>
                <baseImage>frolvlad/alpine-oraclejdk8:slim</baseImage>
                <entryPoint>["java", "-jar", "/${project.build.finalName}.jar"]</entryPoint>

                <!--<dockerDirectory>src/main/docker</dockerDirectory>-->
                <resources>
                        <resource>
                                <targetPath>/</targetPath>
                                <directory>${project.build.directory}</directory>
                                <include>${project.build.finalName}.jar</include>
                        </resource>
                </resources>
        </configuration>
</plugin>
```

You can now build the Docker image using the command below

```
mvn clean package docker:build
```

When you look at the images

```
docker images | grep joaovicente
```

You should be able to see the images listed

```
joaovicente/author    0.0.1-SNAPSHOT    00224d6e274f    25 seconds ago    182MB
joaovicente/author    latest            00224d6e274f    25 seconds ago    182MB
```

You can now run the docker image as follows:

```
docker run -p 8080:8080 joaovicente/author
```



