---
layout: post
title: Observables en Javascript: RXjs Observable, ¿Qué es un Subject?
published: true
categories: javascript rxjs
---

En este cuarto y último post sobre Observables en Javascript vamos a ver el último tema que, según mi opinión, forma parte de los conceptos más básicos para comprender los observables.
Vamos a hablar de Subjects. Si no has leídos los posts anteriores dejo la lista:
 - [Qué son los observables](2021-3-01-Observable-RXJS)
 - [Operadores](2021-3-01-Observable-RXJS2)
 - [MergeMap y SwitchMap](2021-3-01-Observable-RXJS)


## ¿Qué es un Subject?

En pocas palabras, es un objeto que permite crear un observable para controlarlo manualmente, por ejemplo.

``` typescript
// Creamos un subject que emitirá _strings_
let subject = new Subject<string>();
// nos suscribimos
subject.subscribe({
    next: (value) => {
        console.log(value);
    }
});
// invocamos manualmente el método next y emitimos un valor
subject.next('Hola');

```
En el ejemplo de arriba vemos el funcionamiento más básico de Subject, crear un observable que podemos controlar, luego nos suscribimos y el resultado es el esperado, emite un valor.

``` bash
Hola
```

### Crear nuestros propios observables y controlarlos

Entonces, podríamos tranquilamente hacer algo así:

``` typescript
let subject = new Subject<number>();

subject.subscribe({
    next: (value) => console.log(value),
    complete: () => console.log('complete'),
    error: (value) => console.log('Error ' + value)
});

let i = 0;
let counter = 0;
let interval = setInterval(()=>{
    i++;
    if(i === 10){
        counter++;
        i = 0;
        subject.next(counter);
    }
}, 200);

document.querySelector("#stopButton")?.addEventListener("click", ()=>{
    subject.complete();
    clearInterval(interval);
});

document.querySelector("#errorButton")?.addEventListener("click", ()=>{
    subject.error("Mi Error custom");
});

```

Y vemos que el resultado es el esperado, hasta que presionamos _Stop_ y se ejecuta el _complete_

``` bash
1 
2 
3 
4 
complete
```
Si en lugar de _Stop_ forzamos un error el Subject deja de emitir también.

``` bash
1 
2 
3 
4 
5 
6 
7 
Error Mi Error custom
```

### Controlar otros observables

El último ejemplo que es el más interesante es que, ya que los operadores son observables (o funciones que retornan observables) podemos utilizar un _Subject_ para controlar otros Observables, por ejemplo.

``` typescript
let onStop = new Subject<void>();

// creamos un observable del evento click de un botón
// agregamos un operador TakeUntil para que deje de recibir eventos cuando onStop finalice.
const observer = {
    next: (item : any) => console.log(` X:${item.clientX} Y:${item.clientY}`),
    complete: () => console.log("complete"),
    error: () => console.log("error")
};

let onStop = new Subject<boolean>();

fromEvent(document, "click").pipe(takeUntil(onStop)).subscribe(observer);

document.querySelector("#stopButton")?.addEventListener("click", ()=>{
    onStop.next(true);
});

```
En el ejemplo de arriba simplemente controlamos el _takeUntil_ gracias a nuestro Subject, cuando se next es true. La idea de hacerlo así es que un evento proveniente de otro lugar puede detener a un evento diferente. O que si lo hacemos manualmente (por ejemplo el usuario hace click) podemos detener un flujo de ejemplos o varios, recordemos que un Subject al igual que cualquier Observable controla múltiples suscripciones.

Nada más por hoy, nos leemos.

