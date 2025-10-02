this is made so that i would have a solidi understanding of the spring proxy and inkection work.

so lets look at the foolwogin code as i explain it.

```java
public interface MessageService {
    void sendTransactionalMessage(String message);
    String getRegularMessage();
    void sendMessageAndLog(String message);
}
```

this is the intefase that we are tying to impliment with this

```
@Service
public class MessageServiceImpl implements MessageService {
    
    @Transactional
    public void sendTransactionalMessage(String message) {
        // Business logic for sending a message within a transaction.
        System.out.println("--- Executing REAL sendTransactionalMessage() ---");
    }

    public String getRegularMessage() {
        // Normal business logic.
        System.out.println("--- Executing REAL getRegularMessage() ---");
        return "Hello";
    }
    
    public void sendMessageAndLog(String message) {
        System.out.println("--- Executing REAL sendMessageAndLog() ---");
        
        // A call from one method to another within the SAME class.
        this.sendTransactionalMessage(message); 
    }
}
```

so this entire class is anotated with @Service so during the start up SPring boot know it should creat a bean for it and it does, it takes this object and put it in to its ```aplication context```.
now after this anotehre spring boot procces somthign that goes by `posBinPorcssing` will search for any AOP anota  tion like `@transaction, @ASYNC` and so on. and for our bin that we just created, yes there is.
now what happend is the sprinng boot tryes to handel how AOPs?


and it does this, by creating poxy that will be the, metaphoricall envolop for our real class and here is what happens.

# Case 1 (JDK Dynamic Proxy)
this is entirely for class that are elgibal eto be bean but also extend some type of inteface, like ours does.
so the firs thing it does is it creats a new helper class, which is a long named with man $ symboles to show it is a helper class but we will call it `$Proxy0`. now this proxy has some imporatn fields.
like the treasaction manger but more imporatantly for as a field named `target` what is stored in this trage field is the original bean that spring created, it will inject it here.
but that is not it this class also impliments the same inteface that our tragt class is implimenting in this case `MessageService`;

so it implments ever thign that this inetfase define, since spring made this proxy in mind with AOP it know which field is AOP realted and not, so in our case `sendTransactionalMessage` methode is the poitn of intersatee.
now as immplmentin class it shoudl impliment all those methode and it does do this, but it diffferse on what it does for the AOP signalled methode and forr thos that are normal plain old java methodes.
for these all java methode like `public void sendMessageAndLog(String message) ` somthing like this will happen inside the proxeys code block

```java
// inside the proxy objects  sendMessageAndLog methode
@Overide
public void sendMessageAndLog(String message){
this.target.sendMessageAndLog(message)
}
```
for those non AOP methode the proxy deleges the reposiblity of excutin them in to the real bean( so pathtrhough happens in this case)

but for the transactionally annonated once the entire handling of the transaction logic falles on the proxy object

 and lets see what happend here

 ```java
 public sendTransactionalMessage(String message) {
        // --- TRANSACTIONAL ADVICE (BEFORE) ---
        TransactionStatus status = this.transactionManager.getTransaction(...);

        try {
            // --- DELEGATION ---
            // It calls the method on the REAL target object.
            this.target.sendTransactionalMessag(message); 

        } catch (RuntimeException | Error ex) {
            // --- TRANSACTIONAL ADVICE (ON EXCEPTION) ---
            this.transactionManager.rollback(status);
            throw ex;
        }

        // --- TRANSACTIONAL ADVICE (AFTER) ---
        this.transactionManager.commit(status);
    }
```
now as you can see, there is still a call to the orginall methode, but spring btoot make sure there is a fine grained transaction contorle with this proxy.
understading this very idea of passthrough will help me tackle many real world erros.

now the intefase bassed procxy creation has a litlitle bite differn way on it is handel in injection,

so since you cant inject the orginal bean what you shoudl do is injecct the intefase extendin proxy in to you code where you wanted it to do so, so when you want to use the  MessageServiceImpl you shoudlnt not aoutwires
it just like that you shodl instead aoutwire the `MessageService`, wiht constractor Injection or any way of you choosing.

```java
@RestController
public class MessageController {
    private final MessageService messageService; // This field holds the PROXY ($Proxy60)

    public MessageController(MessageService messageService) {
        this.messageService = messageService;
    }
    // ... controller methods ...
}
```
this is the rigth way to use it this one woudl through an execption couse you can inject bin that has proxy that is not a sub class of it  over it just like this one
```java
//worong one
@RestController
public class MessageController {
    private final MessageServiceImpl messageService; // This field holds the PROXY ($Proxy60)

    public MessageController(MessageServiceIMpl messageService) {
        this.messageService = messageService;
    }
    // ... controller methods ...
}
```
the proxy is the one that is being injected and and it should be inject in to a propoer data type with the propor indjection constaction that is themian that you shodul knwo heree

now the othere problme that you can rasi hair is ehre coudl be many @Service anonated clas that implment `MessageService` inteface so injecting this the way we just did would't work becosue spring doenst know;
which proxy to inject so it would through `NoUniqueBeanDefinitionException` we use what is called the @Qualifier to tell spring boot which bean we are looking for in this code base if there where othere
clas that impliment the `MessageService` we woudl aouto wire our class  (MessageServiceImppl this way

```java
@RestController
public class MessageController {
    private final MessageService messageService; // This field holds the PROXY ($Proxy60)

    public MessageController(@Qualifier("messageServiceImpl") MessageService messageService) {
        this.messageService = messageService;
    }
    // ... controller methods ...
}
```
this now works, so know this is important we Aoutwire the proxy usign the its inteface and solve this noUniuebinExvetpion using Qualifier that we just sow.

now what are the things that i should be carfull about? 
1. trying to call, a methode that is not inside an AOP anotation, but makign that methode in that obejct call one that is, would result in an disired outcome. this is becoue just after the first call,
the exction is delegated to the source bean that doesnt have any idea of AOP actiones. so if you call a methode in a service calss that is not trasactional but that methode calles a trasaction methode, the resutl is
no trasaction oppration happens.
2. if you for some reason created addtional methode that are not in the inteface you should reat thme like helper class, this is becouse. ouor proxy wont have a methode for those and trying to call, on this proxy
those methode will crash, but what you can do is you can make those interface defined and our class implimented methode to call and use those helper new class thsi way onse the call is deligated to the real bean they can
use this to call on them

# Case 2: the CGLIB" (Code Generation Library) way.

now this one is easier, and invloeve the library CGLIB. here we assuem the service class doesnt impliment any type of intepase. so now a bean for this class will be meade too, and then spirng will discovre it has
AOP anotation so what it does is it creates a new proxy that extendes the orginall class and has the ban object in its traget field. so now when you aoutwrire and use this class you are aoutwriign this proxy even thou
unliek the intraface bassed proxy you are nto using the differn name here you just aoutwire it. the only thing you shoudl know here is that you can define any of those methode to be Fianl, as it can ovvride those methodes
.

but the rest of the Excution Delegtion algorizm is the same with the inteface bassed one. 

and not callig @Trasactino annotated methode from ouside on both casses woul lead to non Trascation exutio..

now what about we have two @Trasactionnly anotated methode and one calles the othere, what does happen?

the magic is that they will be treated as the same trasaction this is becouse, after the first methode call in teh proxy, that methode will call this.traget.methode; this call is insdi ethe trasaction try and catch.
but after the traget.methode, that methodde could call the othere transaction anotnated methode and that woudl be in the context of the real bean meanin it wong happend inside trasaction meanign, the intire call is stacked
in on trasactionall call.














