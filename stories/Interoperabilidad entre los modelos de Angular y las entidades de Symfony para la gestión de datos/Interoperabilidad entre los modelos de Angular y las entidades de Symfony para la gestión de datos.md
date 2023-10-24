
# Interoperabilidad entre los modelos de Angular y las entidades de Symfony para la gestión de datos

Angular / TypeScript — Symfony / Doctrine ORM

Cuando desarrollamos aplicaciones que implementan operaciones CRUD (Crear, Leer, Actualizar y Borrar), una de las cuestiones más importantes que debemos plantearnos es cómo gestionar y estructurar la información de forma eficaz.

Esto no sólo nos ayudará a mantener la integridad de los datos de nuestra base de datos, sino que será crucial para garantizar que cualquier operación CRUD se ejecute de forma segura y eficiente.

En este artículo veremos cómo optimizar el flujo de datos entre el backend y el frontend utilizando las entidades de Symfony con su ORM Doctrine y los modelos y servicios de Angular.

## Qué es un ORM (Mapeo Relacional de Objetos)❓

Un ORM (*Object Relational Mappig*) es un modelo de programación que vincula las tablas de una base de datos con las entidades de una aplicación.

Estas entidades no son más que clases cuyas propiedades representan las columnas de una tabla, y a su vez, cada instancia de dicha clase representa un registro de esa tabla en la base de datos.

De esta manera, un ORM nos permite interactuar con la base de datos utilizando objetos en lugar de SQL con cualquier lenguaje de programación orientado a objetos que disponga de un ORM como tal.

En definitiva, un ORM proporciona una capa de abstracción que hace que una aplicación sea más flexible, y sin duda una de las mejores características que ofrecen es el hecho de poder interactuar con la base de datos independientemente del motor que se esté usando.

Esto significa que, en términos generales, para el ORM llega a ser transparente el tipo de base de datos que hay detrás. Así, en caso de que necesitemos pasar de MySQL a PostgreSQL, por ejemplo, únicamente tendríamos que cambiar las cadenas de conexión a la base de datos y listo.

## Modelado de datos con entidades de Symfony ➡️

Como hemos explicado en el apartado anterior, las entidades en Symfony son clases PHP que representan tablas y sus propiedades representan columnas.
> En el contexto de Doctrine, las entidades actúan como una interfaz que permite interactuar con la base de datos a través de objetos.

Por ejemplo, supongamos que necesitamos crear la tabla *Person*.

En SQL nativo tendríamos el siguiente código:

    CREATE TABLE Person (
      id INT NOT NULL PRIMARY KEY,
      name VARCHAR(150) NOT NULL,
      email VARCHAR(50)
    );

Esta misma tabla se representaría en Symfony como una entidad de la siguiente manera:

    namespace App\Entity;
    
    use Doctrine\ORM\Mapping as ORM;
    
    #[ORM\Entity(repositoryClass: PersonRepository::class)]
    class Person
    {
      #[ORM\Id]
      #[ORM\GeneratedValue]
      #[ORM\Column]
      private ?int $id = null;
    
      #[ORM\Column(length: 150)]
      private ?string $name = null;
    
      #[ORM\Column(length: 50, nullable: true)]
      private ?string $email = null;
    
      public function getId(): ?int
      {
        return $this->id;
      }
    
      public function getName(): ?string
      {
        return $this->name;
      }
    
      public function setName(string $name): self
      {
        $this->name = $name;
        return $this;
      }
    
      public function getEmail(): ?string
      {
        return $this->email;
      }
    
      public function setEmail(?string $email): self
      {
        $this->email = $email;
        return $this;
      }
    }

En el código anterior podemos ver que además de las propiedades se definen los *getters *y *setters*, que son métodos que permiten obtener y asignar valores a las columnas para un registro específico.

Es importante entender que al usar entidades no sólo estamos creando representaciones de los datos en el código, sino que también estamos definiendo reglas, restricciones y mecanismos que aseguran la integridad de dichos datos y además nos ayudan a prevenir y controlar errores.

De hecho, en las entidades podemos añadir métodos de control adicionales o incluso definir métodos que respondan a eventos de forma similar a los *triggers *de base de datos, por ejemplo.

### ▪ **Crear registros**

Podemos crear un nuevo registro de la siguiente manera:

    use App\Entity\Person;
    use Doctrine\ORM\EntityManagerInterface;
    
    // ...
    
    public function createPerson(EntityManagerInterface $entityManager)
    {
      $newPerson = new Person(); // Creamos una nueva instancia de la entidad Person
      $newPerson->setName("Norman Bates"); // Asignamos el valor de la columna 'name'
      $newPerson->setEmail("norman@email.com"); // Asignamos el valor de la columna 'email'
    
      // En este momento ya dispondríamos del objeto referente al nuevo registro, así que lo único que quedaría sería guardarlo en la base de datos
      $entityManager->persist($newPerson); // Se prepara la instancia para ser almacenada
      $entityManager->flush(); // Doctrine ejecuta el insert en la base de datos y finaliza el proceso
    }

### ▪ **Obtener registros**

Doctrine dispone de varios métodos a través de los repositorios de cada entidad para obtener registros. Por ejemplo:

    // ...
    
    $personRepository = $this->getDoctrine()->getRepository(Person::class);
    $persons = $personRepository->findAll();

En este caso, *$persons* será un array de objetos *Person*, y cada uno representará un registro de la tabla *Person*.

### ▪ **Modificar registros**

A partir de aquí la mecánica siempre será la misma, así que podríamos hacer lo siguiente:

    // ...
    
    $firstArrayItem = $persons[0]; // Obtenemos el primer resultado devuelto por la api y lo guardamos en una variable
    
    $email = $firstArrayItem->getEmail(); // Obtenemos el email y lo guardamos en una variable
    
    $firstArrayItem->setName("Patrick Bateman"); // Modificamos el valor de la columna 'name'

👆 Para que cualquier modificación se refleje en la base de datos será necesario llamar de nuevo a los métodos *persist()* y *flush()*.

## Modelado de datos con modelos de Angular ➡️

En Angular, los modelos son clases de TypeScript que representan la estructura de los datos que se van a manejar en la aplicación, proporcionando una forma consistente de gestionar los datos.

Al igual que las entidades en Symfony, en los modelos de Angular cada propiedad se corresponde con una columna o campo y una instancia de un modelo representa un registro de dicho modelo.

Un ejemplo simple de modelo *Person *en Angular sería el siguiente:

    export class Person {
      id: number;
      name: string;
      email?: string;
    }

Pero, **¿cómo se conectan estos modelos con las entidades de Symfony?**

En nuestro caso, los modelos de Angular representarán **las mismas estructuras de datos que las entidades de Symfony**.

Cuando el frontend reciba datos del backend utilizaremos los modelos para “tipar” dichos datos aprovechando el sistema de tipos de Typescript.

En el siguiente punto veremos cómo mantener tipados e intactos los objetos de base de datos que recibimos y enviamos a la api.

## **Servicio para operaciones CRUD en Angular ⚡**

Aunque lo ideal es disponer de servicios y modelos dinámicos que puedan ser extendidos (mediante herencias, por ejemplo) por los servicios y modelos propios de cada entidad evitando la repetición de código (*DRY — Don’t Repeat Yourself*), vamos a crear un servicio concreto para la entidad *Person *que usará únicamente el modelo *Person *para que se entienda mejor.

Primero, vamos a modificar el modelo que vimos anteriormente:

    export class Person {
      public id: string;
      public name: string;
      public email?: string;
    
      constructor(person?: Partial<Person>) {
        if (person) { Object.assign(this, person); }
      }
    }

Definimos un constructor que permita instanciar el modelo a partir de un objeto del mismo. Es decir, cuando el backend devuelva un registro de la tabla *Person *de la base de datos podremos hacer lo siguiente:

    const person = new Person({
      id: 1
      name: "Paul Allen"
      email: "paul@email.com"
    })

De esta forma conseguiríamos uno de los objetivos de este artículo: tipar un objeto de la base de datos al igual que en Symfony.

Ahora vamos a crear el servicio *PersonService *con los siguientes métodos:

### ▶ **Crear un registro**

El método *create()* recibe un objeto de tipo *Person *y devuelve el objeto creado en el servidor convertido en una instancia de la clase del modelo.

### ▶ **Obtener todos los registros**

El método *getAll()* método devuelve una lista de todos los registros existentes en la tabla *Person *de la base de datos, la cual es iterada para convertir cada registro en una instancia de la clase del modelo.

### ▶ **Obtener un registro**

El método *getOne()* recibe el ID del registro que queremos consultar en la base de datos y lo devuelve instanciado en el modelo *Person*.

### ▶ **Actualizar un registro**

El método *update()* recibe un objeto de tipo *Person* y devuelve una instancia del registro actualizado en el servidor.

### ▶ **Eliminar un registro**

El método *delete()* recibe el ID del registro que queremos eliminar por completo y no devuelve nada.

1️⃣ Los métodos devuelven instancias de la clase del modelo *Person *para garantizar que los datos que recibimos se ajusten a la estructura que esperamos y además poder utilizar todos las funcionalidades y métodos adicionales que haya definidos en ese modelo.

2️⃣ Todos los métodos devuelven *Observables *para que los componentes que se suscriban a ellos sean los encargados de realizar las tareas oportunas según la necesidad, como controlar errores o llamar a otros métodos.

3️⃣ Los métodos que reciben un objeto de tipo *Person *permiten hacerlo parcialmente (*Partial<Person>*), ya que es posible que cuando se crea un registro, por ejemplo, algunas propiedades aún no estén disponibles.

El código del servicio quedaría de la siguiente manera:

    import { HttpClient } from '@angular/common/http';
    
    // RxJs
    import { Observable } from 'rxjs';
    import { map } from 'rxjs/operators';
    
    import { Person } from './person.model';
    
    export class PersonService {
      private apiUrl: string = 'https://api-url';
    
      constructor(private httpClient: HttpClient) {}
    
      toJson(): any {
        return JSON.parse(JSON.stringify(this));
      }
    
      create(person: Partial<Person>): Observable<Person> {
        return this.httpClient
          .post<Person>(this.apiUrl, person.toJson())
          .pipe(map((result) => new Person(result)));
      }
    
      getAll(): Observable<Person[]> {
        return this.httpClient
          .get<Person[]>(this.apiUrl)
          .pipe(map((result) => result.map((i) => new Person(i))));
      }
    
      getOne(id: number): Observable<Person> {
        return this.httpClient
          .get<Person>(`${this.apiUrl}/${id}`)
          .pipe(map((result) => new Person(result)));
      }
    
      update(person: Partial<Person>): Observable<Person> {
        return this.httpClient
          .put<Person>(`${this.apiUrl}`, person.toJson())
          .pipe(map((result) => new Person(result)));
      }
    
      delete(id: number): Observable<void> {
        return this.httpClient.delete<void>(`${this.apiUrl}/${id}`);
      }
    }

Lo verdaderamente interesante es el hecho de poder trabajar con clases siguiendo prácticamente la misma mecánica en el frontend y en el backend trabajando de una forma más intuitiva y organizada, lo que indirectamente nos lleva a crear **código más limpio y mantenible**.

## **Conclusión ✅**

Hasta aquí el artículo! Espero haberme explicado bien y que te haya sido de utilidad.

La intención es mejorar el código de tus aplicaciones haciéndolo más genérico y reutilizable 🎯.

No dudes en ponerte en contacto conmigo si tienes alguna pregunta.

Puedes seguirme en [LinkedIn](https://www.linkedin.com/in/josegarciarodriguez/) y [GitHub](https://github.com/m4n50n).

Muchas gracias por llegar hasta aquí.
