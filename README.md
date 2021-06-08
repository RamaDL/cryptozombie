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