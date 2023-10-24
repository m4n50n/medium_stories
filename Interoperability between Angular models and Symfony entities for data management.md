
# Interoperability between Angular models and Symfony entities for data management

Angular / TypeScript — Symfony / Doctrine ORM

When we develop applications that implement CRUD (Create, Read, Update, and Delete) operations, one of the most important questions we must ask ourselves is how to manage and structure information effectively.

This will help us maintain the integrity of the data in our database and it will be crucial in ensuring that any CRUD operation runs safely and efficiently.

In this article we will see how to optimize the data flow between the backend and the frontend using Symfony entities with Doctrine ORM and Angular models and services.

## What is an ORM (Object Relational Mapping)❓

An ORM ( *Object Relational Mapping* ) is a programming model that links the tables of a database with the entities of an application.

These entities are classes whose properties represent the columns of a table, and each instance of this class represents a record of that table in the database.

In this way, an ORM allows us to interact with the database using objects instead of SQL using any object-oriented programming language that has an ORM as such.

In short, an ORM provides an abstraction layer that makes an application more flexible. One of the best features they offer is the fact of being able to interact with the database regardless of the engine being used.

This means that, in general terms, the type of database behind it becomes transparent to the ORM. In case we need to go from MySQL to PostgreSQL, for example, we would only have to change the connection strings to the database and that’s it.

## Data modeling with Symfony entities ➡️

As we have explained in the previous section, entities in Symfony are PHP classes that represent tables and their properties represent columns.
> In the context of Doctrine, entities act as an interface that allows you to interact with the database through objects.

For example, suppose we need to create the *Person* table.

In native SQL we would have the following code:

    CREATE  TABLE Person ( 
      id INT  NOT  NULL  PRIMARY KEY, 
      name VARCHAR ( 150 ) NOT  NULL , 
      email VARCHAR ( 50 ) 
    );

This same table would be represented in Symfony as an entity as follows:

    namespace  App \ Entity ; 
    
    use  Doctrine \ ORM \ Mapping  as  ORM ; 
    
    #[ORM\Entity(repositoryClass: PersonRepository::class)] 
    class  Person
     { 
      #[ORM\Id] 
      #[ORM\GeneratedValue] 
      #[ORM\Column] 
      private ? int  $id = null ; 
    
      #[ORM\Column(length: 150)] 
      private ? string  $name = null ; 
    
      #[ORM\Column(length: 50, nullable: true)] 
      private ? string  $email =null ; 
    
      public  function  getId ( ): ? int
       { 
        return  $this- >id; 
      } 
    
      public  function  getName ( ): ? string
       { 
        return  $this- >name; 
      } 
    
      public  function  setName ( string  $name ): self
       { 
        $this ->name = $name ; 
        return  $this ; 
      } 
    
      public  function  getEmail ( ): ? string
       { 
        return $this- >email; 
      } 
    
      public  function  setEmail ( ? string  $email ): self
       { 
        $this ->email = $email ; 
        return  $this ; 
      } 
    }

In the previous code we can see that in addition to the properties, *getters* and *setters* are defined , which are methods that allow obtaining and assigning values ​​to the columns for a specific record.

It is important to understand that when using entities we are creating representations of the data in code and we are also defining rules, constraints and mechanisms that ensure the integrity of said data and also help us prevent and control errors.

In fact, in entities we can add additional control methods or even define methods that respond to events in a similar way to database *triggers , for example.*

### ▪ Create records

We can create a new record as follows:

    use  App \ Entity \ Person ; 
    use  Doctrine \ ORM \ EntityManagerInterface ; 
    
    // ... 
    
    public  function  createPerson ( EntityManagerInterface $entityManager )
     { 
      $newPerson = new  Person (); // Create a new instance of the Person entity 
      $newPerson -> setName ( "Norman Bates" ); // Assign the value of the 'name' column 
      $newPerson -> setEmail ( "norman@email.com"); // Assign the value of the 'email' column 
    
      // At this point we would already have the object referring to the new record, so the only thing left would be to save it in the database 
      $entityManager -> persist ( $newPerson ); // Prepare the instance to be stored 
      $entityManager -> flush (); // Doctrine executes the insert in the database and ends the process
     }

### ▪ Get records

Doctrine has several methods through the repositories of each entity to obtain records. For example:

    / ... 
    
    $personRepository = $this -> getDoctrine ()-> getRepository ( Person :: class ); 
    $persons = $personRepository -> findAll ();

In this case, *$persons* will be an array of *Person* objects , each representing a record from the *Person* table.

### ▪ Modify records

From here the mechanics will always be the same, so we could do the following:

    // ... 
    
    $firstArrayItem = $persons [ 0 ]; // Get the first result returned by the api and store it in a variable 
    
    $email = $firstArrayItem -> getEmail (); // Get the email and store it in a variable 
    
    $firstArrayItem -> setName ( "Patrick Bateman" ); // Modify the value of the 'name' column

👆 In order for any modification to be reflected in the database, it will be necessary to call the *persist()* and *flush()* methods again.

## Data modeling with Angular models ➡️

In Angular, models are TypeScript classes that represent the structure of the data to be handled in the application, providing a consistent way to manage the data.

Like entities in Symfony, in Angular models each property corresponds to a column or field and an instance of a model represents a record of that model.

A simple example of a *Person* model in Angular would be the following:

    export  class  Person { 
      id : number ; 
      name : string ; 
      email?: string ; 
    }

But **how do these models connect to Symfony entities?**

In our case, Angular models will represent **the same data structures as Symfony entities**.

When the frontend receives data from the backend we will use the models to “type” said data taking advantage of Typescript’s type system.

In the next point we will see how to keep the database objects that we receive and send to the api typed and intact.

## Service for CRUD operations in Angular ⚡

Although the ideal is to have dynamic services and models that can be extended (through inheritance, for example) by the services and models of each entity avoiding code repetition (DRY — Don’t Repeat *Yourself*), we are going to create a service concrete for the *Person *entity that will only use the *Person *model for better understanding.

First, let’s modify the model we saw earlier:

    export  class  Person { 
      public  id : string ; 
      public  name : string ; 
      public email?: string ; 
    
      constructor ( person?: Partial<Person> ) { 
        if (person) { Object . assign ( this , person); } 
      } 
    }

We define a constructor that allows the model to be instantiated from an object of the same. That is, when the backend returns a record from the *Person* table of the database, we can do the following:

    const person = new  Person ({ 
      id : 1 
      name : "Paul Allen" 
      email : "paul@email.com"
     })

In this way we would achieve one of the goals of this article: typing a database object just like in Symfony.

Now we are going to create the *PersonService *with the following methods:

### ▶ Create a record

The *create()* method receives an object of type *Person* and returns the object created on the server converted to an instance of the model class.

### ▶ Get all records

The *getAll()* method returns a list of all existing records in the database’s *Person *table, which is iterated over to make each record an instance of the model class.

### ▶ Get a record

The *getOne()* method receives the ID of the record we want to query in the database and returns it instantiated in the *Person* model.

### ▶ Update a record

*The update()* method receives an object of type *Person* and returns an instance of the updated record on the server.

### ▶ Delete a record

*The delete()* method receives the ID of the record that we want to completely delete and returns nothing.

*1️⃣ The methods return instances of the Person* model class to guarantee that the data we receive conforms to the structure we expect and also to be able to use all the additional functionalities and methods that have been defined in that model.

2️⃣ All the methods return *Observables* so that the components that subscribe to them are in charge of carrying out the appropriate tasks according to the need, such as controlling errors or calling other methods.

3️⃣ The methods that receive an object of type *Person* allow you to do it partially (*Partial<Person>*), since it is possible that when a record is created, for example, some properties are not yet available.

The service code would be as follows:

    import { HttpClient } from  '@angular/common/http' ; 
    
    // RxJs 
    import { Observable } from  'rxjs' ; 
    import { map } from  'rxjs/operators' ; 
    
    import { Person } from  './person.model' ; 
    
    export  class  PersonService { 
      private  apiUrl : string = 'https://api-url' ; 
    
      constructor ( private httpClient: HttpClient ) {} 
    
      toJson(): any { 
        return  JSON . parse ( JSON .stringify ( this ) ); 
      } 
    
      create ( person : Partial < Person >): Observable < Person > { 
        return  this . httpClient
           . post < Person >( this .apiUrl , person.toJson (       ) ) . pipe ( map ( ( result ) =>
     newPerson  ( result))); 
      } 
    
      getAll (): Observable < Person []> { 
        return  this . httpClient
           . get < Person []>( this . apiUrl ) 
          . pipe ( map ( ( result ) => result. map ( ( i ) =>  new  Person (i)))); 
      } 
    
      getOne ( id : number ): Observable < Person> { 
        return  this . httpClient
           . get < Person >( ` ${ this .apiUrl} / ${id} ` ) 
          . pipe ( map ( ( result ) =>  new  Person (result))); 
      } 
    
      update ( person : Partial < Person >): Observable < Person > { 
        return  this . httpClient
           . put < Person>( ` ${ this .apiUrl } ` , person.toJson ()) 
          . pipe ( map ( ( result ) =>  new  Person (result))); 
      } 
    
      delete ( id : number ): Observable < void > { 
        return  this . httpClient . delete < void >( ` ${ this .apiUrl} / ${id} ` ); 
      } 
    }

What is really interesting is the fact of being able to work with classes following practically the same mechanics in the frontend and in the backend, working in a more intuitive and organized way, which indirectly leads us to create cleaner **and more maintainable code**.

## Conclusion ✅

Here the article! I hope I have explained myself well and that it has been useful to you.

The intention is to improve the code of your applications making it more generic and reusable 🎯.

Feel free to contact me if you have any questions.

You can follow me on [LinkedIn](https://www.linkedin.com/in/josegarciarodriguez/) and [GitHub](https://github.com/m4n50n).

Thank you very much for coming here.
