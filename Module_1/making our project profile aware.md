so when we produce a working pojectc there are three devolopmen stages and profiles
* dev- this is the devlopemnent profile
* stage- this is teh staging devlopmetn profile
* prod- this is teh deploymenr profile

now look you might have many testing configraitons in your application.ymal but you dont want many of those application.ymal proporties to be the one who will pupulate your recored class in the production stage
or staging stage

now to make the profile away configration files what we use is applicatoin-{profile).yaml

now we are on the dev step of our project so what we will do is we creat a new application-dev.yaml in our resourse dir then. we go to our first ymal and introduce the following under the spring

``` yaml
  profiles:
    active: dev
```
now what will happend is when spring scans the main appliacton.yaml file it will look at this andn it will imdiealy stop treating th eapplicanto.yaml and instead go to our dev yaml. now the importan thing is it doesn mea
the applicaton.yaml will be unused but it means if there are two of the same propties in both of those proporties.yaml then what will happen is the values in the dev;yaml will be ovveride the once in the first proprites yaml

