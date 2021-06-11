# cryptozombie
solidity game


El código Solidity está encapsulado en contratos. Un contrato es el bloque de construcción más básico de las aplicaciones de Ethereum — todas las variables y las funciones pertenecen a un contrato, y este será el punto de partida de todos tus proyectos.

Todo el código fuente en Solidity debería empezar con una declaración "version pragma" de la versión del compilador que debe de usarse para ese código. Esto previene problemas con versiones futuras del compilador que podrían no ser compatibles y fallar con tu código.

Esta declaración se asemeja a esto: 

````
pragma solidity ^0.4.25;
````

## Variables
Las Variables de estado se guardan permanentemente en el almacenamiento del contrato. Esto significa que se escriben en la cadena de bloques de Ethereum. Piensa en ellos como en escribir en una base de datos.

````
contract Example {
  // Esto se guardará permanentemente en la cadena de bloques
  uint myUnsignedInteger = 100;
}
````


## Estructuras
Podemos crear estructuras de datos más complejas en solidity y esto lo logramos mediante strucs

````
struct Person {
  uint age;
  string name;
}
````


## Arrays
Hay dos tipos de arrays en Solidity: arrays fijos y arrays dinámicos:

````
// Un Array con una longitud fija de 2 elementos:
uint[2] fixedArray;
// otro Array fijo, con longitud de 5 elementos:
string[5] stringArray;
// un Array dinámico, sin longitud fija que puede seguir creciendo:
uint[] dynamicArray;
````

También puedes crear arrays de estructuras. Si usásemos la estructura Person del capítulo anterior:

````
Person[] people; // Array dinámico, podemos seguir añadiéndole elementos
````

¿Recuerdas que las variables de estado quedan guardadas permanentemente en la blockchain? Así que crear un array de estructuras puede ser muy útil para guardar datos estructurados en tu contrato, como una base de datos.

### Arrays públicos
Puedes declarar un array como público, y Solidity creará automaticamente una función getter para acceder a él. La sintaxis es así:

````
Person[] public people;
````
Otros contratos entonces podrán leer (pero no escribir) de este array. Es un patrón de uso muy útil para guardar datos públicos en tu contrato.


## Funciones
Una declaración de una función en Solidity se asemeja a esto:

````
function eatHamburgers(string _name, uint _amount) {
  //cuerpo de la función
}
````

Nota: la convención (no obligatoria) es llamar los parámetros de las funciones con nombres que empiezan con un subrayado (_) para de esta forma diferenciarlos de variables globales.


## Creación de estructuras
Ahora aprenderemos como crear un nuevo Person y añadirlo a nuestro array people.

````
// crear un nuevo `Person`
Person satoshi = Person(172, "Satoshi");

// añadir esta persona a nuestro array
people.push(satoshi);
````


## Funciones públicas y privadas
En Solidity, las funciones son públicas (public) por defecto. Esto significa que cualquiera (o cualquier otro contrato) puede llamarla y ejecutar su código.

Esto no es algo que queramos que pase siempre, y de hecho puede hacer vulnerables tus contratos. Es por lo tanto una buena práctica marcar tus funciones como privadas (private), y solamente hacer públicas aquellas que queramos exponer al mundo exterior.

Vamos a ver como se declara una función privada:

````
uint[] numbers;

function _addToArray(uint _number) private {
  numbers.push(_number);
}
````

Esto significa que solo otras funciones dentro de tu contrato podrán llamar a esta función y añadir al array numbers.

Como puedes ver, usamos la palabra clave private después del nombre de la función. Y de las misma forma que con los parámetros de funciones, la convención es nombrar las funciones privadas empezando con un guíon bajo (_).

### Valores de Retorno
Para devolver un valor desde una función, la declaración es la siguiente:

````
string greeting = "Qué tal viejo!";

function sayHello() public returns (string) {
  return greeting;
}
````

En Solidity, la declaración de la función contiene al final tipo de dato del valor de retorno (en nuestro caso string).

### Modificadores de Función
La función de arriba no cambia el estado en Solidity, esto es que no cambia ningún valor o escribe nada.

En este caso podríamos declararla como función view, que significa que solo puede ver los datos pero no modificarlos:

````
function sayHello() public view returns (string) {
Solidity también contiene funciones pure, que significa que ni siquiera accedes a los datos de la aplicación. Por ejemplo:

function _multiply(uint a, uint b) private pure returns (uint) {
  return a * b;
}
````

Esta función no lee desde el estado de la aplicación - el valor devuelto depende por completo de los parámetros que le pasemos. En este caso deberíamos declarar la función como pure.


## Casteo de variables
A veces es necesario convertir entre tipos de datos. Por ejemplo en el siguiente caso:

```` 
uint8 a = 5;
uint b = 6;
// dará un error porque a * b devuelve un `uint` y no un `uint8`:
uint8 c = a * b;
// debemos de forzar la variable b para que sea convertida a `uint8`
uint8 c = a * uint8(b);
````

En el código de arriba. a * b devuelve un uint, pero estábamos intentando guardarlo como uint8, lo que podría causar problemas. Casteándolo a uint8 funcionará y el compilador no nos dará error.


## Eventos
Los eventos son la forma en la que nuestro contrato comunica que algo sucedió en la cadena de bloques a la interfaz de usuario, el cual puede estar 'escuchando' ciertos eventos y hacer algo cuando sucedan.

Ejemplo:

````
// declara el evento
event IntegersAdded(uint x, uint y, uint result);

function add(uint _x, uint _y) public {
  uint result = _x + _y;
  // lanza el evento para hacer saber a tu aplicación que la función ha sido llamada:
  emit IntegersAdded(_x, _y, result);
  return result;
}
````
La aplicación con la interfaz de usuario podría entonces estar escuchando el evento. Una implementación en JavaScript sería así:

````
YourContract.IntegersAdded(function(error, result) {
  // hacer algo con `result`
}
````


## Web3.js
Ethereum tiene una librería Javascript llamada Web3.js.

````
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
````

Lo que nuestro código javascript hace es recoger los valores generados en zombieDetails más arriba, y usar un poco de magia Javascript en el navegador (usamos Vue.js) para intercambiar las imágenes y aplicar filtros CSS. Te daremos todo el código para hacer esto en una lección posterior.


## Direcciones
La blockchain de Ethereum está creada por cuentas, que podrían ser como cuentas bancarias. Una cuenta tiene un balance de Ether (la divisa utilizada en la blockchain de Ethereum), y puedes recibir pagos en Ether de otras cuentas, de la misma manera que tu cuenta bancaria puede hacer transferencias a otras cuentas bancarias.

Cada cuenta tiene una dirección, que sería como el número de la cuenta bancaria. Es un identificador único que apuntado a una cuenta, y se asemejaría a algo así:

0x0cE446255506E92DF41614C46F1d6df9Cc969183

Por ahora solo necesitas entender que una direccion está asociada a un usuario específico (o un contrato inteligente).

Así que podemos utilizarlo como identificador único para nuestros zombis. Cuando un usuario crea un nuevo zombi interactuando con nuestra app, adjudicaremos la propiedad de esos zombis a la dirección de Ethereum que ha llamado a la función.

## Mapeando
Los mapeos son otra forma de organizar los datos en Solidity al igual que lo son las structs y los arrays

Definir un mapping se asemejaría a esto:

// Para una aplicación financial, guardamos un uint con el balance de su cuenta:

````
mapping (address => uint) public accountBalance;
````

// O podría usarse para guardar / ver los usuarios basados en ese userId

````
mapping (uint => string) userIdToName;
````

Un mapeo es esencialmente una asociación valor-clave para guardar y ver datos. En el primer ejemplo, la llave es un address (dirección) y el valor correspondería a un número de tipo uint, y en el segundo ejemplo la llave es un número de tipo uint y el valor un string.


## Msg.sender
En Solidity, hay una serie de variables globales que están disponibles para todas las funciones. Una de estas es msg.sender, que hace referencia a la dirección de la persona (o el contrato inteligente) que ha llamado a una función.

Nota: En Solidity, la ejecución de una función necesita empezar con una llamada exterior. Un contrato se sentará en la blockchain a esperar a que alguien llame añguna de sus funciones. Así que siempre habrá un msg.sender.

Aquí tenemos un ejemplo de como usar msg.sender y actualizar un mapping:

````
mapping (address => uint) favoriteNumber;

function setMyNumber(uint _myNumber) public {
  // Actualiza tu mapeo `favoriteNumber` para guardar `_myNumber` dentro de `msg.sender`
  favoriteNumber[msg.sender] = _myNumber;
  // ^ La sintaxis para guardar datos en un mapeo es como en los arrays
}

function whatIsMyNumber() public view returns (uint) {
  // Conseguimos el valor guardado en la dirección del emisor
  // Será `0` si el emisor no ha llamado a `setMyNumber` todavía
  return favoriteNumber[msg.sender];
}
````

En este trivial ejemplo, cualquiera puede llamar a setMyNumber y guardar un uint en nuestro contrato, que estará atado a su dirección. Entonces, cuando llamen a whatIsMyNumber, debería devolverles el uint que han guardado.

Usando msg.sender te da la seguridad de la blockchain de Ethereum — la única forma de que otra persona edite la información de esta sería robandole la clave privada asociada a la dirección Ethereum.


## Require
Require es una estructura de control que se usa en solidity para verificar si una expresión, si le expresión se cumple se deja continuar con la ejecución del código, caso contrario se detiene la ejecución.

````
function sayHiToVitalik(string _name) public returns (string) {
  // Compara si _name es igual a "Vitalik". Lanza un error si no lo son.
  // (Nota: Solidity no tiene su propio comparador de strings, por lo que
  // compararemos sus hashes keccak256 para ver si sus strings son iguales)
  require(keccak256(_name) == keccak256("Vitalik"));
  // Si es verdad, continuamos con la función:
  return "Hi!";
}
````

Si llamas a la función con sayHiToVitalik("Vitalik"), esta devolverá "Hi!". Si la llamas con cualquier otra entrada, lanzará un error y no se ejecutará.

De este modo require es muy útil a la hora de verificar que ciertas condiciones sean verdaderas antes de ejecutar una función.


## Herencia
Mejor que hacer un contrato extremandamente largo, a veces tiene sentido separar la lógica de nuestro código en multiples contratos para organizar el código.

Una característica de Solidity que hace más manejable esto es la herencia de los contratos:

````
contract Doge {
  function catchphrase() public returns (string) {
    return "So Wow CryptoDoge";
  }
}

contract BabyDoge is Doge {
  function anotherCatchphrase() public returns (string) {
    return "Such Moon BabyDoge";
  }
}
````

BabyDoge hereda de Doge. Eso significa que si compilas y ejecutas BabyDoge, este tendrá acceso tanto a catchphrase() como a anotherCatchphrase() (y a cualquier otra función publica que definamos en Doge).

Esto puede usarse como una herencia lógica (como una subclase, un Gato es un Animal). Pero también puede usarse simplemente para organizar tu código agrupando lógica similar en diferentes clases.


## Importar
Cuando tienes multiples archivos y quieres importar uno dentro de otro, Solidity usa la palabra clave import:

````
import "./someothercontract.sol";

contract newContract is SomeOtherContract {

}
````

Entonces si tenemos un fichero llamado someothercontract.sol en el mismo directorio que este contrato (eso es lo que significa ./), será importado por el compilador.


## Storage vs Memory
En Solidity, hay dos sitios donde puedes guardar variables - en storage y en memory.

Storage se refiere a las variables guardadas permanentemente en la blockchain. Las variables de tipo memory son temporales, y son borradas en lo que una función externa llama a tu contrato. Piensa que es como el disco duro vs la RAM de tu ordenador.

La mayoría del tiempo no necesitas usar estas palabras clave ya que Solidity las controla por defecto. Las variables de estado (variables declaradas fuera de las funciones) son por defecto del tipo storage y son guardadas permanentemente en la blockchain, mientras que las variables declaradas dentro de las funciones son por defecto del tipo memory y desaparecerán una vez la llamada a la función haya terminado.

Aun así, hay momentos en los que necesitas usar estas palabras clave, concretamente cuando estes trabajando con structs y arrays dentro de las funciones:

````
contract SandwichFactory {
  struct Sandwich {
    string name;
    string status;
  }

  Sandwich[] sandwiches;

  function eatSandwich(uint _index) public {
    // Sandwich mySandwich = sandwiches[_index];

    // ^ Parece algo sencillo, pero solidity te dará un warning
    // diciendo que deberías declararlo `storage` o `memory`.

    // De modo que deberias declararlo como `storage`, así:
    Sandwich storage mySandwich = sandwiches[_index];
    // ...donde `mySandwich` es un apuntador a `sandwiches[_index]`
    // de tipo storage, y...
    mySandwich.status = "Eaten!";
    // ...esto cambiará permanentemente `sandwiches[_index]` en la blockchain.

    // Si únicamente quieres una copia, puedes usar `memory`:
    Sandwich memory anotherSandwich = sandwiches[_index + 1];
    // ...donde `anotherSandwich` seria una simple copia de 
    // los datos en memoria, y...
    anotherSandwich.status = "Eaten!";
    // ...modificará la variable temporal y no tendrá efecto
    // en `sandwiches[_index + 1]`. Pero puedes hacer lo siguiente:
    sandwiches[_index + 1] = anotherSandwich;
    // ...si quieres que la copia con los cambios se guarde en la blockchain.
  }
}
````


## Más en la Visibilidad de Funciones: Internal y External
Además de public y private, Solidity tiene dos tipos de visibilidad más para las funciones: internal y external.

### internal 
Es lo mismo que private, a excepción de que es también accesible desde otros contratos que hereden de este. (¡Ey, suena como lo que necesitamos aquí!).

### external 
Es parecido a public, a excepción que estas funciones SOLO puedes ser llamadas desde fuera del contrato — no pueden ser llamadas por otras funciones dentro de ese contrato. Hablaremos más adelante sobre cuando querrás usar external vs public.

Para declarar las funciones internal o external, la sintaxis es igual que private y public:

````
contract Sandwich {
  uint private sandwichesEaten = 0;

  function eat() internal {
    sandwichesEaten++;
  }
}

contract BLT is Sandwich {
  uint private baconSandwichesEaten = 0;

  function eatWithBacon() public returns (string) {
    baconSandwichesEaten++;
    // Podemos llamar a esta función aquí porque es internal
    eat();
  }
}
````


## Interfaz Interactuando con otros contratos
Para que nuestro contrato pueda hablar a otro contrato de la blockchain que no poseemos, necesitamos definir una interfaz.

Vamos a ver un simple ejemplo. Digamos que hay un contrato en la blockchain tal que así:

````
contract LuckyNumber {
  mapping(address => uint) numbers;

  function setNum(uint _num) public {
    numbers[msg.sender] = _num;
  }

  function getNum(address _myAddress) public view returns (uint) {
    return numbers[_myAddress];
  }
}
````

Este seria un simple contrato donde cualquiera puede guardar su número de la suerte, y este estará asociado a su dirección de Ethereum. De esta forma cualquiera podría ver el número de la suerte de una persona usando su dirección.

Ahora digamos que tenemos un contrato externo que quiere leer la información de este contrato usando la función getNum.

Primero debemos usar una interfaz del contrato LuckyNumber:

contract NumberInterface {
  function getNum(address _myAddress) public view returns (uint);
}
Ten en cuenta que esto se asemeja a definir un contrato, con alguna diferencia. Primero, solo declaramos las funciones con las que queremos interactuar - en este caso getNum — y no mencionamos ninguna otra función o variables de estado.

Segundo, no definimos el cuerpo de la función. En vez de usar las llaves ({ y }), solamente terminaremos la función añadiendo un punto y coma al final de la declaración (;).

Sería como definir el esqueleto del contrato. Así es como conoce el compilador a las interfaces.

Incluyendo esta interfaz en el código de tu dapp nuestro contrato sabe como son las funciones de otro contrato, como llamarlas, y que tipo de respuesta recibiremos.

### Usando una Interfaz
Continuando con nuestro ejemplo anterior de NumberInterface, una vez hemos definido la interfaz como:

````
contract NumberInterface {
  function getNum(address _myAddress) public view returns (uint);
}
````

Podemos usarla en el contrato de esta manera:

````
contract MyContract {
  address NumberInterfaceAddress = 0xab38... 
  // ^ La dirección del contrato FavoriteNumber en Ethereum
  NumberInterface numberContract = NumberInterface(NumberInterfaceAddress)
  // Ahora `numberContract` está apuntando al otro contrato

  function someFunction() public {
    // Ahora podemos llamar a `getNum` de ese contrato:
    uint num = numberContract.getNum(msg.sender);
    // ...y haz algo con `num` aquí
  }
}
````
De esta manera, tu contrato puede interactuar con otro contrato de la blockchain de Ethereum, siempre y cuando la función esté definida como public o external.


## Múltiples retornos
Una función puede devolver múltiples valores. Vamos a ver como manejarlos:

````
function multipleReturns() internal returns(uint a, uint b, uint c) {
  return (1, 2, 3);
}

function processMultipleReturns() external {
  uint a;
  uint b;
  uint c;
  // Así es como hacemos múltiples asignaciones:
  (a, b, c) = multipleReturns();
}

// O si solo nos importa el último de estos valores:
function getLastReturnValue() external {
  uint c;
  // Podemos dejar el resto de campos en blanco:
  (,,c) = multipleReturns();
}
````

## Inmutabilidad de los Contratos
Hasta ahora, Solidity se ha parecido bastante a otros lenguajes como JavaScript. Pero hay unas cuantas maneras en las que las DApps de Ethereum son diferentes a las aplicaciones normales.

Para empezar, despues de implementar un contrato en Ethereum, es inmutable, lo que significa que nunca va a ser modificado o actualizado de nuevo.

El código inicial que implementes en el contrato es el que va a permanecer, permanentemente, en la blockchain. Esto es debido a que una de las mayores preocupaciones de Solidity es la seguridad. Si hay un error en el código del contrato, no hay forma de solucionarlo más adelante. Tendrás que decirles a tus usuarios que empiecen a usar otra dirección de contrato inteligente que incluya ese arreglo.

Pero es también una característica de los contratos inteligentes. El código es la ley. Si lees el código de un contrato inteligente y lo verificas, vas a estar totalmente seguro de que cada vez que lo llames va a hacer exactamente lo que el código dice que va a hacer. Nadie va a poder más adelante cambiar la función y que te devuelva resultados inesperados.

### Dependencias externas
¿qué pasa si un contrato externo del que hacemos uso tiene un error y alguien destruye toda la iformación contenida en él?

Es improbable, pero si esto pasase dejaría nuestra DApp completamente inservible si harcodeamos direcciones externas en nuestro contrato y no podremos modificar nuestro contrato para solucionarlo.

Por este motivo, a veces tiene sentido programar funciones que te permitan actualizar partes de nuestra DApp.

Por ejemplo, en vez de hardcodear la dirección del contrato en nuestra DApp, probablemente deberíamos tener una función que nos deje cambiar esta dirección en el futuro en caso de que algo ocurra con el contrato del cual dependemos.


## Contratos Apropiables
Queremos tener el poder de actualizar nuestro contrato, pero no queremos que todo el mundo sea capaz de hacerlo.

Para manejar casos como este, una práctica emergente común es hacer el contrato Ownable — significa que tiene un dueño (tú) con privilegios especiales.

Contrato Ownable de OpenZeppelin
Abajo está el contrato Ownable definido en la librería Solidity de OpenZeppelin. OpenZeppelin es una librería segura donde hay contratos inteligentes para utilizar en tus propias DApps revisados por la comunidad. Despues de esta lección, mientras esperas ansiosamente la liberación de la Lección 4, ¡te recomendamos encarecidamente que visites su sitio web para fomentar tu aprendizaje!

Échale un vistazo al contrato más abajo. Vas a ver algunas cosas que no hemos aprendido aún, pero no te preocupes, hablaremos de ellas más adelante.

````
/**
 * @title Ownable
 * @dev The Ownable contract has an owner address, and provides basic authorization control
 * functions, this simplifies the implementation of "user permissions".
 */
contract Ownable {
  address public owner;
  event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

  /**
   * @dev The Ownable constructor sets the original `owner` of the contract to the sender
   * account.
   */
  function Ownable() public {
    owner = msg.sender;
  }

  /**
   * @dev Throws if called by any account other than the owner.
   */
  modifier onlyOwner() {
    require(msg.sender == owner);
    _;
  }

  /**
   * @dev Allows the current owner to transfer control of the contract to a newOwner.
   * @param newOwner The address to transfer ownership to.
   */
  function transferOwnership(address newOwner) public onlyOwner {
    require(newOwner != address(0));
    OwnershipTransferred(owner, newOwner);
    owner = newOwner;
  }
}
````

### Alguna de las cosas que no hemos visto todavía:

Constructores: function Ownable() es un constructor, que es una función especial opcional que tiene el mismo nombre que el contrato. Será ejecutada una única vez, cuando el contrato sea creado por primera vez.
Modificadores de Funciones: modifier onlyOwner(). Los modificadores son como semifunciones que son usadas para modificar otras funciones, normalmente para comprobar algunos requisitos antes de la ejecución. En este caso, onlyOwner puede ser usada para limitar el acceso y que solo el dueño del contrato pueda ejecutar función. Hablaremos sobre los modificadores en el siguiente capítulo, y que hace ese extraño _;.
Palabra clave indexed: no te preocupes por esto, no lo necesitamos todavía.

Basicamente Ownable hace lo siguiente:
Cuando el contrato ha sido creado, su constructor inicializa owner con msg.sender (la persona que lo ha implementado)

Añade el modificador onlyOwner, que puede restringir el acceso a solo el owner en una función.

Permite transferir el contrato a un nuevo owner.

onlyOwner es un requisito tan común que la mayoría de las DApps en Solidity suelen empezar con un copia/pega de este contrato Ownable, y después su primer contrato heredaría de él.

Como queremos limitar el acceso de setKittyContractAddress a onlyOwner, vamos a hacer lo mismo para nuestro contrato.


## Modificador de Función onlyOwner
Cuando un contrato hereda de otro contrato Ownable, podemos usar también el modificador de función onlyOwner dentro de este último.

Esto es por como funciona la herencia de contratos.


### Modificadores de Funciones
Un modificador de función es igual que una función, pero usa la palabra clave modifier en vez de function. Pero no puede ser llamado directamente como una función - en vez de eso, podemos añadirle el nombre del modificador al final de la definición de la función para cambiar el comportamiento de ella.

Vamos a verlo con más detalle examinando onlyOwner:

````
/**
 * @dev Throws if called by any account other than the owner.
 */
modifier onlyOwner() {
  require(msg.sender == owner);
  _;
}
````

Tendremos que usar el modificador de esta manera:

````
contract MyContract is Ownable {
  event LaughManiacally(string laughter);

  // Mira como se usa `onlyOwner` debajo:
  function likeABoss() external onlyOwner {
    LaughManiacally("Muahahahaha");
  }
}
````

Observa el modificador onlyOwner en la función likeABoss. Cuando llamas a likeABoss, el código dentro de onlyOwner se ejecuta primero. Entonces cuando se encuentra con la sentencia _; en onlyOwner, vuelve y ejecuta el código dentro de likeABoss.

Hay otras maneras de usar los modificadores, uno de los casos de uso más comunes es añadir una rápida comprobación require antes de que se ejecute la función.

En el caso de onlyOwner, añadiendole este modificador a la función hace que solo el dueño del contrato (tú, si eres el que lo ha implementado) puede llamar a la función.

Nota: Darle poderes especiales de esta manera al dueño a lo largo del contrato es usualmente necesario, pero puede también ser usado malintencionadamente.Por ejemplo, el dueño puede añadir una función oculta.

Así que es importante recordar que solo porque una DApp esté en Ethereum no significa automáticamente que sea descentralizada — tienes que leerte el código fuente completo para asegurarte que esté libre de poderes especiales controlados por su dueño que puedan ser potencialmente preocupantes. Hay un cuidadoso balance entre mantener el control sobre la DAPP para poder arreglar los bugs potenciales, y construir una plataforma sin dueño donde tus usuarios puedan confiar la seguridad de sus datos.


## Gas
¡Genial! Ahora sabemos como actualizar partes clave de la DApp mientras prevenimos que otros usuarios nos fastidien nuestros contratos.

Vamos a ver otra característica por la que Solidity es diferente a otros lenguajes de programación:

Gas — el combustible que mueven las DApps de Ethereum
En Solidity, tus usuarios tienen que pagar cada vez que ejecuten una función en tu DApp usando una divisa llamada gas. Los usuarios compran gas con Ether (la divisa de Ethereum), así que deben gastar ETH para poder ejecutar funciones en tu DApp.

La cantidad de gas necesaria para ejecutar una función depende en cuán compleja sea la lógica de la misma. Cada operación individual tiene un coste de gas basado aproximadamente en cuantos recursos computacionales se necesitarán para llevarla a cabo (p. ej. escribir en memoria es más caro que añadir dos integers). El total coste de gas de tu función es la suma del coste de cada una de sus operaciones.

Como ejecutar funciones cuestan dinero real a los usuarios, la optimización de código es mucho más importante en Ethereum que en cualquier otro lenguaje. Si tu código es descuidado, tus usuarios van a tener que pagar un premium para ejecutar tus funciones - esto puede suponer millones de dolares gastados innecesariamente por miles de usuarios en tasas.

¿Por qué es necesario el gas?
Ethereum es como un ordenador grande, lento, pero extremandamente seguro. Cuando ejecutas una función, cada uno de los nodos de la red necesita ejecutar esa misma función para comprobar su respuesta - miles de nodos verificando cada ejecución de funciones es lo que hace a Ethereum descentralizado, y que sus datos sean inmutables y resistentes a la censura.

Los creadores de Ethereum querian estar seguros de que nadie pudiese obstruir la red con un loop infinito, o acaparar todos los recursos de la red con cálculos intensos. Por eso no hicieron las transacciones gratuitas, y los usuarios tienen que pagar por su poder de computo así como por su espacio en memoria.

Nota: Esto no es necesariamente verdadero en las sidechains, así como los autores de CryptoZombies están construyendo en Loom Network. Es probable que nunca se ejecute un juego como World of Warcraft directamente en la mainnet de Ethereum - el coste de gas sería excesivamente caro. Pero puede ejecutarse en una sidechain con un algoritmo de consenso diferente. Hablaremos más sobre que tipos de DApps querrás implementar en la sidechain y cual en la mainnet de Ethereum en lecciones futuras.

Empaquetado struct para ahorrar gas
En la Lección 1, mencionamos que hay otros tipos de uint: uint8, uint16, uint32, etc.

Normalmente no hay ningún beneficio en usar cualquiera de estos subtipos porque Solidity reserva 256 bits de almacenamiento independientemente del tamaño del uint. Por ejemplo, usando uint8 en vez de uint (uint256) no te ahorrará nada de gas.

Pero hay una excepción para esto: dentro de los struct.

Si tienes varios uints dentro de una estructura, usar un uint de tamaño reducido cuando sea posible permitirá a Solidity empaquetar estas variables para que ocupen menos espacio en memoria. Por ejemplo:

````
struct NormalStruct {
  uint a;
  uint b;
  uint c;
}

struct MiniMe {
  uint32 a;
  uint32 b;
  uint c;
}

// `mini` costará menos gas que `normal` debido al empaquetado de la estructura
NormalStruct normal = NormalStruct(10, 20, 30);
MiniMe mini = MiniMe(10, 20, 30); 
````

Por esta razón, dentro de una estructura querrás usar los subtipos más pequeños que vayas a necesitar.

Querrás también agrupar los tipos de datos que sean iguales (es decir, ponerlos al lado en la estructura) así Solidity podrá minimizar el espacio requerido. Por ejemplo, una estructura con campos uint c; uint32 a; uint32 b; costará menos gas que una estructura con campos uint32 a; uint c; uint32 b; porque los campos uint32 están agrupados al lado.

## Unidades de Tiempo
Solidity proporciona algunas unidades nativas para trabajar con el tiempo.

La variable now devolverá el actual tiempo unix (la cantidad de segundos que han pasado desde el 1 de Enero de 1970).

Nota: El tiempo unix es tradicionalmente guardado en un número de 32 bits. Esto nos llevará a el problema del "Año 2038", donde las variables timestamp de tipo unix desbordarán y dejará inservibles muchos sistemas antiguos. Así que si queremos que nuestra DApp siga funcionando después de 20 años, podemos usar un número de 64 bits - pero de mientras nuestros usuarios tendrán que gastar más gas para usar nuestra DApp. ¡Decisiones de diseño!

Solidity también contiene segundos, minutos, horas, días, semanas y años como unidades de tiempo. Estos convertirán a un uint la cantidad de segundos que contengan esos números. Es decir 1 minuto son 60, 1 hora son 3600 (60 segundos x 60 minutos), 1 día son 86400 (24 horas x 60 minutos x 60 segundos), etc.

Aquí un ejemplo de como estas unidades pueden ser útiles:

uint lastUpdated;

````
// Ajustamos `lastUpdated` a `now`
function updateTimestamp() public {
  lastUpdated = now;
}

// Devolverá `true` si han pasado 5 minutos desde que `updateTimestamp`
// fue llamado, `false` si no han pasdo 5 minutos todavía
function fiveMinutesHavePassed() public view returns (bool) {
  return (now >= (lastUpdated + 5 minutes));
}
````


## Funciones Públicas & Seguridad
Una práctica importante de seguridad es examinar todas tus funciones public y external, y prueba a pensar las maneras en las que los usuarios podrían abusar de ellas. Recuerda - a no ser que estas funciones tengan un modificador como onlyOwner, cualquier usuario puede llamarlas y pasarles los datos que quieran.

Una mánera fácil de prevenir problemas que cualquiera llame una función es hacer la función internal.


## Modificadores de funciones con argumentos
Antes hemos visto el ejemplo simple de onlyOwner. Pero un modificador de función puede también recibir argumentos. Por ejemplo:

````
// Un mapeo para guardar la edad de un usuario:
mapping (uint => uint) public age;

// Modificador que requiere que ese usuario sea mayor a cierta edad:
modifier olderThan(uint _age, uint _userId) {
  require (age[_userId] >= _age);
  _;
}

// Tiene que ser mayor a 16 años para conducir un coche (en EEUU, al menos).
// Podemos llamar al modificador de función `olderThan` pasandole argumentos de esta manera:
function driveCar(uint _userId) public olderThan(16, _userId) {
  // La lógica de la función
}
````

Puedes ver aquí como el modificador olderThan recibe los argumentos de la misma manera que las funciones. Y cómo la función driveCar le pasa sus argumentos al modificador.


## Las funciones view (IMPORTANTE) 
Las funciones view no cuestan gas cuando son llamadas externamente por un usuario.

Esto es debido a que las funciones view no cambia nada en la blockchain - solo leen datos. Así que marcar una función con view le dice a web3.js que solo necesita consultar a tu nodo local de Ethereum para ejecutar la función, y que no necesita crear ninguna transacción en la blockchain (la cual debería ejecutarse por todos y cada uno de los nodos, y costaría gas).

Hablaremos sobre configurar web3.js en tu propio nodo mas tarde. Por ahora la mayor ventaja es que puedes optimizar el uso de gas de tu DApp haciendo que tus usuarios utilicen funciones external view siempre que sea posible.

Nota: Si una función view es llamada internamente por otra función que no sea view en el mismo contrato, esta seguira costando gas. Esto es porque la otra función crea una transacción en Ethereum, y necesita ser verificada por el resto de nodos. Por lo que las funciones view son gratuitas siempre que sean llamadas externamente.


## Storage es caro
Una de las operaciones más caras en Solidity es usar storage — especialmente la escritura.

Esto es debido a que cada vez que escribes o cambias algún dato, este se guarda permanentemente en la blockchain. ¡Para siempre! Miles de nodos alrededor del mundo necesitan guardar esos datos en sus discos duros, y esa cantidad de datos sigue creciendo a lo largo del tiempo a medida que crece la blockchain. Así que tiene que haber un coste para hacer eso.

Para seguir manteniendo los costes bajos, querrás evitar escribir datos en "storage" a no ser que sea absolutemente necesario. A veces esto implica usar en programación una lógica ineficiente - como volver a construir un array en memoria cada vez que una función es llamada en vez de simplemente guardar ese array en una variable para acceder a sus datos más rápido.

En la mayoría de lenguajes de programación, usar bucles sobre largos conjuntos de datos es costoso. Pero en Solidity, esta es una manera más barata que usar storage si está en una función external view, debido a que las funciones view no les cuesta a los usuarios nada de gas. (¡Y el gas le cuesta a tus usuarios dinero real!).

Veremos los bucles for en el siguiente capítulo, pero primero, vamos a ver como declarar los arrays en memoria.

### Declarando arrays en memoria
Puedes usar la palabra clave memory con arrays para crear un nuevo arrays dentro de una función sin necesidad de escribir nada en storage. El array solo existirá hasta el final de la llamada de la función, y esto es más barato en cuanto a gas que actualizar un array en storage - gratis si está dentro de una función view llamada externamente.

Así es como se declara un array en memoria:

````
function getArray() external pure returns(uint[]) {
  // Instanciamos un nuevo array en memoria con una longitud de 3
  uint[] memory values = new uint[](3);
  // Le añadimos algunos valores
  values.push(1);
  values.push(2);
  values.push(3);
  // Devolvemos el arrays
  return values;
}
````

Esto es un ejemplo trivial para enseñarte a cómo usar la sintaxis, pero en el próximo capítulo veremos como combinarlo con bucles for para usarlo en casos de uso reales.

Nota: los arrays de tipo memory deben ser creados con una longitud como argumento (en este ejemplo, 3). Actualmente no pueden ser redimensionados como los arrays storage pueden serlo usando array.push(), de todas maneras esto podría cambiar en futuras versiones de Solidity.


## Bucles For
La sintaxis de los bucles for en Solidity es similar a JavaScript.

Vamos a ver un ejemplo donde queremos hacer un array de números pares:

````
function getEvens() pure external returns(uint[]) {
  uint[] memory evens = new uint[](5);
  // Guardamos el índice del nuevo array:
  uint counter = 0;
  // Iteramos del 1 al 10 con un bucle for:
  for (uint i = 1; i <= 10; i++) {
    // Si `i` es par...
    if (i % 2 == 0) {
      // Añadelo a nuestro array
      evens[counter] = i;
      // Incrementamos el contador al nuevo índice vacío de `evens`:
      counter++;
    }
  }
  return evens;
}
````

La función devolverá un array con este contenido [2, 4, 6, 8, 10].


## Payable
Hasta ahora, hemos cubierto unos cuantos modificadores de función. Puede resultar difícil tratar de recordar todo, así que hagamos un breve repaso:

1. Tenemos modificadores de visibilidad que controlan desde dónde y cuándo la función puede ser llamada: private significa que sólo puede ser llamada desde otras funciones dentro del contrato; internal es como private pero también puede ser llamada por contratos que hereden desde este; external sólo puede ser llamada desde afuera del contrato; y finalmente public puede ser llamada desde cualquier lugar, tanto internamente como externamente.

2. También tenemos modificadores, los cuales nos dicen cómo interactúa la función con la BlockChain: view nos indica que al ejecutar la función, ningún dato será guardado/cambiado. pure nos indica que la función no sólo no guarda ningún dato en la blockchain, si no que tampoco lee ningún dato de la blockchain. Ambos no cuestan nada de combustible para llamar si son llamados externamente desde afuera del contrato (pero si cuestan combustible si son llamado internamente por otra función).

3. Luego tenemos los modifiers personalizados, onlyOwner, por ejemplo. Para estos podemos definir la lógica personalizada para determinar cómo afectan a una función.

Todos estos modificadores pueden ser apilados juntos en una definición de función de la siguiente manera:

````
function test() external view onlyOwner anotherModifier { /* ... */ }
````

### El Modificador payable
Las funciones payable son parte de lo que hace de Solidity y Ethereum algo tan genial — son un tipo de función especial que pueden recibir Ether.

Pienselo por un momento. Cuando llama una función API en un servidor web normal, no puede enviar dólares (USD$) junto con su llamada de función — ni enviar Bitcoin.

Pero en Ethereum, ya que tanto el dinero (Ether), los datos (payload de transacción) y el mismo código de contrato viven en Ethereum, es posible para usted llamar a una función y pagar dinero por el contrato al mismo tiempo.

Esto abarca una lógica realmente interesante, como requerir cierto pago por el contrato para, de esta manera, ejecutar una función.

Veamos un ejemplo

````
contract OnlineStore {
  function buySomething() external payable {
    // Check to make sure 0.001 ether was sent to the function call:
    require(msg.value == 0.001 ether);
    // If so, some logic to transfer the digital item to the caller of the function:
    transferThing(msg.sender);
  }
}
````
Aquí, msg.value es una manera de ver cuanto Ether fue enviado al contrato, y ether es una unidad incorporada.

Lo que sucede aquí es que alguien llamaría a la función desde web3.js (desde la interfaz JavaScript del DApp) de esta manera:

````
// Assuming `OnlineStore` points to your contract on Ethereum:
OnlineStore.buySomething({from: web3.eth.defaultAccount, value: web3.utils.toWei(0.001)})
````

Nótese el campo value, donde la llamada de función javascript especifíca cuánto de ether enviar (0.001). Si piensas en la transacción como un sobre, y los parámetros que usted envía a la llamada de función son los contenidos de la carta que coloca adentro, entonces añadir un value es como poner dinero en efectivo dentro del sobre — la carta y el dinero son entregados juntos al receptor.

Nota: Si una función no es marcada como payable y usted intenta enviar Ether a esta, como se hizo anteriormente, la función rechazará su transacción.


## Retiros
En el capitulo anterior, aprendimos cómo enviar Ether a un contrato. Entonces ¿Qué ocurre cuando lo envía?

Luego de enviar Ether a un contrato, este se almacena en la cuenta de Ethereum del contrato y estará atrapado ahí — a menos que añada una función para retirar el Ether del contrato

Puede escribir una función para retirar Ether del contrato de la siguiente forma:

````
contract GetPaid is Ownable {
  function withdraw() external onlyOwner {
    owner.transfer(this.balance);
  }
}
````
Nótese que estamos utilizando owner y onlyOwner del contrato Ownable, asumiendo que este fue importado.

Puede transferir Ether a una dirección utilizando la función transfer y this.balance devolverá el balance total almacenado en el contrato. Así que si 100 usuarios han pagado 1 Ether a nuestro contrato, this.balance sería igual a 100 Ether.

Puede utilizar transfer para enviar fondos a cualquier dirección de Ethereum. Por ejemplo, podría tener una función que transfiera Ether de vuelta al msg.sender si rebasan el precio al pagar un item.

````
uint itemFee = 0.001 ether;
msg.sender.transfer(msg.value - itemFee);
O en un contrato con un comprador y un vendedor, usted podría guardar la dirección del vendedor en la memoria, luego, cuando alguien adquiera su item, transferirle la tasa pagada por el comprador: seller.transfer(msg.value).
````

Estos son algunos ejemplos de lo que hace de la programación de Ethereum algo realmente genial — puede tener mercados descentralizados como este que no son controlados por nadie.


## Números Aleatorios
¿Cómo generamos números aleatorios en Solidity?

La respuesta correcta es que no puede. Bueno, al menos no puede hacerlo de una manera segura.

Veamos por qué.

### La generación aleatoria de números a través de keccak256
La mejor fuente de aleatoriedad que tenemos en solidity es la función hash keccak256.

Podríamos hacer algo como lo siguiente para generar un número aleatorio:

````
// Generate a random number between 1 and 100:
uint randNonce = 0;
uint random = uint(keccak256(now, msg.sender, randNonce)) % 100;
randNonce++;
uint random2 = uint(keccak256(now, msg.sender, randNonce)) % 100;
````

Lo que esto haría es tomar la marca de tiempo de now, el msg.sender, y un nonce (un número que sólo se utiliza una vez, para que no ejecutemos dos veces la misma función hash con los mismos parámetros de entrada) en incremento.

Luego entonces utilizaría keccak para convertir estas entradas a un hash aleatorio, convertir ese hash a un uint y luego utilizar % 100 para tomar los últimos 2 dígitos solamente, dándonos un número totalmente aleatorio entre 0 y 99.

#### Este método es vulnerable a ataques de nodos deshonestos
En Ethereum, cuando llama a una función en un contrato, lo transmite a un nodo o nodos en la red como una transacción. Los nodos en la red luego recolectan un montón de transacciones, intentan ser el primero en resolver el problema de matemática intensamente informático como una "Prueba de Trabajo", para luego publicar ese grupo de transacciones junto con sus Pruebas de Trabajo (PoW) como un bloque para el resto de la red.

Una vez que un nodo ha resuelto la PoW, los otros nodos dejan de intentar resolver la PoW, verifican que las transacciones en la lista de transacciones del otro nodo son válidas, luego aceptan el bloque y pasan a tratar de resolver el próximo bloque.

##### Esto hace que nuestra función de números aleatorios sea explotable.

Digamos que teníamos un contrato coin flip — cara y duplica su dinero, sello y pierde todo. Digamos que utilizó la función aleatoria anterior para determinar cara o sello. (random >= 50 es cara, random < 50 es sello).

Si yo estuviera ejecutando un nodo, podría publicar una transacción a mi propio nodo solamente y no compartirla. Luego podría ejecutar la función coin flip para ver si gané — y si perdí, escojo no incluir esa transacción en el próximo bloque que estoy resolviendo. Podría seguir haciendo esto indefinidamente hasta que finalmente gané el lanzamiento de la moneda y resolví el siguiente bloque, beneficiandome de ello.

### Entonces ¿Cómo generamos números aleatorio de manera segura en Ethereum?
Ya que todos los contenidos de la blockchain son visibles para todos los participantes, este es un problema dificil, y su solución está más allá del rango de este tutorial. Puede leer [este hilo de StackOverflow](https://ethereum.stackexchange.com/questions/191/how-can-i-securely-generate-a-random-number-in-my-smart-contract "Named link title") para que se haga un idea. Una idea sería utilizar un oráculo para ingresar una función de número aleatorio desde fuera de la blockchain de Ethereum.

Por supuesto, debido a que cientos de miles de nodos de Ethereum en la red están compitiendo por resolver el próximo bloque, mis probabilidades de resolver el siguiente bloque son extremadamente escasas. Me tomaría mucho tiempo o recursos informáticos para explotar esto y que sea beneficioso — pero si la recompensa fuera lo suficientemente alta (como si pudiera apostar $100,000,000 en la función coin flip), para mi valdría la pena atacar.

Así que mientras esta generación de número aleatorio NO sea segura en Ethereum, en la práctica a menos que nuestra función aleatoria tenga mucho dinero en riesgo, es probable que los usuarios de su DAPP no tengan suficientes recursos para atacarla.

## Tokens en Ethereum
Hablemos de tokens.

Si has estado en el ecosistema de Ethereum durante un tiempo, es probable que hayas oído hablar de los tokens — en concreto, de los tokens ERC20.

Un token es básicamente un contrato inteligente que sigue algunas reglas comunes, es decir, implementa un conjunto estándar de funciones que comparten el resto de tokens (contratos), como por ejemplo transfer(address _to, uint256 _value) y balanceOf(address _owner).

Internamente, el contrato inteligente tiene por lo general un mapeo, mapping(address => uint256) balances, que realiza un seguimiento de cuánto saldo tiene cada dirección.

Así que, básicamente, un token es sólo un contrato que realiza un seguimiento de quién posee la cantidad de ese token y algunas funciones para que los usuarios puedan transferir sus tokens a otras direcciones.

### ¿Por qué importa?
Dado que todos los tokens ERC20 comparten el mismo conjunto de funciones con los mismos nombres, todos pueden interactuar de la misma manera.

Esto significa que si creas una aplicación que es capaz de interactuar con un token de tipo ERC20, también será capaz de interactuar con cualquier token de tipo ERC20. De esta forma, más tokens se pueden añadir fácilmente a tu aplicación en el futuro, sin necesidad de tener códigos personalizados por cada uno de los tipos de token ERC20. Simplemente puede conectar la nueva dirección de contrato del token y boom, tu aplicación tiene otro token que puedes usar.

Un ejemplo de esto sería un exchange (casa de cambio). Cuando un exchange añade un nuevo token ERC20 solo necesita agregar otro contrato inteligente con el que comunicarse. Los usuarios pueden indicarle a ese contrato que envíe los tokens a la dirección de su monedero en el exchange, y el exchange puede indicarle al contrato que devuelva los tokens a los usuarios cuando soliciten un retiro.

El exchange solo necesita implementar esta lógica de transferencia una vez, luego, cuando quiera añadir un nuevo token ERC20, es simplemente una cuestión de añadir la nueva dirección del contrato a su base de datos.

### Otros estándares de token
Hay otro estándar de token que encaja mucho mejor con los cripto-coleccionables como CryptoZombies, y se llaman tokens ERC721.

Los tokens ERC721 no son intercambiables entre sí, ya que se supone que cada uno de ellos es totalmente único e indivisible. Solo se pueden intercambiar en unidades completas, y cada uno tiene una ID única.

Nota: Tenga en cuenta que el uso de un estándar como ERC721 tiene el beneficio de que no tenemos que implementar la lógica que está detrás del sistema que permite las operaciones de compra/venta en nuestro contrato. Si cumplimos con la especificación, alguien más podría construir otra plataforma de intercambio para los activos ERC721, y nuestros tokens ERC721 se podrían usar en esa plataforma. Por lo tanto, existen beneficios claros en el uso de un token estándar en lugar de desarrollar su propia lógica comercial.


## Estandar ERC721 Standard, Herencia Multiple
Vamos a echar un vistazo al estándar ERC721:

````
contract ERC721 {
  event Transfer(address indexed _from, address indexed _to, uint256 _tokenId);
  event Approval(address indexed _owner, address indexed _approved, uint256 _tokenId);

  function balanceOf(address _owner) public view returns (uint256 _balance);
  function ownerOf(uint256 _tokenId) public view returns (address _owner);
  function transfer(address _to, uint256 _tokenId) public;
  function approve(address _to, uint256 _tokenId) public;
  function takeOwnership(uint256 _tokenId) public;
}
````

Nota: El estándar ERC721 actualmente es un borrador, y aún no hay una implementación oficialmente acordada.

### Implementando un contrato de token
Al implementar un contrato de token, lo primero que hay que hacer es copiar el fichero de la interfaz a nuestro propio directorio e importarlo con import "./erc721.sol";. Luego, declararemos a nuestro contrato que herede de él y reescribiremos cada método con una definición de la función.

En Solidity, nuestro contrato puede tener herencia múltiple de la siguiente forma:

````
contract SatoshiNakamoto is NickSzabo, HalFinney {
  // Omg, the secrets of the universe revealed!
}
````

Como puede ver, para usar herencia múltiple sólo es necesario separar por comas , cada uno de los contratos múltiples. En este caso, nuestro contrato heredará de NickSzabo y HalFinney.


## balanceOf & ownerOf
### balanceOf
````
function balanceOf(address _owner) public view returns (uint256 _balance);
````

Esta función simplemente recibe una dirección address, y devuelve cuántos tokens tiene esa dirección address.

### ownerOf
````
function ownerOf(uint256 _tokenId) public view returns (address _owner);
````

Esta función recibe un ID de un token (en nuestro caso, un ID de un Zombie) y devuelve la dirección de la persona que lo posee.


## ERC721: Lógica de transferencia
Fíjese que la especificación de ERC721 tiene 2 formas distintas de poder transferir tokens:

````
function transfer(address _to, uint256 _tokenId) public;
function approve(address _to, uint256 _tokenId) public;
function takeOwnership(uint256 _tokenId) public;
````

La primera forma consiste en que el propietario llame a transfer con la dirección address a la cual quiere enviar, y el _tokenId del token que quiere transferir.

La segunda forma consiste en que el propietario llame primero a approve, y envía la misma información que el caso anterior. Después, el contrato almacena quién está autorizado para tomar un token, generalmente en un mapping (uint256 => address). Entonces, cuando alguien llame a takeOwnership, el contrato comprueba si ese msg.sender está autorizado por el propietario para tomar ese token y si es así, le transfiere el token.

Si te das cuenta, tanto transfer como takeOwnership contienen la misma lógica para la transferencia, sólo que en orden inverso. (En un caso, el emisor del token llama a la función, en el otro el receptor del token lo llama).

Entonces, tiene sentido que abstraigamos esta lógica en su propia función privada, _transfer, que luego es llamada por ambas funciones. De esa forma no repetiremos el mismo código dos veces.

## ERC721: Approve
Recuerda, con approve / takeOwnership, la transferencia se produce en 2 pasos:

Tú, el propietario, llamas a approve y envías la dirección address del nuevo propietario, y el _tokenId que quieres enviar.

El nuevo propietario llama a takeOwnership con el _tokenId, el contrato comprobará que este nuevo usuario está autorizado para ello y si es correcto, le transferirá el token.

Como esto ocurre durante 2 llamadas distintas, necesitamos una estructura de datos para almacenar quién ha sido aprobado para cada cosa, en cada uno de los pasos donde se produce cada llamada a una función


## Previniendo debordamientos
### Mejoras de seguridad en el contrato: Desbordamientos por exceso (Overflows) y por defecto (Underflows)
Vamos a ver una característica de seguridad importante que debe tener en cuenta al escribir contratos inteligentes: Prevención de desbordamientos por exceso (overflow) y por defecto (underflow).

¿Qué es un desbordamiento?

Digamos que tenemos un campo uint8, que sólo puede tener hasta 8 bits. Esto significa, que el número en binario más grande posible sería 11111111 (o en decimal, 2^8 - 1 = 255).

Eche un vistazo al código siguiente. ¿Qué valor tendrá number al final?

````
uint8 number = 255;
number++;
````

En este caso, hemos provocado un desbordamiento por exceso (overflow) — así que number convertirá su valor igual a 0 aunque lo hayamos aumentado. (Si añades +1 al valor binario 11111111, se reiniciará al valor inicial 00000000, igual que un reloj pasa de las 23:59 a las 00:00).

Un desbordamiento por defecto es similar, cuando intentemos restar un 1 a un campo uint8 que tiene valor 0, ahora pasará a valer 255 (porque el tipo uint no tiene signo y por lo tanto, no puede ser negativo).

### Usando SafeMath
Para prevenir esto, OpenZeppelin ha creado una librería llamada SafeMath que previene estos problemas por defecto.

Pero antes de entrar en eso... ¿Qué es una librería?

Una librería es un tipo de contrato especial en Solidity. Una de las cosas para las cuales es útil, es para asociar funciones a tipos de datos nativos.

Por ejemplo, con la librería SafeMath, usaremos la sintaxis using SafeMath for uint256. La librería SafeMath tiene 4 funciones— add, sub, mul y div. Y ahora podemos acceder a estas funciones desde uint256 de la siguiente manera:

````
using SafeMath for uint256;

uint256 a = 5;
uint256 b = a.add(3); // 5 + 3 = 8
uint256 c = a.mul(2); // 5 * 2 = 10
````

Veremos qué hacen estas funciones en el próximo capítulo, pero por ahora agreguemos la librería SafeMath a nuestro contrato.


## SafeMath Part 2
Echemos un vistazo al código detrás de SafeMath:

````
library SafeMath {

  function mul(uint256 a, uint256 b) internal pure returns (uint256) {
    if (a == 0) {
      return 0;
    }
    uint256 c = a * b;
    assert(c / a == b);
    return c;
  }

  function div(uint256 a, uint256 b) internal pure returns (uint256) {
    // assert(b > 0); // Solidity automatically throws when dividing by 0
    uint256 c = a / b;
    // assert(a == b * c + a % b); // There is no case in which this doesn't hold
    return c;
  }

  function sub(uint256 a, uint256 b) internal pure returns (uint256) {
    assert(b <= a);
    return a - b;
  }

  function add(uint256 a, uint256 b) internal pure returns (uint256) {
    uint256 c = a + b;
    assert(c >= a);
    return c;
  }
}
````

Primero, tenemos la palabra clave reservada library— las librerías son similares a los contracts pero con algunas diferencias. Para nuestros propósitos, las librerías nos permiten usar la palabra clave reservada using, que automáticamente asocia a todos los métodos de la librería con unos tipos de datos:

````
using SafeMath for uint;
// now we can use these methods on any uint
uint test = 2;
test = test.mul(3); // test now equals 6
test = test.add(5); // test now equals 11
````

Fíjese que las funciones mul y add requieren dos parámetros de entrada, pero cuando declaramos using SafeMath for uint, el uint al que llamamos en la función (test) se pasa automáticamente como parámetro.

Veamos el código que tiene add para ver qué hace SafeMath:

````
function add(uint256 a, uint256 b) internal pure returns (uint256) {
  uint256 c = a + b;
  assert(c >= a);
  return c;
}
````

Básicamente add suma 2 valores uints como hace el símbolo +, pero también contiene una declaración assert para asegurarse de que la suma sea mayor que a. Esto nos protege de desbordamientos por exceso (overflows).

assert es similar a require, donde lanzará un error si es falso. La diferencia entre assert y require es que require devolverá al usuario el resto del gas cuando la función falle, mientras que assert no lo hará. Por lo tanto, la mayor parte del tiempo deseará utilizar require en su código; assert se usará normalmente sólo cuando algo ha ido terriblemente mal con el código (como un desbordamiento en uint).

Entonces, en pocas palabras, las funciones de SafeMath's de add, sub, mul, ydiv son funciones que realizan las 4 operaciones básicas de matemáticas, pero lanzan un error si ocurre un desbordamiento por exceso o por defecto.

### Usando SafeMath en nuestro código.
Para evitar el desbordamiento, podemos buscar en nuestro código los lugares donde se utilicen +, -, *, o /, y sustituirlos por add, sub, mul, div.

Ejemplo. En lugar de escribir:

````
myUint++;
````

Deberíamos poner:

````
myUint = myUint.add(1);
````


## SafeMath Part 3
````
// Si usamos `.add` con un `uint8`, lo convertirá en `uint256`.
// Por lo tanto no se desbordará, ya que 256 funciona en `uint256`.
````

Esto significa que vamos a tener que implementar 2 librerías más para evitar los casos con uint16 y uint32. Podemos llamarlos SafeMath16 y SafeMath32.

El código será exactamente igual al de SafeMath, excepto que todas las instancias de uint256 serán reemplazadas por uint32 o uint16 .

## Comentarios
Comentar en Solidity es como JavaScript:

````
// Este es un comentario de una sola línea. Es como una nota para uno mismo (o para otros)
````

Simplemente añade // en cualquier lugar y estarás comentando. Es tan fácil que deberías hacerlo todo el tiempo.

Pero te entiendo— a veces una sola línea no es suficiente. ¡Eres escritor, después de todo!

Por lo tanto, también tenemos comentarios de varias líneas:

````
contract CryptoZombies {
  /* Este es un comentario con múltiples líneas. 
      Me gustaría agradecerles a todos vosotros, los que se han tomado su    
      tiempo para probar este curso de programación.
      Sé que es gratis para todos ustedes, y se mantendrá libre
    para siempre, pero aún ponemos nuestro corazón y nuestra alma en hacer
    esto tan bueno como puede ser.

    Sepa que este es todavía el comienzo del desarrollo de Blockchain.
    Hemos llegado muy lejos, pero hay muchas maneras de hacer esto en
    comunidad mejor. Si cometimos un error en alguna parte, puedes
    ayudarnos y abrir una pull request aquí:
    https://github.com/loomnetwork/cryptozombie-lessons

    O si tiene algunas ideas, comentarios o simplemente quiere decirnos hola
    - Únase a nuestra comunidad de Telegram en https://t.me/loomnetwork
  */
}
````

En particular, es una buena práctica comentar su código para explicar el comportamiento esperado de cada función en su contrato. De esta forma, otro desarrollador (¡O tú, después de un paréntesis de 6 meses en un proyecto!) puede leer rápidamente y comprender a un nivel alto lo que hace su código sin tener que leer el código en sí.

El estándar en la comunidad Solidity es usar un formato llamado natspec, que tiene esta apariencia:

````
/// @title Un contrato para operaciones matemáticas básicas
/// @author H4XF13LD MORRIS 💯💯😎💯💯
/// @notice Por ahora, este contrato solo añade una función de multiplicar
contract Math {
  /// @notice Multiplica 2 números juntos
  /// @param x el primer uint.
  /// @param y el segundo uint.
  /// @return z el resultado de (x * y)
  /// @dev Esta función actualmente no verifica desbordamientos
  function multiply(uint x, uint y) returns (uint z) {
    // Este es solo un comentario normal, y no será recogido por natspec
    z = x * y;
  }
}
````

@title y @author son simples.

@notice explica a un usuario lo que hace el contrato o la función. @dev es para explicar detalles adicionales a los desarrolladores.

@param y @return son para describir para qué sirve cada parámetro y el valor de retorno de una función.

Tenga en cuenta que no siempre tiene que usar todas estas etiquetas para cada función — todas las etiquetas son opcionales. Pero al menos, deje una nota @dev explicando lo que hace cada función.