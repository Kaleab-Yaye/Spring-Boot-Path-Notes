so this test is differnt intead of testing a code that we written, it will largely test and will prove to use that tout data base can curty optmitic locking. so how should we do it.
well our test shoudl simulate the following things
* it should be the case that the two opration that we need them to be done should at teh same time meanign they shoudl be able to read the same data from the data base two work with and we want to see what will happedn when they
* try to update the orginal data that they have read at the same time.
so our test should include atleast two trades to simulate this, and creating two trades adn makiing a request is nto the end game we shouls also make sure these reqeust aare actually `sencronised` and we will see all that
* the other thign sis we should not be usign a reall data base that we have instead we shoudl let ( or find a way) for our code to creat a mock data base from our containers and then  after finishign the test destroy the data base

for this test wee will be nedding three actors (dependancies)

`org.testcontainers:junit-jupiter`, ` org.springframework.boot:spring-boot-testcontainers`, `org.springframework.boot:spring-boot-testcontainers`
now lets see what each actor gives us or what it is used for.

## org.testcontainers:junit-jupiter
ok so firs the test containers are in the docker and we want to integrate thema nd with them in inside the `junit-jupiter`  provided `@TestAnotations` so to do so this library will be the birge between this Junit world andt
and the container world, it provide the anotations `@Testcontainers` ( which signale the a container must be used for that spring test) and `Containers`( that actually start and manage the reall container)
now the testconainer library will start, the docker container, it will give it a random port number a rundum pass word user id and staff all which is just a mock and temporary, it will more imporatanly overide
any data source configration, but with all that the data source need to actaully have JDBC so that hibernate knows the dilecto fo the actuall data base that we are usignng

## org.testcontainers:postgresql
now this is what a dirver is, this define the deilect that the data base is goign to talk, thsi code be any of thsoe it coudl have been an SQL it doesn matter it is wholley on our choice 

## org.springframework.boot:spring-boot-testcontainers
this integrate or become the bridge between our container and oour application, it does this by readin tthe ovverdign test data source configration and crreating a bena of data source that Hibernate will use that is 
configer for this taset, now reconfigering oud data source configration in the modern sprin test is done by a singel `@ServiceConnection` and the magic will happen

all that is what we need for the test the actaull logic of the adat abse is a litble beat more than that and need farther looking
the other thing we should know about is how we will creat trade pools and how we can syncrrnize them using. and this is done using `ExecutorService` this si a high level of abstractino to trades that helps to creat and
now lets look how it is created 

## ExecutorService
so this is the one that will take over our trade contorl and do a very grnual work with it.
the most common way to cated this object is to call on its helper class's Static Factory Methode such us, `neFixedTradePool(int numOfTrade)`
```java
Executor executor = Executors.newFixedThreadPool(2);
```
now we just create an Executor object aht is goign to mange to trade pools, now hod do you cataully assine a task to the new two ttades that we just created, you use `submit(Runnable runnable)`
this is a methdoe that will submite our exction in the form of lamda and what we need to do with it, we will see how we will use this on the reall test.

## Problaem
now the things is the two trades will do there two jobs but they all must start at the same time( to make sure they have read the same data) and one shoudl end after the othere in the order that we want it. and aslo the
Exceptption that we will be cathd by teh on eof the two trades shoudl propagate to the amin trade. those two problems are what we call `Visiblit` issues.

the reason for thsi we dont know where those treades will be runing on the mulit cpu core decive we have, so the change that one cpu has made may not be visisble to the othere part of the cpu, if they stroe the variable values
on there own catch. so , we have to use a data types tasht are actaully meant for multi trade updates and they will grante that a hcange the happedn in one cpus trade will be visisble to othere cpus trade
and those are 

### AtomicRefference;
so as we said the cpu catch which is there for performance and othere hardware related reasons it is th ebigges enemy and the couse of our viviblity porblem, so we need to use object and data types that include the instraction
to the cpu to actaully flash and read the current stat of our data so we sue Atomic class, the have  get() set() and c`ompareAndSet()`( which is calles (CAS) algortithm behind the java level optismisitc loackign it tell the cput to compare the
data it read with the actually data in mmeory and if they are the same set it with the new value). thuse theose instraction makes sure all the trades that are workgin on are reading and workgin and have acces to the 
curretn data that is update from any trade, this is improant for our test it wont be a `flacky tests`

exmaple ;
```java
AtomicRefferece<String> atomic = new AttomicRefferce<>("string that fights our cpu catch problem")
```

### LatchCountdown

this is anothere lowlevel utltiltiy class that is used to be like a messenager between differnt trades, when ont trade does somthign to thsi the othere trades sees it and bassed on our logic does somthign with it.
and the way it does this is when a COuntdownLatch is created it involves setting the upper limit number the countdown happend for exp it coudl be 5, now differ trades does stomhign and then call .countDown on it 
this can happend till it is 0, now anothere trade that was put to sleep by the os would be woken up by the os when countdown reaches 0 of that COuntdownLatch object. so how will we use it in the test

the executer.submit() will make sure both of our trades read the same data, becouse we will cope it with 2 CountdownLatC that willbe use
* to singla both trades have read the smae data
* to make sure the one that we need to encounter OptmitsicLockException to wait for the first one to complete.

now wtih all that information we can actaully writie our test


