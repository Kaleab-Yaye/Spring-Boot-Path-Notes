ok so here are hte most imporant points you will need that are realted about maven at our begginign phase.

but before we start lets look at some imporatn points you shoudl remmeber.

all the classes that we compiled from our srouce code all the .jar files in our source code is foudn in the Target directory. this target directory is never commited and is ingroned by git commite
and this is possible thanks to the spring boot provided .gitignore file where it invovled all the things that git should ignore.

with that in minde here are the phases of the maven_LifeCycle

*   `validate`: Checks if the project is valid and all necessary information is available.
*   `compile`: Compiles the source code of the project.
*   `test`: Runs the unit tests. (This is what our pre-commit hook uses).
*   `package`: Takes the compiled code and packages it in its distributable format, such as a JAR.
*   `verify`: Runs any checks on the results of integration tests to ensure quality criteria are met. **(This is where we hooked Spotless in our `pom.xml`).** It's a perfect place for a final quality check *after* testing but *before* the artifact is considered "installed."
*   `install`: Installs the package into the local repository (`.m2` folder), for use as a dependency in other projects on your local machine.
*   `deploy`: Copies the final package to a remote repository for sharing with other developers and CI/CD systems.

now lets give some explnation on how i understand them

* `validate`: just cheks if the things that are need are there
* `Compile`: happend after validation ( if you call compile , validation will happen first) this will copile your source code in to the .class filles
* `test`: this is not the same us compilation and this is jsut a test run ( uniti test) we do. it test the logic is intact like the projet wide bussines logic rules that hsould be repected like a fields that should never be 
a null and so on. we defines those testsss letter on 
* `pakage ` now this one takes the compiled files and put them in a distruibitable that is in the form of the .jar file
* `verfiy ` this step reall dones do much by default but when we want to hook adtionlad checks on the src code like teh maven spottles formating then we it will do it beforre the pakcge is install in to our mave .m2 dir
 to be used by othere futre projects
* `insall ` this actuall puts the .jar file of the resutl fo the pakege phase to be here
* `deploy ` thsis reserved command for the final phase and only run on the server that will host the whole thing.

now there is addtion phase called clean. what this does is it just deletes the entirity of the target class.

but what we will actually run in our day to day cycle is the comman `./mvmw clean install ` this will delete the target fodler and then does all that step pressiding `install` phase. this is clean and safe.


