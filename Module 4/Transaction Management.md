So sata bases oprate at teh level of integrity that is called ACID(atomicity, Consistancey, ISolation, Durablity).

sor for this topic our main intersit is in teh data bases nature of atomicity. 

so trasaciton is a group of prosses in the data base that are happenign and that the data base makes sure that all or non of the are exuted or persisited.
we will see the grnual detailes ( to teh extend allowed) but, lets just say the spring boot data jpa handels this with teh anotation, @Trasaction and this involes creating a proxy class just to handel this.

so lets look at this class.
```java
@Service
public class UserServiceImpl implements UserService{
    @Transactional
    public void updateUser(User user){
        System.out.println("Updating User data" + user.getName());
    }
}
```
now what thisi trasactionl is sigligng spring boot is that it want to treat all the things that happend here as one and if one fail to rolle back and not do any thing.

so you can see in our helper note for proxies how this class become a proxy that is a bean and how it is injected but the secret beyond this is this,

so it might look intimidateing but it really is not and we don have to knwo that deep the key take away is in this trasactional algorithm

1.there is `trasactional Manger` that has acces to a datasourc's connectin object that is tied with the current trade so any futre data base requsts go though this connetions pile( this insure the methode don
exute else where othere than the curretn data soruce)
2. the algortim involves a try and catch block, the is configures in a way the trasaction manger called COMMITE(on SUcces ) and RolleBack (on failer)
```java
// --- FINAL, MORE DETAILED PSEUDOCODE of the Spring-generated Proxy ---

public class $Proxy60 implements UserService {

    private final UserService target; 
    private final PlatformTransactionManager transactionManager; 

    // Constructor...
    
    @Override
    public void updateUser(Long id) {

        // Define transaction rules (propagation, isolation, etc.)
        TransactionDefinition definition = new DefaultTransactionDefinition();

        // -------------------------------------------------------------------------
        // --- "BEFORE" ADVICE ---
        // -------------------------------------------------------------------------
        
        // 1. START THE TRANSACTION
        // The transactionManager.getTransaction() method is called.
        // Internally, it does the following:
        //    a. Asks the DataSource for a NEW physical connection (let's call it Connection A).
        //    b. Issues "BEGIN TRANSACTION" on Connection A (e.g., connection.setAutoCommit(false)).
        //    c. **THE MAGIC STEP:** It binds Connection A to the current thread.
        //       TransactionSynchronizationManager.bindResource(dataSource, Connection A);
        TransactionStatus status = this.transactionManager.getTransaction(definition);


        try {
            // -------------------------------------------------------------------------
            // --- DELEGATION TO YOUR CODE ---
            // -------------------------------------------------------------------------
            
            // 2. CALL THE REAL METHOD
            // The call is delegated to your original object.
            // When your code calls userRepository.save(), Hibernate will eventually
            // ask the DataSource for a connection. The DataSource will check the
            // TransactionSynchronizationManager, find Connection A, and reuse it.
            this.target.updateUser(id);

        } catch (RuntimeException | Error ex) {
            
            // -------------------------------------------------------------------------
            // --- "ON EXCEPTION" ADVICE (ROLLBACK) ---
            // -------------------------------------------------------------------------
            
            // 4a. ROLLBACK THE TRANSACTION
            // The proxy catches the failure.
            // It calls rollback() on the transactionManager.
            // The transactionManager gets the current transaction status, finds
            // Connection A from the ThreadLocal, and issues "ROLLBACK" on it.
            this.transactionManager.rollback(status);
            
            // It then re-throws the exception so the caller knows what happened.
            throw ex; 

        } finally {
            // -------------------------------------------------------------------------
            // --- "FINALLY" BLOCK (CLEANUP) ---
            // -------------------------------------------------------------------------

            // 5. CLEAN UP THE THREADLOCAL
            // This code runs whether there was a success or a failure.
            // It's crucial to unbind the connection from the thread so it
            // doesn't leak and get accidentally reused by a future request.
            // The transactionManager's commit/rollback logic handles this cleanup.
            // It would call something like:
            // TransactionSynchronizationManager.unbindResource(dataSource);
            // And then return Connection A to the DataSource's pool.
        }

        // -------------------------------------------------------------------------
        // --- "AFTER SUCCESS" ADVICE (COMMIT) ---
        // -------------------------------------------------------------------------
        
        // 3. COMMIT THE TRANSACTION
        // If the try block completed without an exception, this line is executed.
        // The transactionManager gets Connection A from the ThreadLocal and issues "COMMIT".
        this.transactionManager.commit(status);
    }
}
```

# how does RoleBack and Commite actually Work?

so lets say there are two data rows,
NAME    BALANCE
ABEBE    100
ASTER    100

now lets assume Aster wants to trasnfer 10 birr to abbe, now this is nto just a simpel update opration it involves dedactin adn addin value on two differn, but for this context realted rows and the mone should not be 
take from aster and not be added to abbe. money doest have to disaper.

now our basicll we do is I am now is 
ASTER.getBaLence.SetBalence(90);
ABBEBE.getBalence.SetBalence(110);

now those hibernate oprtion will be trasalete and will be exuteted, but the exction is wher the magic lays.
First, the data base created 2 new rows in the data base memrory once with the new vlue, or importantly two instraction are store in teh daa base like

SET BALENCE 110 WHERE USERNAME ABEBE
SET BALENCE 90 WHERE USERNAME ASTER

thos instraction are awayign to be exuted, and the gree flag is always COMMITE, if this COMMITE comes fom the same connection pipe that issued those instraction. then the will be exctued,
if it recives a ROLLABCK it smiply reomves thos instraction form the mmeroy and nothign really happens.

those points are really important with @Version too when we talk about OptimistLoaking.





