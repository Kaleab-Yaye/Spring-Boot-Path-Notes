ok so the thing you have to knwo is that when our API starts runing there are things that we wont to never change and usually those propties are needed by othere class.
so we will use the two things here.

* the consept of records
* the concept of @ConfigrationPropoties

records are defined this way. they are a speiclly tip of calss taht java handels to refuce boilerplate code and humn erros. adn they are mainly used to creat an **imutable** object. and, thye follow what is called 
**constractor data binding ** meaningi you can assine value to those field once though constractor. there are not setters  but there are getters to those final fields and they are named the same as the feilds.
tehre is not getC..() kind of defintion now here is how you define it.

```java
public record BillingProperties (String apikey, int trailPeriod){
}
```
now as you can see this is the BillinProporties record calss it wil have final fields called api key and trailPeriod. and gettter methode called apiKey() and trialPeriod().

## connecting this to spring
now spring uses anotations to know the use of those spically class now oenof the anotatin that we will care here is called `@ConfigrationProporties`. this anotatin relates our class to out porpties.yml files,
this anotation take prefix as a parameter meanign if yuu uses this perfix in those porpties files then spring can easily relate those to the bean object i will creat and do constractor binding

```java
@ConfigurationProperties("platform.billing")
public record BillingProperties (String apiKey, int trailPeriodDays){
}
```
now in the proporites ffiles you coudl have somthing lookin like his

```proporties
platform.billing.api-key=some value #propoties use the kebab-case
platform.billin.trail-period-days=30
```
the name coudl even be a litle bite differn and spring is samer enoph to populate the right values to the right final field in our record class.

now all  is set but even if there is a configrationPropoties anotaition above our record class it wont do any thing to it anless we add antehre anotation that is specifally desined to tell sprign to scan where to fidn spring
conifgrationProporties(). and this is anoatation is  
`ConfigurationPropertiesScan`
this is what telles the spring that it has to search for configrationPropoties tags and this is define in the APiAplicatoin inside your souce code from where every thing just starts.

```java
@ConfigurationPropertiesScan
@SpringBootApplication
public class StreamingApiApplication {

  public static void main(String[] args) {
    SpringApplication.run(StreamingApiApplication.class, args);
  }
}
```
now how does we realte this to the file where we store our configration values.
so the firs thing we need to do is chnage the apliatoin.propties file inour src resouse dir to application.yaml .
you can remmebr the prefix that @ConfigrationPropties took was "platform.billing"
now and the field in our BillingPropties record where apiKey and trialdPeriodDays

so here is how our yaml will look lik 

```yaml
spring:
  application:
  name: streaming-api

platform:
  billing:
    api-key: "dummy api key"
    trial-period-days: 30
```
the kebeb case should be all in small cases 

now even thoug i dont undertsadn its use i am being told to add the CdnPropoties record with string baseUrl and with the boolean privateStreamingEnabled

## tesiting to see if our configration is working fine 

here is the thing in spring boot in the ...java/test dirst there is java class that spirng inlier provvide for us to be used for testing staff.

now look we will proally see more about injection and how it works but when you start you test run . the whols spring boot starts woking and important beans are created and the jvm will inject those bean though @Autowiret the
class we need it. and here we will aoutwire our main configr recoreds that we created after populating them from our yaml file and it checks stuff.
so here is the entire test we didi

```java
package com.adnakiwoch.platform.streaming_api;

import com.adnakiwoch.platform.streaming_api.config.BillingProperties;
import com.adnakiwoch.platform.streaming_api.config.CdnProperties;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest
class StreamingApiApplicationTests {

  @Autowired
  BillingProperties billingProperties;

  @Autowired
  CdnProperties cdnProperties;


  @Test
  void contextLoads() {}

  @Test
  void billingPropertiesAreINBound(){
    assertThat(billingProperties.apiKey()).isEqualTo("dummy api key");
    assertThat(billingProperties.trailPeriodDays()).isEqualTo(30);
  }

  @Test
  void cdnPropertiesAreInBound(){
    assertThat(cdnProperties.baseUrl()).isEqualTo("https://cdn.streaming-platform.com");
    assertThat(cdnProperties.privateStreamingEnabled()).isTrue();

  }

}
```
now the main fouces shold be on the @Test anotated methode. so when mvn run the test run what i will do is try to catch any erro from those tes anotaated methode and report it. if one of those test fail then the packeging
step fails. that is not the point thoug what i need to now here that if any of those emthode though an exception then the test will fail.
so in thoery we could creat out own exceptions after checks and give us the type of message we want. but usind the assertThat assertion library gives us the most conises gailer messages like this one 

<img width="390" height="177" alt="image" src="https://github.com/user-attachments/assets/43abdf87-834b-4d27-8cd4-760280243d7c" />

so assertTaht(actuall).isequallTo(Expected) syntax is the pro wway to go and more conisese.


