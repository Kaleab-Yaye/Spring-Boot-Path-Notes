Ok, so now our API can talk to the worl with our API endpoints but it doesn have memroy or storage to sotre thigns we need it to work on and sotre.
so now we need to integrate with our data base that is running on the docker.
so how do it lets see

#ORM
so before ORMs where a thing when you wnat to manipulate your Data base you had to litrlay put SQL langue prompts, or commands. this was tediou and pron to erro and was risky as sql injectio is a thing.
but ow we use *Object Relational Mapping*( ORM) thos are librareys and depednacyes that will trnaslate you Object Orintted command and obect in to ENtitiyes that a data base ccan sotre or work with.

in the java world the standard ORM is JPA ( Jackarta persistent API). this deffines the set of abstract methodes that any one who want to talk to a relational data base should impliment and know about.
and there need to be an actuall engine that knows how to talk to the realtional data base bassed on the rules that are defined in the jap and for this we use wha tis called, mos of the time. **hibernate**
a standard library that comes by default when you include JPA as you dependacy in spring Boot.

# Now how does the magic Really Happen?

the thing is there are 3 thing that work togethe that makes the magic happend
## JPA
this as we said defines the basic rules how one ccna comunicate with the data base
## Hiberante
this one bassed on the ruels defiend by the JPA actually makes the connection with the data base, persistes it and so on this isthe tool that has the final access to the data base
## The Spring Data JPA
this is the part that actaully Impliments defined by JPAREPo extendidng Classe. we will see how this happens

. so lets assume you have a class anontate wiht `@Enitity` now at start up hibeyourate look at this class checks if there are feilds, and it will genrate the a table for the entitiiy. ( we will seee how)
* now anothere interphase class called `UserREpository` extends `JPARespository` and it defines the methodes in there like `User findByEmail()` now. here is were the magic of the SPring data JPA Shines So the firs thing
the SDJPA does is it looks at this class and then it creates A proxy Class object for it at runtime. now usign somthing called reflaction it looks teh entitiy that is in interse is User, and the hiberante has most
defintly ceated a table for it. now it lookd somthign like findByEmail() then it parse it with its won entrnal logic, ok so the user is expecting me to find them  data in teh tabale Enityty, based on Email. then it goes to the
User bean and opend it up and sees if there is a field `email` there there is then it goes back, and wirtites teh actaull implimentaiton that i needed so that the hibernate can do the Job, adn finilie the bean adn every thign
is ready. now the magic part is you aouto wire that bean, and you pefrome things you want it to do by calling the onse intersase but now actually impliment methodes that has all the logic to talk to hibernate and tell it to do thisngs.
**so in spring boot naming reall matters as that is how most of the libraray will tray to parse you code and info**

# incorporating the JPA into our code base
so we need to inlude JPA itself, and Data Base Driver, thing of it this way the JPA is the set of rules like what do you do when you want to get a user, you do thsi command but what are the syntax of that command
it depend on the type of data base we are using so we will use the postgres data base driver.
so now we will bring the `Spring boot stater data Jpa` whihc will bring the jpa API and aslo the srpign boot data jpa that does most of the magic
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```
now that we have teh spring data jpa, we need to add one thing but before we add it lets talk abou somthing called JavaDataBaseConectivity *JDBC*.
what this really is a driver that Java must be able to be talk to with Data base. that is all we need to knwo not to fall rthough the rabit hole. but you shoudl know evey data base teaccm developes this JDB for there
own data base so java lanue using application can talk with the data base. and when they wrigh this we call it JDBC-driver. the same way a certain driver for mouth changes an electic signal for usb port in to an actuall
cursor acffecting information, the sae thing happend here this direver povides all the ways librayes like hibernate can use to talk to the data base. so we get this from the internate

```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
</dependency>
```

now we can add thos ein the pom.xml file and we have a way to do ORM and actually talk to a data base with JDBC.

now we have the ORM and the JDBC ready, but which data base do we actual need our postgres JDBC driver to talk to, 
and for this we use what is called data source configration that will configer a data source bin that will letter given to our JDBC to work on here is the data souuce configation that we are going to use,
bassed on teh docker contained posttgress server that is running for so we go to our `applicatio-dev.yaml` proportiees file and then add the following in there

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/streaming_api_db
    username: devuser
    password: devsecret
  jpa:
    hibernate:
      ddl-auto: none # This is CRITICAL
```
so lets explain what is goign on;
* data source configers out data soruce object that hibernat or the jdbc will use to make conncetion and quieryed in to out data base
* url: here as you can see we are tellig the spring to ceat the data source object with t jDBC drive for psogresql. so srpign boot goes there creats `HikariDatasource` object bean.
* Jpa: this part is is nor realted too the data base directly but to spring boot, we are telling spring boot specifically to use Hibernate ( the emplimentation of JPA)
* ddl-auto: there two types of langues in relatinal data base DDL, adn DML, now DDL envolves, ceattging a table, droping a table,  and changin the scema of the entire data base. now by defualt the hibernat will copare
* our src code class with Entitiy and try to compare them with the tabel on the data base ad whenn this happend if it sees a differen between the two we dont kwno whta it is going to do will it only rename the table?
will it only drop the clomon? or drop the entire tabel? we dont knwo and entirly deepedn on the balck box code of how hibernate oprated bassed on that specific verion. so we don tneed this to happedn so
we specificall set this to non; meanign hibernat even when run for the frist time will no be able to ceate a table or do DDL manupaltions.



now basses on this lets trace the logic.

so our spring data jpa has some kind of query it need to be complected, so it talkes to Hibernate, then hiberante will change the qeusry in to an SQL command. 
now hibernate has an SQL commadn but doesn have a way to talk to the data base.( sotorey 1)

in the othere side of the world sping has already crreated an objec bin that is oprated by spring called DataSource. up on its creation.
* it knows the url to the data base
* it knows the user name and pasword
* and more importanly it has an obejct called pgConnection, that empliments the JDBC interphases

( the two worlds collided)
* now spring wil inject this data source object in to hibernate
* hibernate will call .getConnection()( or smtn like that) the data source will give one of its pgConnection
* then on this object Hibernate will call the methodes it was meant to be called bassed on the JDBC interface( now hibernat dont have to work about the specific driver, the password and staff)
* thins will get done 

then we run `.mvnw spring-boot:run`

now if that is succefull then we need to do one thing if our hibernate is actaully set up. so for the Hibernate commoponent *Entitiy Manager* to be spownede and seen at work it needs on Enitity anotated class.

Enitity is at the core of the ORM and represents a table in a data base. 

so we ceat a new packege called domain. domain is the conventiaonl place to store core bussines objects. 

and this is the code

```java
package com.adnakiwoch.platform.streaming_api.domain;


import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.Table;

@Entity
@Table(name = "Platform Users")
public class User {
    @Id
    private long id;

    private String name;

    private String email;


}
```

problmes that we run in to:

so if you remmebr we had a test that 

```java
void apiKeyMustNotBeBlank() {
    // so first we load our calss in to the context runner
    this.contextRunner
        .withUserConfiguration(
            StreamingApiApplication.class) // this is where we load our application class
        .withPropertyValues(
            "platform.billing.trail-period-days=30") /*this is where we specify the propties our memmeroy only exisiting aplicationproperies file
                                                     and as you can see we are creting a propties for the trialPeriodDays field which weillbe populated with it but
                                                     on perpouse leaving the api-key proprtie empety so it would lead to the failing of the aplication context when run*/
        .run(
            context -> {
              /*when you run this conntext runner runs it return an object we could name that what evver we wan
              but it has all the information that we need abotu the start of the app */
              assertThat(context)
                  .hasFailed(); // if the app fiald then this wontn though an error and we know
              // ommitin of the api-key coused this
              assertThat(context)
                  .getFailure()
                  .hasRootCauseInstanceOf(BindValidationException.class);
            });
  }
```

now this was our test. and our test was entirly build to see if the starter validation does its job the start and stopes our application adn this was fine as starter validation came before all the otehre
out starter configations that we have seen the test pass many time.
but today with out work we add the spring-bot-start-data depednacy and it happnes to be the case this dependacy is what runs before the `validation dependancy` so what happend is the applcaion will fail to start
but when we asser the reason we where expeting was `BindValidationException.class` but now the issues of it failling is actually `DatasourceNotFOundException` or somthign like that.
so the key take away is in teh future i could be wirting many tests that worked adn didnt then when i add a dependacy the reaosn for this is

1. addin a starter depedacy, adds a boot configration race that will couse one cofigratin to run befoe the othere, and if one configration fails all the coming will fail explainin why out test faild.

with thsi in mind when you run in to such an isseu youcan fix it by goolig which depdeacy configration faild and which one you shodld exclude for you work, thatn not kow what happend.

so the fixed code; 

```java
void apiKeyMustNotBeBlank() {
    // so first we load our calss in to the context runner
    this.contextRunner
        .withUserConfiguration(
            StreamingApiApplication.class) // this is where we load our application class
        .withPropertyValues(
            "spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration",
            "platform.billing.trail-period-days=30") /*this is where we specify the propties our memmeroy only exisiting aplicationproperies file
                                                     and as you can see we are creting a propties for the trialPeriodDays field which weillbe populated with it but
                                                     on perpouse leaving the api-key proprtie empety so it would lead to the failing of the aplication context when run*/
        .run(
            context -> {
              /*when you run this conntext runner runs it return an object we could name that what evver we wan
              but it has all the information that we need abotu the start of the app */
              assertThat(context)
                  .hasFailed(); // if the app fiald then this wontn though an error and we know
              // ommitin of the api-key coused this
              assertThat(context)
                  .getFailure()
                  .hasRootCauseInstanceOf(BindValidationException.class);
            });
  }

```
the main key work here is `"spring.autoconfigure.exclude= `org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration",
i shodul remmeber this is teh key to exluding the aoutconfigration that came before the target configration for the test.




