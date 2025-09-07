so her is the thign lets say we created those configration recordes adn the ymal. but the thing is most of the configration info of an api should not be share on git hub like direct api key shoudl be left an populate if the
applicaiton.yaml is shared.

so in the team of many devvs one coudl potnetionally forrget to make sure to pupulate the api-key field of there yaml with an actaull value. leadin to a null pointer execption nightmare if any object that Aoutowired oout
configration class tyr to acces it and do its own logic on it.

this is just on of the many readon why we will use validation depednacyes in our project. it is specifcally know as( in our case) spring-boot-starter-validation. what it does it it will validate if any our programes violet
the rules we stated for it to validate.

this is the build 

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
    <version>4.0.0-M2</version>
</dependency>
```
you put that in to the dependacny feild in you  pmo.xml where maven will handel teh depedancy risolve. you shoudl remove the verion part if you are inhring a parent build.

this exmpale how you woudl use it to solce the prblem we talked about

```
@Validated
@ConfigurationProperties("platform.billing")
public record BillingProperties (@NotBlank String apiKey, int trailPeriodDays){
}
```

so how do we test it?

well here is the thign the tests we did with wethere the data in our application.yaml is bing poupulated in to out ConfigrationPropoties anotated class wwas possible wiht the @SPringTest anotatnio becouse
those test have the application context that actually starts and they do the checks. but testing weathere our validation faild or didnot canot be simulated wih the simple @SpringTest becouse the application context will faild
at the start hence the name "spring-boot-starter-validation".

so what we will do instaed is use a powerfull utility called ApplicationConetextRunner. which will we start a whole aplication bassed on our main class with the properties of our own
the follwoign code will show you evey thign you shoudl do and know. i will be adding way to much comment there read the to understadn them.
```java
@SpringBootTest
class StreamingApiApplicationTests {
  //here you ceat the actuall APPlicationContextRunner
  private final ApplicationContextRunner contextRunner = new ApplicationContextRunner();
  //now here we define the test that we are going to do a good naming is the way to go

  @Test
  void apiKeyMustNotBeBlank(){
    //so first we load our calss in to the context runner
    this.contextRunner
            .withUserConfiguration(StreamingApiApplication.class)//this is where we load our application class
            .withPropertyValues("platform.billing.trail-period-days=30")/*this is where we specify the propties our memmeroy only exisiting aplicationproperies file
            and as you can see we are creting a propties for the trialPeriodDays field which weillbe populated with it but
            on perpouse leaving the api-key proprtie empety so it would lead to the failing of the aplication context when run*/
            .run(context -> { /*when you run this conntext runner runs it return an object we could name that what evver we wan
            but it has all the information that we need abotu the start of the app */
              assertThat(context).hasFailed();// if the app fiald then this wontn though an error and we know ommitin of the api-key coused this
              assertThat(context).getFailure().hasRootCauseInstanceOf(); #ConfigurationPropertiesBindException.class this is thown when ever there is an exption when spring can change a value in to for instanse a numebr and so on
#the BindValidationException however happend when the binding happens but when there is a validaton problem.
            });
  }
```



