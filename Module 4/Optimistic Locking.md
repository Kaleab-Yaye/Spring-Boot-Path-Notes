so lets imgine a senario on the streaming api we building.

so lets say there is a person who has 50 birr with of subscitpion point that he can use to donate to modes. but this person is not just a nomral user he is tryign to take advantage of our systtem.
he know at exaclty 6:00 30 poitns will be deductued from him to pay to his monthly subscription, but this perosn make a 50 birr payiment to anothere meodel at the exact time.
now what hppens lets look at data base level

USER    SUBPOINTS
A        50

now transaction 1; that takes 50 from this user to a meodle that he is payin ( he intiated this).
* the requist reaches our api
* it gets mapped to the exact requist it was making
* the data base will be asked to give this users info bassed on id
* then calucaltion will be done on it and the last SUBPOINTS will be set to 0
At the same time the monthely Subscrtiop start a prosses
now this prosse was requested at the same time or alitle bit afte the first oen befoe first prosses updates it,
now this one will look at 50 didacted 30 and thignkthe final SUBPOINT to be 20
so both woudl commit and the final monew the persoo i left wiht will 20, when in reallity teh subcsctiopn teh monthly one should have happend.

now to solve this we could potentioally make tha that case to nver let othere prosses tocuh teh user info untill theothere posses is done we call this `pesimistic locking ` we dont need this most of the time as it 
hold a lot of trade we need every thing to happen in the ( first come first served loagic) and we do this as follow

bags like this woudl cost as fortune if someone take adanvantage we dont need that, insteaed what we do is we introdce somthign called @Version anotation to our Enitity like user, that willl be used for optimisic locking
in a way that we will talk here

```java
@Entity
@Table(name = "Platform Users")
public class User {
  @Id private Long id;

  @Version
  private Integer version;

  private String name;

  private String email;

    public String getName() {
        return name;
    }
}
```
so now lets assum this is how user A row is made,
now inadtion to his name and balence this person would have anothere field that is named `version`
this filed sinse it i anotaed will be used as a signe for opstmitic lock.
so now lets go back to out problem at hand.
the intial state of A
USER  BALENCE VERSION
A     50       0

* the two procces final qeury in teh mmeroy before commit woudl look like
  1. Update USER BALENECE = 0 AND VERSION =1 WHERE USER NAME =A AND VERSION =0
     seond proces commit waiting instracttion
  3. Update USER BALENECE = 20 AND VERSION =1 WHERE USER NAME =A AND VERSION = 0

  now lets assume the first isntactnon 1 get completed first and then it get excuted and commited so the new inforamtion in A I

  USER BALENCE VERION
  A    0         1

  now the scone prosses in the memroy doend knwo the changes but onnly opration in the asuumtion that the orginal data is still there.
  so the ` Update USER BALENECE = 20 AND VERSION =1 WHERE USER NAME =A AND VERSION = 0` gets commited but here is the twist teh datap will not find a user with teh name A and version 0, couse the prosses that came
  first has alread update the version to 1, so this will return anExcetption that will propget back to our application and our aplication will handel it. this is the fogiving locking.


  
