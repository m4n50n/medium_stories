
# Angular Data Binding — What is it?

Binding data is a fundamental part of web application development with Angular because it provides a powerful mechanism for exchanging information between the component class and the HTML view that defines the DOM.

This feature allows for building dynamic and interactive applications while automatically keeping the data synchronized in *one *or *two *directions.

### One-way binding

One-way data binding entails the concept of linking data solely from the component class to the view or vice versa. This is particularly useful when we need to display static information on the screen, for example.

**Binding from the class to the view**

* Interpolation: the value of a property or the result of an expression will be dynamically inserted into a specific place in the HTML when it is rendered. To do this, we include the name of a property or expression within double curly braces ‘{{ }}’:

    import { Component } from '@angular/core';
    
    @Component({
      selector: 'app-root',
      template: `
        <h1>Hi, {{ name }}</h1>
        <h2>{{ 2 + 2}}</h2>
        <h3>{{ getText() }}</h3>
      `
    })
    export class AppComponent {
      name: string = 'Jose';
    
      getText(): string {
        return 'Main text';
      }
    }

Obviously, each interpolated property or expression **must exist** in the component class.

Property / attribute / class / style: using the same logic as interpolation, we dynamically assign values to properties and attributes of HTML elements, but using square brackets ‘[]’ instead of double curly braces. 👆 In Angular, square brackets always indicate binding **from the class to the view**:

    import { Component } from '@angular/core';
    
    @Component({
      selector: 'app-root',
      template: `
        <div [class]="customClass">
          <h1 [innerHTML]="title"></h1>
          <input [value]="mainValue" [placeholder]="placeholder" [style.color]="color">
        </div>
      `
    })
    export class AppComponent {
      customClass: string = 'd-flex';
      title: string = 'Main title';
      mainValue: string = 'Main value';
      placeholder: string = 'Write me';
      color: string = 'blue';
    }

**Binding from the view to the class**

To bind data from the view to the class, we use Angular’s event binding, which allows us to listen for and respond to user actions. The syntax consists of an event name within parentheses and a statement. For example:

    <button (click)="onClick()">Click here!</button>

When the user clicks the button, the *onClick()* method of the component class will be called, which in turn executes the statement or set of statements we define (similar to adding event listeners in vanilla JavaScript). 👆 In Angular, parentheses always indicate binding **from the view to the class**.

It is also true that *data binding* involves linking data, and when we call a method from the view to the class (in this one-way binding), we are not strictly binding data in the traditional sense. Instead, we are establishing a “connection” between the view and the class, giving a broader meaning to the concept of *data binding*.

**Two-way binding**

Two-way data binding allows data to flow in both directions: from the class to the view and from the view to the class.

The most typical scenario for this case occurs when we use the ***NgModel ***directive when creating forms in Angular using *@angular/forms*. This is where we expect a user to input data.

    <input [(ngModel)]="name" placeholder="Write your name">

In the above code, two-way binding is established between the *name *property in the class and *the value of the input*. When the user enters a value in the input, the property value will automatically update with that value. Similarly, if the property is updated in the class, the input value will also update.

In this case, the concept of *data binding* through event binding makes more sense, as there is a strict binding of data and not just method calls.

### **Conclusion**

We have just seen two essential and very useful ways to make information flow between the class and view of a component.

Additionally, it’s important to note that Angular has the ability to detect and prevent, for example, infinite loops of data updates, making this process highly efficient.

I hope this guide has been helpful to you.

Thank you very much for reaching this point.
