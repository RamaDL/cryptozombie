# cryptozombie
solidity game


El código Solidity está encapsulado en contratos. Un contrato es el bloque de construcción más básico de las aplicaciones de Ethereum — todas las variables y las funciones pertenecen a un contrato, y este será el punto de partida de todos tus proyectos.

Todo el código fuente en Solidity debería empezar con una declaración "version pragma" de la versión del compilador que debe de usarse para ese código. Esto previene problemas con versiones futuras del compilador que podrían no ser compatibles y fallar con tu código.

Esta declaración se asemeja a esto: 
    pragma solidity ^0.4.25;

## Variables
Las Variables de estado se guardan permanentemente en el almacenamiento del contrato. Esto significa que se escriben en la cadena de bloques de Ethereum. Piensa en ellos como en escribir en una base de datos.

contract Example {
  // Esto se guardará permanentemente en la cadena de bloques
  uint myUnsignedInteger = 100;
}


## Estructuras
Podemos crear estructuras de datos más complejas en solidity y esto lo logramos mediante strucs

struct Person {
  uint age;
  string name;
}


## Arrays
Hay dos tipos de arrays en Solidity: arrays fijos y arrays dinámicos:

// Un Array con una longitud fija de 2 elementos:
uint[2] fixedArray;
// otro Array fijo, con longitud de 5 elementos:
string[5] stringArray;
// un Array dinámico, sin longitud fija que puede seguir creciendo:
uint[] dynamicArray;

También puedes crear arrays de estructuras. Si usásemos la estructura Person del capítulo anterior:

Person[] people; // Array dinámico, podemos seguir añadiéndole elementos

¿Recuerdas que las variables de estado quedan guardadas permanentemente en la blockchain? Así que crear un array de estructuras puede ser muy útil para guardar datos estructurados en tu contrato, como una base de datos.

### Arrays públicos
Puedes declarar un array como público, y Solidity creará automaticamente una función getter para acceder a él. La sintaxis es así:

Person[] public people;
Otros contratos entonces podrán leer (pero no escribir) de este array. Es un patrón de uso muy útil para guardar datos públicos en tu contrato.


## Funciones
Una declaración de una función en Solidity se asemeja a esto:

function eatHamburgers(string _name, uint _amount) {
  //cuerpo de la función
}

Nota: la convención (no obligatoria) es llamar los parámetros de las funciones con nombres que empiezan con un subrayado (_) para de esta forma diferenciarlos de variables globales.


## Creación de estructuras
Ahora aprenderemos como crear un nuevo Person y añadirlo a nuestro array people.
// crear un nuevo `Person`
Person satoshi = Person(172, "Satoshi");

// añadir esta persona a nuestro array
people.push(satoshi);


## Funciones públicas y privadas
En Solidity, las funciones son públicas (public) por defecto. Esto significa que cualquiera (o cualquier otro contrato) puede llamarla y ejecutar su código.

Esto no es algo que queramos que pase siempre, y de hecho puede hacer vulnerables tus contratos. Es por lo tanto una buena práctica marcar tus funciones como privadas (private), y solamente hacer públicas aquellas que queramos exponer al mundo exterior.

Vamos a ver como se declara una función privada:

uint[] numbers;

function _addToArray(uint _number) private {
  numbers.push(_number);
}
Esto significa que solo otras funciones dentro de tu contrato podrán llamar a esta función y añadir al array numbers.

Como puedes ver, usamos la palabra clave private después del nombre de la función. Y de las misma forma que con los parámetros de funciones, la convención es nombrar las funciones privadas empezando con un guíon bajo (_).

### Valores de Retorno
Para devolver un valor desde una función, la declaración es la siguiente:

string greeting = "Qué tal viejo!";

function sayHello() public returns (string) {
  return greeting;
}
En Solidity, la declaración de la función contiene al final tipo de dato del valor de retorno (en nuestro caso string).

### Modificadores de Función
La función de arriba no cambia el estado en Solidity, esto es que no cambia ningún valor o escribe nada.

En este caso podríamos declararla como función view, que significa que solo puede ver los datos pero no modificarlos:

function sayHello() public view returns (string) {
Solidity también contiene funciones pure, que significa que ni siquiera accedes a los datos de la aplicación. Por ejemplo:

function _multiply(uint a, uint b) private pure returns (uint) {
  return a * b;
}
Esta función no lee desde el estado de la aplicación - el valor devuelto depende por completo de los parámetros que le pasemos. En este caso deberíamos declarar la función como pure.


## Casteo de variables
A veces es necesario convertir entre tipos de datos. Por ejemplo en el siguiente caso:

uint8 a = 5;
uint b = 6;
// dará un error porque a * b devuelve un `uint` y no un `uint8`:
uint8 c = a * b;
// debemos de forzar la variable b para que sea convertida a `uint8`
uint8 c = a * uint8(b);
En el código de arriba. a * b devuelve un uint, pero estábamos intentando guardarlo como uint8, lo que podría causar problemas. Casteándolo a uint8 funcionará y el compilador no nos dará error.


## Eventos
Los eventos son la forma en la que nuestro contrato comunica que algo sucedió en la cadena de bloques a la interfaz de usuario, el cual puede estar 'escuchando' ciertos eventos y hacer algo cuando sucedan.

Ejemplo:

// declara el evento
event IntegersAdded(uint x, uint y, uint result);

function add(uint _x, uint _y) public {
  uint result = _x + _y;
  // lanza el evento para hacer saber a tu aplicación que la función ha sido llamada:
  emit IntegersAdded(_x, _y, result);
  return result;
}
La aplicación con la interfaz de usuario podría entonces estar escuchando el evento. Una implementación en JavaScript sería así:

YourContract.IntegersAdded(function(error, result) {
  // hacer algo con `result`
}


## Web3.js
Ethereum tiene una librería Javascript llamada Web3.js.

// Así es como accederiamos a nuestro contrato:
var abi = /* abi generado por el compilador */
var ZombieFactoryContract = web3.eth.contract(abi)
var contractAddress = /* nuestra dirección del contrato en Ethereum después de implementarlo */
var ZombieFactory = ZombieFactoryContract.at(contractAddress)
// `ZombieFactory` ha accedido a las funciones y eventos públicos de nuestro contrato

// algunos eventos están escuchando para recoger el valor del texto:
$("#ourButton").click(function(e) {
  var name = $("#nameInput").val()
  // Llama a la función `createRandomZombie` de nuestro contrato:
  ZombieFactory.createRandomZombie(name)
})

// Escucha al evento de `NewZombie`, y actualiza la interfaz
var event = ZombieFactory.NewZombie(function(error, result) {
  if (error) return
  generateZombie(result.zombieId, result.name, result.dna)
})

// Recogemos el adn del zombi y actualizamos nuestra imagen
function generateZombie(id, name, dna) {
  let dnaStr = String(dna)
  // rellenamos el ADN con ceros si es menor de 16 caracteres
  while (dnaStr.length < 16)
    dnaStr = "0" + dnaStr

  let zombieDetails = {
    // los primeros 2 dígitos hacen la cabeza. Tenemos 7 posibles cabezas, entonces hacemos % 7
    // para obtener un número entre 0 - 6, después le sumamos 1 para hacerlo entre 1 - 7. Tenemos 7
    // imagenes llamadas desde "head1.png" hasta "head7.png" que cargamos en base a 
    // este número:
    headChoice: dnaStr.substring(0, 2) % 7 + 1,
    // Los siguientes 2 dígitos se refieren a los ojos, 11 variaciones:
    eyeChoice: dnaStr.substring(2, 4) % 11 + 1,
    // 6 variaciones de camisetas:
    shirtChoice: dnaStr.substring(4, 6) % 6 + 1,
    // los últimos 6 digitos controlas el color. Actualiza el filtro CSS: hue-rotate
    // que tiene 360 grados:
    skinColorChoice: parseInt(dnaStr.substring(6, 8) / 100 * 360),
    eyeColorChoice: parseInt(dnaStr.substring(8, 10) / 100 * 360),
    clothesColorChoice: parseInt(dnaStr.substring(10, 12) / 100 * 360),
    zombieName: name,
    zombieDescription: "A Level 1 CryptoZombie",
  }
  return zombieDetails
}
Lo que nuestro código javascript hace es recoger los valores generados en zombieDetails más arriba, y usar un poco de magia Javascript en el navegador (usamos Vue.js) para intercambiar las imágenes y aplicar filtros CSS. Te daremos todo el código para hacer esto en una lección posterior.

