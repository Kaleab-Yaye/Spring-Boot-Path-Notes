# First Endpoint: Health Check (Hello API).

So lets say I am going to make a Get requist in to a srping boot running aplication and my requist is `http://localhost:8080/api/v1/health`. now at that pot a spring boot aplicaton is running and when a get requist with tha 
address is recived is it goes to any class taht s anoonated with `@RestController` then it will find bunch of them then it will go one step down searching for any class that is annotated with `RequistMapping("apit/v1/health/)`
now inside this class there must be or shoul be a `GetMapping()` annotated methode and spring boot will return what ever this methode is confifured to return. but now lets take a look at how all this happens

# the magic of jakson
so for spring boot a return from any methode inside a `@RestController` it is meant to be treated as an object that should be translated in to jason.
so lets assume there is a Student object and this obejct has feilds like name, age, grade. now for some reason we wanted to return this student object to whom ever is making the get request.
now the methodde will return the object, and spring boot knwo this object is coming from inside a methode that was inside @RestContorller annonated class. so it know it has to change it to jason before returing it to the
get caller. now spring uses a powerfull libraray jakson to do what is needed. 

jakson is a library that scans the proprites of an object and then tanslates them in to jason compatible format. so this studnet object shoudl have feilds like getName(), getAge() etc. now when jakson sees this.
it takes the getName -->name: and after using that methode to get the value in that object it sortes it in the format {"name": "abebe", "aget": 22"}.
it hands it over to spring-boot and spring boot returns it, thiw works for all *Plaing-Old-Java-Objects(POJO)*

*for recordes it is the same it might seem counterinuitive becosue we dont define the get and set methodes but the compiler genrates them after wards so jakso works for records too*

# Myreal work

so here i created a new record class `HelthResponse(String status)` ( this will creat an object with the filed status, and the getter for this the field will be populated upon creation)
and anothere class that is the restController `HealthController` wiht the methode `health` that will actaul return our record object pupulated with "ok"

```java
package com.adnakiwoch.platform.streaming_api.config.web.controller;

public record HealthResponse(String Status) {
}
```
and the RestContorller class

```java
package com.adnakiwoch.platform.streaming_api.config.web.controller;


import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("api/v1")
public class HelthController {
    @GetMapping("/health")
    public HealthResponse health(){
        return new HealthResponse("ok");
    }

}
```
now as you can see the Srpring-boot + jakson combo will return what we needd but there is a reason why we are using "api/v1/health".

and whey this is imporant.
so though out our project we will have many instances where we will be creating many end points so "api" just mean we are talkign to a resfull entitiy rahtr than somthign that provides the web view for the clinet.
but the most imporant one is "v1" what htis means is ( this is not like a law but a common good practice) that this class is my version 1 restcontorller, and mean i wont change thigns inside here after check. if there is a new
feture that i woudl like to add or work on i woudl creat anotheere "apit/v2" end points ( rest contorller class) and that class will handel it. thi smakes sure if there is a system that si build on our first version of ende
point nothings breaks when we change it.



# Writing a test for our end point

mow to test our end point we can `./mvnw spring-boot:run` and then try to send get requests and it work but for such simple api endpoint starting up the entire aplication is slow and as our project grows in size it woudl
be impracticall so instead we designe a *unit test* that targets only our end point.

so the first thign we need to know is the `@WebMVCTest(target)` which `(Spring MVC Slice Test Annotation)` this anotation is what makes our test to have a scope of a unit. it takes oue target class( it takes the our class object as anargument). now what this means
is when a test inside this class runs it only loads the essential beans for the test, but igrnores all othere beans unless we manually use the `MockBean`( which is not in the socope of what i am leaning). this means bootspring
don have to load all the beans.

the othere thing that i shoudl know is `MockMVC` this is used for *Mock Web Client for HTTP Request Simulation* you could potnionally use it for all the REST requests.but we will use it for our getRequist;
Note the requist should start with `/` 
now this class has the `.perfome` object where we will be doing the actuall requist. this will then do all the thigns that spring boot whould have done for the nomral requist ( like delgeting control to jakson) and it would
recive the jason object. wchi would it then wrap in the ActionObject( or what ever the real name is). now one this object you can preform the `.andExcpect` and perfromrs asseration inside a test to catch any exception
the most used assertion are `(status.isOk())` and `(jsonpath.(.$"my key").value("value you expect")`.
and since `andExpect` return the `ActionObject` you can do as many `andExpect()` opation one after the othere here is the test

```java
package com.adnakiwoch.platform.streaming_api.web.controller;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;

@WebMvcTest(HealthController.class)
public class HealthControllerTest {
    @Autowired
    MockMvc mockMvc;

    @Test
    void helthCheck()throws Exception{
        mockMvc.perform(get("/api/v1/health"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.status").value("ok"));
    }
}
```
# the final test

this is jjust a safe tatt wethere our spring-web starter is behavin in a way that we expect it to or not.
so when a cline send a requist to a servr the HTTP requist has a field called `Header-Value` now this headder value includes many meta data about the requist but one of those that is the interset here is calleed `accept` this is a way that  the clinet tells the api to send it responec in the fomrmat it needs it ( like xml or json). now since we are setting ou api to only response in jason. the test will do will reveal this.

so by defaulw wehn we included the `sping-boot-starter-web` dependacny we are able to used the `DispaturServelate` ( not imporatn now) but this that will make sure that the clinete expection muchs what we configerd our API to respond. so if they dont much it send  a message of the sutus code 406(isNotAllowed) to the clinet. nwo our test tryed to test this by custmely adding the `accept` filled and mimicing a clinet that is expexting xml response

```java
   @Test
    void healthCheckReturns406ForUnsupportedMedea()throws Exception{
        mockMvc.perform((get("/api/v1/health").accept(MediaType.APPLICATION_XML)))
                .andExpect(status().isNotAcceptable());

```
it is in the same class as the above test so it can accces the `MockMVC` and MockMVc know which class to work on tanks to the argumetn we passed on `@WebMVCTest(healthController.class`
