Spotify has a great Maven plugin \([https://github.com/spotify/docker-maven-plugin](https://github.com/spotify/docker-maven-plugin\)\), which builds a docker image via Maven.

To use this plugin we are going to add the `docker.image.prefix` property to the`pom.xml `

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



