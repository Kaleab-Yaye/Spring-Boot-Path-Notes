so when you rigth code you souce code could look very much difffernt. so you migh havee heard the debate about using tabs and whitspaces and which one is better. and also abut the CRLF and Lf
talkes. those all are miner diffence that are visible to human but when a copueter reads them in teh ASII formating they creat a huge differnt this impacts the verstion control and you will see too many
uncalled for blames in the offciall repostery

so to stop this from happenin accros our projeccts we will implement what is called spotless plugin for mvn and here is what you add should look like and i will explain what teh most imporant lines means.
```xml
<plugin>
    <groupId>com.diffplug.spotless</groupId>
    <artifactId>spotless-maven-plugin</artifactId>
    <version>2.43.0</version>
    <configuration>
        <java>
            <googleJavaFormat>
                <version>1.17.0</version>
                <style>GOOGLE</style>
            </googleJavaFormat>
            <lineEndings>WINDOWS</lineEndings>
        </java>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>check</goal>
            </goals>
            <phase>verify</phase>
        </execution>
    </executions>
</plugin>
```

now look here is the thing  the first three lines are teh simples and ware definfig what the plug in is.

the text inside configration bunch is actaully what is reall setting the imporang configrations
* fso the java format that we are goign to use is google java format. one of the use of this format is it wil always change tab in to four consicative spaces
the line ending just fixes the new line issue when every one tryes to git and we are telling meven to fix every once code so that it will be in the windows encoding

<Excution>; this section is configraing and telling mevon what to do with in the maven phases with this plug in and plug inss are just MOJO classes what we call maven old java objects.
that has one methode so in the .jara file that all the classes for the spottles maven plug in we have chekc and apply so we are telling mav in the verrfy phase of your life cyle verfiy it.

but once we have this plug in we can run the follwoing in the root terminall

```powershell
/.mvnw spotless: apply # this will actually check our source code and make sure it is in the rigth format by applying the format we told it to repspect
/.mvnw spotless: check # this checks wok.
```

now if we trust the person in our project then that would be it they would run the spotless check and apply on the terminal but what if they foget to do that and commite it any ways.
now that would contamnate our entire repository with meangless blames. so instead we ulize what is called pre-commite hooks. those are scipts that git will run just beofere wen we run git commite -m.
now by making the /.mvnw spotless:apply/check scripts to run as the pre-commite hook we are makign sure teh commite wont happend if there is any violation in formating in our source code.




