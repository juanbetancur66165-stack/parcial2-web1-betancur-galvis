# Parcial 2 · Respuestas de razonamiento

Nombre: Juan Sebastian Betancur Galvis

Son cuatro preguntas de 5 puntos sobre código que ya conoces. Para cada una:

1. Lee el código y escribe primero lo que **crees** que pasa.
2. Compruébalo: pégalo en la consola de DevTools (F12 → Console) con tu página del parcial abierta en Live Server. En esa consola sí se puede usar `await` fuera de una función.
3. Escribe tu respuesta debajo de cada letra: (a), (b), (c)…

Se califica la explicación, no solo el resultado: con dos o tres frases por letra es suficiente. Escríbela con tus palabras, porque te la pueden preguntar.

## P1 · ¿En qué orden se imprime? (5 puntos)

**Lo viste en:** la función `cargar` del Explorador (semana 8): «await pausa ESTA función, no el navegador».

```js
console.log('1');
setTimeout(() => console.log('2'), 0);
const cargar = async () => {
  console.log('3');
  const respuesta = await fetch('https://rickandmortyapi.com/api/episode/1');
  console.log('4');
};
cargar();
console.log('5');
```

**(a)** Los cinco números en el orden en que salen en la consola. (2 pts)

Respuesta: en el orden que sale en la conosla es 1,3,5,2,4

**(b)** Por qué el `3` sale **antes** que el `5`, si `cargar` es una función `async`. (2 pts)

Respuesta: Aunque cargar() es async, su código comienza a ejecutarse inmediatamente cuando se llama. Por eso imprime 3 antes de que el programa principal llegue al console.log('5').

**(c)** Por qué el `5` no espera a que termine el `fetch`, y por qué el `2` y el `4` salen al final. (1 pt)

Respuesta:await pausa la ejecución de la función asíncrona, no la del programa completo. El temporizador puede ejecutarse mientras se espera la respuesta de la red; el 4 se imprime cuando termina la espera del fetch.

## P2 · Un 404 que no llega al catch (5 puntos)

**Lo viste en:** el `pedirJSON` del Explorador (semana 8): «fetch solo falla si NO pudo hablar con el servidor».

El episodio 9999 no existe: la API responde con código **404** y el cuerpo `{"error":"Episode not found"}`.

```js
try {
  const respuesta = await fetch('https://rickandmortyapi.com/api/episode/9999');
  const datos = await respuesta.json();
  console.log('Episodio:', datos.name);
} catch (error) {
  console.log('Falló:', error.message);
}
```

**(a)** Qué imprime la consola, exactamente. (1 pt)

Respuesta: undefined

**(b)** Por qué **no** entra al `catch`, aunque el episodio no existe. Pista: piensa en la diferencia entre «el servidor respondió con un 404» y «no hubo respuesta» (por ejemplo, sin internet). (2 pts)

Respuesta:fetch() no lanza automáticamente un error cuando el servidor responde con HTTP 404. El servidor sí contestó, pero se indica que el episodio no existe.

**(c)** La línea que falta para que sí entre al `catch`: escríbela e indica en qué parte del código va. Luego escribe qué imprimiría la consola con esa línea. (2 pts)

Respuesta:La línea que falta es if (!respuesta.ok) throw new Error(\HTTP $`);y debe ir después de la línea del fetch y antes de pasar la respuesta a JSON. Esto permite detectar cuando el servidor responde con un error HTTP y enviar el error al catch, la consola imprimiría **Falló: HTTP 404`**, porque el episodio 9999 no existe

## P3 · LocalStorage sin JSON (5 puntos)

**Lo viste en:** `js/favoritos.js` del Explorador (semana 8): «localStorage solo guarda TEXTO».

```js
const vistos = [1, 5, 8];
localStorage.setItem('vistos', vistos);
const leido = localStorage.getItem('vistos');
console.log(leido, typeof leido);      // (a)
console.log(leido.length);             // (b)
const recuperado = JSON.parse(leido);  // (c)
```

**(a)** Qué imprime la línea marcada (a): el valor y su tipo. (1 pt)

Respuesta: La consola imprime 1,5,8 string, porque localStorage guarda los datos como texto. Al guardar el array directamente, JavaScript lo convierte en la cadena "1,5,8".

**(b)** Qué imprime la línea marcada (b), y por qué da ese número. (1 pt)

Respuesta: Imprime 5, porque la cadena 1,5,8 tiene cinco caracteres, incluyendo las dos comas.

**(c)** Qué pasa en la línea marcada (c): ¿funciona o sale un error? Si sale un error, cuál y por qué. (1 pt)

Respuesta: Se produce un error de tipo SyntaxError, porque "1,5,8" no es un JSON válido. La función JSON.parse() no puede convertir esa cadena en un array

**(d)** La segunda línea corregida, para que `recuperado` vuelva a ser el array `[1, 5, 8]`. (1 pt)

Respuesta: localStorage.setItem('vistos', JSON.stringify(vistos)); 
sí, el array se guarda como JSON y luego JSON.parse(leido) puede recuperarlo como [1, 5, 8]

**(e)** Por qué en tu `leerVistos` el `JSON.parse` va dentro de un `try/catch`: qué podría pasarle a la app si no estuviera. (1 pt)

Respuesta: se utiliza para evitar que la aplicación se interrumpa si los datos guardados están dañados o no tienen un formato JSON válido. En ese caso, el catch permite devolver un array vacío ([]) y continuar utilizando la pagina web

## P4 · Botones que dejan de funcionar (5 puntos)

**Lo viste en:** la delegación en la grilla del Explorador (semana 7, paso 04). Está explicada en el README, en R6: «¿Qué es la delegación?».

Un compañero escribió este código. Al cargar la página, el primer clic en «Marcar como visto» funciona; pero después de ese clic, o después de escribir una letra en el buscador, **ningún botón responde**.

```js
render();
document.querySelectorAll('[data-id]').forEach((boton) => {
  boton.addEventListener('click', () => {
    vistos = alternarId(vistos, Number(boton.dataset.id));
    guardarVistos(vistos);
    render();
  });
});
```

**(a)** Qué les pasó a los botones. Pista: fíjate en qué hace `render()` con el contenido de `#lista` cada vez que se llama, y en cuántas veces se ejecuta el `forEach`. (2 pts)

Respuesta: Los botones dejan de funcionar porque render() reemplaza el contenido de #lista y crea botones nuevos. Como los eventos se asignaron solamente a los botones que existían al principio, los nuevos quedan sin listener y dejan de responder

**(b)** El código reescrito con **delegación**: un solo listener en `#lista` que funcione con cualquier botón, aunque `render()` lo haya creado después. Se revisan tres cosas, 1 punto cada una: que el listener esté en `#lista` y no en los botones; que use `closest('[data-id]')` (o algo equivalente) para saber qué botón se presionó; y que no haga nada si el clic no fue en un botón. (3 pts)

Respuesta: lista.addEventListener('click', (evento) => {
  const boton = evento.target.closest('[data-id]');

  if (!boton) {
    return;
  }

  vistos = alternarId(vistos, Number(boton.dataset.id));
  guardarVistos(vistos);
  render();
});

Este código coloca un solo listener en #lista y utiliza closest('[data-id]') para identificar el botón presionado. Si el clic no corresponde a un botón con ese atributo, no hace nada; de lo contrario, actualiza los episodios vistos, guarda los cambios y vuelve a mostrar la lista. 
