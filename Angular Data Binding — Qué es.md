
# Angular Data Binding — ¿Qué es?

Vincular o enlazar datos es una parte fundamental en el desarrollo de aplicaciones web con Angular, ya que nos proporciona un mecanismo muy poderoso de intercambio de información entre la clase del componente y la vista HTML que define el DOM.

Esta característica permitirá construir aplicaciones dinámicas e interactivas manteniendo los datos sincronizados automáticamente en *una *o *dos *direcciones.

### **Enlace unidireccional (One-way binding)**

En enlace de datos en una sola dirección conforma la idea de vincular datos únicamente desde la clase del componente a la vista o viceversa. Esto nos será muy útil cuando necesitemos mostrar información estática por pantalla, por ejemplo.

**Enlazando desde la clase a la vista**

* Interpolación: se insertará dinámicamente el valor de una propiedad o resultado de una expresión en un lugar específico del HTML cuando este se renderice. Para ello incluiremos el nombre de una propiedad o expresión dentro de llaves dobles ‘{{ }}’:

    import { Component } from '@angular/core';
    
    @Component({
      selector: 'app-root',
      template: `
        <h1>Hola, {{ nombre }}</h1>
        <h2>{{ 2 + 2}}</h2>
        <h3>{{ getTexto() }}</h3>
      `
    })
    export class AppComponent {
      nombre: string = 'Jose';
    
      getTexto(): string {
        return 'Texto de ejemplo';
      }
    }

Obviamente cada propiedad o expresión interpolada **debe existir** en la clase del componente.

* Propiedad / atributo / clase / estilo: usando la misma lógica que en la interpolación, asignaremos dinámicamente valores a propiedades y atributos de elementos HTML pero usando corchetes ‘[ ]’ en lugar de llaves dobles. 👆 En Angular, los corchetes siempre indicarán una vinculación **desde la clase a la vista**:

    import { Component } from '@angular/core';
    
    @Component({
      selector: 'app-root',
      template: `
        <div [class]="clase">
          <h1 [innerHTML]="titulo"></h1>
          <input [value]="valor" [placeholder]="placeholder" [style.color]="color">
        </div>
      `
    })
    export class AppComponent {
      clase: string = 'd-flex';
      titulo: string = 'Título de ejemplo';
      valor: string = 'Valor de ejemplo';
      placeholder: string = 'Escribe un valor';
      color: string = 'blue';
    }

**Enlazando desde la vista a la clase**

Para vincular datos desde la vista a la clase usaremos el enlace de eventos de Angular, el cual permite escuchar y responder a acciones del usuario. La sintaxis consta de un nombre de evento entre paréntesis y una instrucción. Por ejemplo:

    <button (click)="onClick()">Haz click aquí!</button>

Cuando el usuario haga click sobre el botón se llamará al método *onClick()* de la clase del componente, que a su vez ejecutará la instrucción o conjunto de instrucciones que definamos (sí, de forma similar a cuando agregamos *listeners *de eventos en vanilla JavaScript). 👆 En Angular, los paréntesis siempre indicarán una vinculación **desde la vista a la clase**.

También es cierto que *data binding* implica la vinculación de datos, y cuando llamamos a un método de la clase desde la vista (en este enlace unidireccional) en realidad no estamos vinculando datos en el sentido estricto, sino que más bien establecemos una “conexión” entre la vista y clase, dando un significado más amplio al concepto de *data binding*.

### Enlace bidireccional (Two-way binding)

El enlace de datos en dos direcciones permite que los datos fluyan en ambos sentidos: de la clase a la vista y de la vista a la clase.

El escenario más típico para este caso se presenta cuando usamos la directiva ***NgModel*** a la hora de crear formularios en Angular mediante *@angular/forms*, donde se espera que un usuario introduzca datos.

    <input [(ngModel)]="nombre" placeholder="Escribe tu nombre">

En el código anterior se establece la vinculación bidireccional entre la propiedad *nombre *en la clase y el *valor del input*: cuando el usuario escriba un valor en el input, el valor de la propiedad se actualizará con dicho valor automáticamente. Del mismo modo, si se actualiza la propiedad en la clase, el valor del input también lo hará.

En este caso cobra más sentido el concepto *data binding* enlazando eventos, ya que aquí sí que existe una vinculación estricta de datos y no solo llamadas a métodos.

### **Conclusión**

Acabamos de ver dos maneras esenciales y muy útiles de conseguir que la información fluya entre clase y vista de un componente.

Además, debemos tener en cuenta que Angular tiene la capacidad de detectar y prevenir, por ejemplo, bucles infinitos de actualización de datos, por lo que consigue que este proceso se realice de una forma totalmente eficiente.

Espero que esta guía te haya sido de utilidad.

Muchas gracias por llegar hasta aquí.
