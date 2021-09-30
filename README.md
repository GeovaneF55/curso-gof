## Descrevem soluções para problemas recorrentes no desenvolvimento de sistemas orientado a objetos

- possui um nome
- define um problema
- define a solução
- quando aplicar esta solução
- consequências
- [https://github.com/eimg/design-patterns-php](https://github.com/eimg/design-patterns-php)

## Padrões de Criação ("criacionais") - 5

- aplicam-se em situações que envolvem a criação de objetos
- ajudam a fazer um sistema independente de como seus objetos são criados, compostos e representados

### 1 - Singleton

- Intenção: Garantir que uma determinada classe tenha uma, e somente uma instância, mantendo um ponto global de acesso para a mesma

![singleton.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/58b15b0e-1748-4f74-aa85-91e78f614a75/singleton.png)

```php
<?php

// === Singleton Class ===
// This class doesn't accept creating object directly
// But only accept creating object through make() method
// to make sure there is only one instance of this class
class Single
{
    protected static $me = null;

    protected function __construct()
    {
        $this->info = "I am a single.\n";
    }

    static function make()
    {
        if (!static::$me) {
            static::$me = new static;
        }

        return static::$me;
    }
}

// ---

$single = Single::make();
echo $single->info;
// Output: I'm a single.
```

### 2 - Factory Method

- Intenção: definir uma interface para criar um objeto, mas deixar as subclasses decidirem que classe instanciar.
- cria uma instância de várias classes derivadas.

![factory_method.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fc3dde3a-e351-4565-812b-7ab23f42ba2f/factory_method.png)

```php
<?php

class CarOptions
{
    var $color = "White";
    var $doors = 4;
}

class Car
{
    private $color;
    private $doors;

    public function __construct(CarOptions $options)
    {
        $this->color = $options->color;
        $this->doors = $options->doors;
    }

    public function info()
    {
        return "This is a $this->doors doors $this->color car.\n";
    }
}

// === CarFactory that create Car object for us ===
// We can create Car object directly using Car class
// But, by using this factory, we can skip the detail
class CarFactory
{
    private $options;

    public function __construct($color, $doors)
    {
        $options = new CarOptions();
        $options->color = $color;
        $options->doors = $doors;

        $this->options = $options;
    }

    public function build()
    {
        return new Car($this->options);
    }
}

// ---

$factory = new CarFactory("Red", 2);
$car = $factory->build();

echo $car->info();
// Output: This is a 2 doors Red car.
```

### 3 - Abstract Factory

- Intenção: fornecer uma interface para criação de famílias de objetos relacionados ou dependentes sem especificar suas classes concretas
- cria uma instância de várias famílias de classes

![Abstract_Factory.svg](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/954796d5-4d2c-43db-9da9-dd85b9ec6133/Abstract_Factory.svg)

### 4 - Builder

- Intenção: separar a construção de um objeto complexo da sua representação de modo que o mesmo processo de construção possa criar diferentes representações
- separa a construção do objeto de sua representação

![Builder.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/33448bff-9727-45e3-8dca-5ff42495ffcd/Builder.png)

```php
<?php

// === CarBuilder that create Car object for us ===
// But instead of providing all properties on creation
// we can set them one-by-one through setter methods
class CarBuilder
{
    private $color;
    private $doors;

    public function setColor($color)
    {
        $this->color = $color;
        return $this;
    }

    public function setDoors($doors)
    {
        $this->doors = $doors;
        return $this;
    }

    public function getColor()
    {
        return $this->color;
    }

    public function getDoors()
    {
        return $this->doors;
    }

    function build()
    {
        return new Car($this);
    }
}

class Car
{
    public $color;
    public $doors;

    public function __construct(CarBuilder $cb)
    {
        $this->color = $cb->getColor();
        $this->doors = $cb->getDoors();
    }

    static function builder()
    {
        return new CarBuilder();
    }
}

// ---

$car = Car::builder()->setColor("Silver")->setDoors(5)->build();

print_r($car);
// Output: Car Object([color] => Silver, [doors] => 5)
```

### 5 - Prototype

- Instância: especificar os tipos de objetos a serem criados usando uma instância protótipo e criar novos objetos pela cópia deste protótipo
- Uma instância inicializada a ser copiada ou clonada

![Prototype.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6ad28c72-9f7c-4edc-bd2b-9df8e8699039/Prototype.png)

```php
<?php

class Car
{
    public $color = "White";
    public $doors = 4;

    // This magic method automatically invoke when attempting
    // to copy instance (object) of this class with `clone`
    public function __clone()
    {
        echo "Cloning Car...\n";
    }
}

// ---

$car = new Car();

// Creating new object by cloning
$car2 = clone $car;

print_r($car2);
// Output:
// Cloning Car...
// Car Object([color] => White, [doors] => 4)
```

## Padrões de Estrutura ("estruturais") - 7

- de que maneira as classe e objetos são compostos para a formação de grandes estruturas

### 1 - Bridge

- Intenção: desacoplar uma abstração da sua implementação, de modo que as duas possam variar independentemente
- separa a interface do objeto de sua implementação

![Bridge.svg](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f6e03992-2929-4cc7-b8de-e3ac8bd32374/Bridge.svg)

```php
<?php

interface Engine
{
    public function run();
}

class Diesel implements Engine
{
    public function run()
    {
        echo "Broom! Broom!\n";
    }
}

class Petrol implements Engine
{
    public function run()
    {
        echo "Wroom! Wroom!\n";
    }
}

// === EngineBridge ===
// That help us switch engine type
// at run-time through set() method
class EngineBridge
{
    public $engine;

    public function __construct(Engine $engine)
    {
        $this->engine = $engine;
    }

    public function set(Engine $engine)
    {
        $this->engine = $engine;
    }
}

// ---

$diesel = new Diesel();
$petrol = new Petrol();

$bridge = new EngineBridge($diesel);
$bridge->engine->run();
// Output:
// Broom! Broom!

$bridge->set($petrol);
$bridge->engine->run();
// Output:
// Wroom! Wroom!
```

### 2 - Adapter

- Intenção: converter a interface de uma classe em outra interface, esperada pelos clientes. Permite que classes com interfaces incompatíveis trabalhem em conjunto o que, de outra forma, seria impossível
- Equiparar interfaces de diferentes classes.

![Adapter.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/17e34441-e94b-44f9-b822-221b039c0f93/Adapter.png)

```php
<?php

interface TwoPins
{
    public function one();
    public function two();
}

interface ThreePins
{
    public function one();
    public function two();
    public function three();
}

class TwoPinsSocket implements TwoPins
{
    public function one()
    {
        echo "Pin one exists.\n";
    }

    public function two()
    {
        echo "Pin two exists.\n";
    }
}

class ThreePinsSocket implements ThreePins
{
    public function one()
    {
        echo "Pin one exists.\n";
    }

    public function two()
    {
        echo "Pin two exists.\n";
    }

    public function three()
    {
        echo "Pin three exists.\n";
    }
}

// === Adapter  ===
// which convert TwoPins object to ThreePins
class ThreePinsAdapter implements ThreePins
{
    private $socket;

    public function __construct(TwoPins $socket)
    {
        $this->socket = $socket;
    }

    public function one()
    {
        $this->socket->one();
    }

    public function two()
    {
        $this->socket->two();
    }

    public function three()
    {
        echo "Pin three doesn't exists, but it's OK.\n";
    }
}

class Lamp
{
    private $socket;

    public function __construct(ThreePins $socket)
    {
        $this->socket = $socket;
    }

    public function turnOn() {
        $this->socket->one();
        $this->socket->two();
        $this->socket->three();
    }
}

// ---

$threepins = new ThreePinsSocket();
$twopins = new ThreePinsAdapter(new TwoPinsSocket());

$lamp1 = new Lamp($threepins);
$lamp1->turnOn();
// Output:
// Pin one exists.
// Pin two exists.
// Pin three exists.

$lamp2 = new Lamp($twopins);
$lamp2->turnOn();
// Output:
// Pin one exists.
// Pin two exists.
// Pin three doesn't exists, but it's OK.
```

### 3 - Proxy

- Intenção: fornece um substituto ou marcador da localização de outro objeto para controlar o acesso ao mesmo
- Um objeto representando um objeto

![proxy.gif](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e58b0ea1-6165-43d4-bb56-bec868c0344e/proxy.gif)

```php
<?php

class Car
{
    public function __construct($car)
    {
        echo "The $car has started and ready.\n";
    }
}

// === CarProxy that forward to Car class ===
// We can have many Car objects but actually instantiated
// only when necessary with start() method (a.k.a lazy-load)
class CarProxy
{
    private $car;

    public function __construct($car)
    {
        $this->car = $car;
    }

    public function start()
    {
        new Car( $this->car );
    }
}

// ---

$cars = [ new CarProxy("Fit"), new CarProxy("Vitz") ];
$cars[1]->start();
// Output: The Vitz has started and ready.
```

### 4 - Decorator

- Intenção: agregar dinamicamente responsabilidades adicionais a um objeto

![decorator.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d82b12e2-7e96-4e9a-97d5-30a034a092b3/decorator.png)

```php
<?php

interface CarInterface
{
    public function setup();
}

class Car implements CarInterface
{
    public $features;

    public function setup()
    {
        $this->features = "Alloy Wheel";
    }
}

abstract class CarDecorator implements CarInterface
{
    protected $car;

    public function __construct(CarInterface $car)
    {
        $this->car = $car;
    }

    abstract public function setup();
}

// === A CarDecorator that can add ===
// additional feature to existing Car object
class SunroofDecorator extends CarDecorator
{
    public function setup()
    {
        $this->car->setup();
        $this->car->features .= ", Sunroof";
        $this->features = $this->car->features;
    }
}

// === A CarDecorator that can add ===
// additional feature to existing Car object
class SpoilerDecorator extends CarDecorator
{
    public function setup()
    {
        $this->car->setup();
        $this->car->features .= ", Spoiler";
        $this->features = $this->car->features;
    }
}

// ---

$car = new Car();
$car = new SunroofDecorator($car);
$car = new SpoilerDecorator($car);
$car->setup();

echo $car->features;
// Output: Alloy Wheel, Sunroof, Spoiler 
```

### 5 - Composite

- Intenção: compor objetos em estruturas de árvore para representarem hierarquias todo-parte

![composite.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/516299f2-289b-4bcf-806b-63a90a488737/composite.png)

```php
<?php

// === Component ===
interface Honk
{
    public function honk();
}

// === Leaf ===
class Car implements Honk
{
    private $name;

    public function __construct($name)
    {
        $this->name = $name;
    }

    public function honk()
    {
        echo "$this->name : Honk, honk...\n";
    }
}

// === Composit ===
// Both Leaf and Composit come from same Component
// So, both act the same way.
class CarList implements Honk
{
    public $list = [];

    public function add(Car $car)
    {
        $this->list[] = $car;
    }

    public function honk()
    {
        foreach($this->list as $car) {
            $car->honk();
        }
    }
}

// ---

$fit = new Car("Fit");
$vitz = new Car("Vitz");

$list = new CarList();
$list->add($fit);
$list->add($vitz);

$list->honk();
// Output:
// Fit : Honk, honk...
// Vitz : Honk, honk...
```

### 6 - Facade

- Intenção: fornecer uma interface unificada para um conjunto de interfaces em um subsistema. Define uma interface de nível mais alto que torna o subsistema mais fácil de ser usado
- Uma única classe representa um subsistema inteiro

![facade.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9c2b4058-ae3e-4fae-860d-13422d33914d/facade.png)

```php
<?php

class CheckOilPressure
{
    public function check()
    {
        echo "Oil Pressure OK.\n";
    }
}

class CheckFuel
{
    public function check()
    {
        echo "Fuel Status OK.\n";
    }
}

class CheckBreakFluid
{
    public function check()
    {
        echo "Break Fluid OK.\n";
    }
}

// === Car Facede ===
// That provide simple interface (wrapper) to complex steps
// To start a car engine, the all we need is start() method
class Car
{
    public $oil;
    public $fuel;
    public $break;

    public function __construct()
    {
        $this->oil = new CheckOilPressure();
        $this->fuel = new CheckFuel();
        $this->break = new CheckBreakFluid();
    }

    public function start()
    {
        $this->oil->check();
        $this->fuel->check();
        $this->break->check();

        echo "Car Engine Started.\n";
    }
}

// ---

$car = new Car();
$car->start();

// Output:
// Oil Pressure OK.
// Fuel Status OK.
// Break Fluid OK.
// Car Engine Started.
```

### 7- Flyweight

- Intenção: usar compartilhamento para suportar eficientemente grande quantidades de objetos de granularidade fina

![flyweight.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c2cbe5c3-124a-460a-9202-1745e0b8d843/flyweight.png)

```php
<?php

class Car
{
    public $name;

    public function __construct($name)
    {
        $this->name = $name;
    }
}

// === A Flyweight ===
// That cache objects and re-use when possible
// And create new object only when necessary
class CarFlyweightFactory
{
    private $cars = [];

    public function make($name)
    {
        if (!isset($this->cars[$name])) {
            $this->cars[$name] = new Car($name);
        }

        return $this->cars[$name];
    }
}

// ---

$factory = new CarFlyweightFactory();
$fit = $factory->make("Fit");

echo $fit->name;
// Output: Fit
```

## Padrões de Comportamento ("comportamentais") - 11

- preocupam-se com algoritmos e atribuição de responsabilidades entre objetos. Descrevem, também, um padrão de comunicação entre classes ou objetos

### 1 - Command

- Intenção: encapsular uma solicitação como um objeto, desta forma permitindo parametrizar clientes com diferente solicitações, enfileirar ou fazer o registro (log) de solicitações e suportar operações que podem ser desfeitas.
- Encapsular comandos como um objeto

![command.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/51cbcb49-a05e-405c-aa03-4684dc80615d/command.png)

```php
<?php

class Lamp
{
    public function open()
    {
        echo "Lamp is opened.\n";
    }

    public function close()
    {
        echo "Lamp is closed.\n";
    }
}

interface Button
{
    public function execute();
}

// === A command that can be executed on Lamp ===
class SwitchUp implements Button
{
    public $lamp;

    public function __construct(Lamp $lamp)
    {
        $this->lamp = $lamp;
    }

    public function execute()
    {
        $this->lamp->open();
    }
}

// === A command that can be executed on Lamp ===
class SwitchDown implements Button
{
    public $lamp;

    public function __construct(Lamp $lamp)
    {
        $this->lamp = $lamp;
    }

    public function execute()
    {
        $this->lamp->close();
    }
}

// === A Collection ===
// That accept and execute commands
class Commands
{
    public $commands = [];

    public function add(Button $button)
    {
        $commands[] = $button;
        $button->execute();
    }
}

// ---

$lamp = new Lamp();
$up = new SwitchUp($lamp);
$down = new SwitchDown($lamp);

$commands = new Commands();
$commands->add($up);
// Output: Lamp is opened.

$commands->add($down);
// Output: Lamp is closed.
```

### 2 - Strategy

- Intenção: definir uma família de algoritmos, encapsular cada uma delas e torná-las intercambiáveis. Permite que o algoritmo varie independentemente dos clientes que o utilizam.
- Encapsular algoritmos ("estratégias") como um objeto.

![strategy.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/70ee4a84-fce2-4da3-80be-0c3ca257d904/strategy.png)

```php
<?php

interface Car
{
    public function pick();
}

class Fit implements Car
{
    public function pick()
    {
        echo "Picking Fit for today.\n";
    }
}

class Vitz implements Car
{
    public function pick()
    {
        echo "Picking Vitz for today.\n";
    }
}

// === CarPickerStrategy ===
// That decide which car object to use based on the situation
// The benefit is that we can add more strategies later,
// clients doesn't have to know or change their implementation
class CarPickerStrategy
{
    public $today;

    public function __construct($today)
    {
        $this->today = $today;
    }

    public function pick()
    {
        if( $this->today == "Monday" ) {
            $car = new Vitz();
        } else {
            $car = new Fit();
        }

        $car->pick();
    }
}

// ---

$carpicker = new CarPickerStrategy("Sunday");
$carpicker->pick();
// Output: Picking Fit for today.

$carpicker->today = "Monday";
$carpicker->pick();
// Output: Picking Vitz for today.
```

### 3 - State

- Intenção: permite a um objeto alterar seu comportamento quando o seu estado interno muda. O objeto parecerá ter mudado sua classe.
- Alterar o comportamento de um objeto quando seu estado muda.

![state.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3ce3e8b9-cbfa-4af6-ac4a-b660935737ac/state.png)

```php
<?php

// === Available states ===
interface StateInterface
{
    public function drive();
    public function stop();
}

// === All state changes are denied by default ===
abstract class CarStates implements StateInterface
{
    public function drive()
    {
        echo "You are not allow to drive.\n"; exit();
    }

    public function stop()
    {
        echo "You are not allow to stop.\n"; exit();
    }
}

// === DriveState is allow to stop ===
// But not allow to drive (by default)
class DriveState extends CarStates
{
    public function stop()
    {
        echo "The car has stopped.\n";
        return new StopState();
    }
}

// === StopState is allow to drive ===
// But not allow to stop (by default)
class StopState extends CarStates
{
    public function drive()
    {
        echo "The car is driving now.\n";
        return new DriveState();
    }
}

class Car
{
    private $state;

    public function __construct(StateInterface $state)
    {
        $this->state = $state;
    }

    public function drive()
    {
        $this->setState($this->state->drive());
    }

    public function stop()
    {
        $this->setState($this->state->stop());
    }

    private function setState(StateInterface $state)
    {
        $this->state = $state;
    }
}

// ---

// The car is initially stop
$car = new Car(new StopState());

// So, it's allow to drive
$car->drive();
// Output: The car is driving now.

// Now it's driving, so,
// it's allow to stop once again
$car->stop();
// Output: The car is stopped.

// Now it has stopped, so,
// not allow to stop again
$car->stop();
// Output: You are not allow to stop.
```

### 4 - Observer

- Intenção: definir uma dependência um-para-muitos entre objetos, de maneira que quando um objeto muda de estado todos os seus dependentes são notificados e atualizados automaticamente.

![observer.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/14b20a0d-3cae-4925-b747-5df8bde2cd8a/observer.png)

```php
<?php

// === Subject ===
// Have a observer list. When there is a change,
// iterate through observer list and invoke update()
// on each observer.
class Car
{
    protected $phones = []; // observer list
    protected $status;

    public function attach(Phone $phone)
    {
        $this->phones[] = $phone;
    }

    public function getStatus()
    {
        return $this->status;
    }

    public function setStatus($status)
    {
        $this->status = $status;
        $this->notify();
    }

    public function notify() {
        foreach($this->phones as $phone) {
            $phone->update();
        }
    }
}

// === Observer ===
// Which add itself to observer-list of subject,
// by invoking attach() method of the subject.
class Phone
{
    private $car;
    public $name;

    public function __construct(Car $car, $name)
    {
        $this->name = $name;
        $this->car = $car;
        $this->car->attach($this);
    }

    public function update()
    {
        echo $this->name . " - Car is " . $this->car->getStatus() . "\n";
    }
}

// ---

$car = new Car();
$iphone = new Phone( $car, "iPhone" );
$nexus = new Phone( $car, "Nexus" );

$car->setStatus("driving");
// Output:
// iPhone - Car is driving
// Nexus - Car is driving
```

### 5 - Mediator

- Intenção: definir um objeto que encapsula a forma como um conjunto de objetos interage. Promove o acoplamento fraco ao evitar que os objetos se refiram uns aos outros explicitamente e permite variar suas interações independentemente.
- Definir uma comunicação simplificada entre as classes

![mediator.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0656e430-e1a3-4917-bb3c-ec65752e4ebb/mediator.png)

```php
<?php

interface Filter
{
    public function check();
}

// === A Colleague ===
class OilFilter implements Filter
{
     public function check()
     {
         echo "Checking Oil Filter... OK\n";
     }
}

// === A Colleague ===
class AirFilter implements Filter
{
     public function check()
     {
         echo "Checking Air Filter... OK\n";
     }
}

// === A Mediator ===
// Collection of colleagues designed to make
// all those colleague work together
class Car
{
    protected $filters;

    public function addFilter(Filter $filter)
    {
        $this->filters[] = $filter;
    }

    public function check()
    {
        foreach ($this->filters as $filter) {
            $filter->check();
        }
    }
}

$car = new Car();
$car->addFilter( new OilFilter() );
$car->addFilter( new AirFilter() );
$car->check();

// Output:
// Checking Oil Filter... OK
// Checking Air Filter... OK
```

### 6 - Chain of Responsability

- Intenção: evitar o acoplamento do remetente de uma solicitação ao seu receptor, ao dar a mais de um objeto a oportunidade de tratar a solicitação. Encadear os objetos receptores, passando a solicitação ao longo da cadeia até que um objeto a trate.
- Uma maneira de passar uma requisição entre uma cadeia de objetos.

![chain.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d2de50c7-2d49-4621-999a-0fad0c2c290b/chain.png)

```php
<?php

// === CarStatus ===
// Can set next object in line with setNext()
// Can run next object if exists with runNext()
abstract class CarStatus
{
    protected $next;

    public function setNext(CarStatus $next)
    {
        $this->next = $next;
    }

    public function runNext(Car $car)
    {
        if($this->next) {
            $this->next->check($car);
        }
    }

    abstract public function check(Car $car);
}

// A class that inhirit CarStatus and call runNext()
// to invoke similar function on next object in line
class OilPressure extends CarStatus
{
    public function check(Car $car)
    {
        if( $car->oil != "OK" ) {
            echo "Oil Pressure is not OK.\n";
        } else {
            echo "Oil Pressure is OK.\n";
        }

        $this->runNext($car);
    }
}

// A class that inhirit CarStatus and call runNext()
// to invoke similar function on next object in line
class FuelLevel extends CarStatus
{
    public function check(Car $car)
    {
        if( $car->fuel != "OK" ) {
            echo "Fuel Level is not OK.\n";
        } else {
            echo "Fuel Level is OK.\n";
        }

        $this->runNext($car);
    }
}

// A class that inhirit CarStatus and call runNext()
// to invoke similar function on next object in line
class BreakFluid extends CarStatus
{
    public function check(Car $car)
    {
        if( $car->break != "OK" ) {
            echo "Break Fluid is not OK.\n";
        } else {
            echo "Break Fluid is OK.\n";
        }

        $this->runNext($car);
    }
}

class Car
{
    public $oil = "OK";
    public $fuel;
    public $break = "OK";
}

// ---

$oil = new OilPressure();
$fuel = new FuelLevel();
$break = new BreakFluid();

// Setting next objects in line
$oil->setNext($fuel);
$fuel->setNext($break);

$oil->check(new Car());
// Output:
// Oil Pressure is OK.
// Fuel Level is not OK.
// Break Fluid is OK.
```

### 7 - Template

- Intenção: definir o esqueleto de um algoritmo em uma operação, postergando alguns passos para subclasses. Permite que subclasses redefinam certos passos de um algoritmo sem mudar a estrutura do mesmo.

![template.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/842a95d2-78fa-41c1-85c8-5532648cdc95/template.png)

```php
<?php

// === A Template for all Car ===
abstract class Car
{
    public $color = "White";
    public $doors = 4;

    public function __construct($name)
    {
        $this->name = $name;
    }

    public function loadPassenger()
    {
        echo "The $this->name is loading passengers...\n";
    }
}

class Minivan extends Car
{
    public $doors = 5;

    public function loadGrocery()
    {
        echo "The $this->name is loading grocery...\n";
    }
}

class SportCar extends Car
{
    public $doors = 2;
    public $color = "Red";

    public function loadPassenger()
    {
        echo "The $this->name is loading a buddy...\n";
    }
}

// ---

$fit = new Minivan("Fit");
$fit->loadPassenger();
// Output: The Fit is loading passengers...

$evo = new SportCar("Evo");
$evo->loadPassenger();
// Output: The Evo is loading a buddy...
```

### 8 - Interpreter

- Intenção: dada uma linguagem, definir uma representação para a sua gramática juntamente com um interpretador que usa representação para interpretar sentenças da linguagem.
- Uma forma de incluir elementos da linguagem em um programa.
- Usa classes para representar cada regra de uma gramática.

![interpreter.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/026d5bc1-26ab-41af-9b57-754b432db32d/interpreter.png)

```php
<?php

interface Converter
{
    public function show();
}

// === A Simple Converter ===
// That convert Gallon to Litre
class GallonToLitre implements Converter
{
    private $gallon;

    public function __construct($gallon)
    {
        $this->gallon = $gallon;
    }

    public function show()
    {
        return round($this->gallon * 3.79);
    }
}

// === A Simple Converter ===
// That convert Mile to Kilometer
class MileToKilometer implements Converter
{
    private $mile;

    public function __construct($mile)
    {
        $this->mile = $mile;
    }

    public function show()
    {
        return round($this->mile * 1.6);
    }
}

// === An Intepreter ===
// That interpret to Kilometer per Litre by using existing converters.
// Consider this as structuring a sentence using grammers (converters).
class MpgToKml implements Converter
{
    private $g2l;
    private $m2k;

    public function __construct(Converter $g2l, Converter $m2k)
    {
        $this->g2l = $g2l;
        $this->m2k = $m2k;
    }

    public function show()
    {
        echo  $this->g2l->show() . "l/" . $this->m2k->show() . "k\n";
    }
}

// ---

$mpg2kml = new MpgToKml(new GallonToLitre(1), new MileToKilometer(20));
$mpg2kml->show();
// Output: 4l/32k
```

### 9 - Memento

- Intenção: Sem violar o encapsulamento, capturar e externalizar um estado interno de um objeto, de maneira que o objeto possa ser restaurado para este estado mais tarde.

![memento.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/522991e2-d3f9-473f-bff0-d6508ea7f917/memento.png)

```php
<?php

// === A Memento ===
// That store backup copy of a Car object
class Memento
{
    private $car;

    public function __construct(Car $car)
    {
        $this->car = $car;
    }

    public function getCar()
    {
        return $this->car;
    }
}

class Car
{
    public $color;

    public function __construct($color)
    {
        $this->color = $color;
    }
}

class Customizer
{
    private $car;

    public function __construct(Car $car)
    {
        $this->car = $car;
    }

    // Store backup copy in Memento
    public function copy()
    {
        return new Memento( clone $this->car );
    }

    // Retrive backup from Memento
    public function restore(Memento $memento)
    {
        $this->car = $memento->getCar();
    }

    public function changeColor($color)
    {
        $this->car->color = $color;
    }

    public function getColor()
    {
        return $this->car->color;
    }
}

// ---

$custom = new Customizer( new Car("White") );
echo $custom->getColor() . "\n";
// Output: White

$backup = $custom->copy();

$custom->changeColor("Black");
echo $custom->getColor() . "\n";
// Output: Black

$custom->restore($backup);
echo $custom->getColor() . "\n";
// Output: White
```

### 10 - Iterator

- Intenção: fornecer um meio de acessar, sequencialmente, os elementos de um objeto agregado sem expor a sua representação.

![iterator.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7b160b39-7323-4590-b93c-3242ff50464c/iterator.png)

```php
<?php

// === PHP has build-in Iterator interface ===
// That enforce iteration methods such as current(),
// next(), rewind(), valid() on given collection
class CarIterator implements Iterator
{
    private $position = 0;
    private $cars;

    public function __construct(CarCollection $cars)
    {
        $this->cars = $cars;
    }

    public function current()
    {
        return $this->cars->getName($this->position);
    }

    public function key()
    {
        return $this->position;
    }

    public function next()
    {
        $this->position++;
    }

    public function rewind()
    {
        $this->position = 0;
    }

    public function valid()
    {
        return !is_null($this->cars->getName($this->position));
    }
}

// === PHP has build-in IteratorAggregate interface ===
// That enforce getIterator() method to work
// in conjunction with Iterator interface
class CarCollection implements IteratorAggregate
{
    private $names = [];

    public function getIterator()
    {
        return new CarIterator($this);
    }

    public function setName($string)
    {
        $this->names[] = $string;
    }

    public function getName($key)
    {
        if (isset($this->names[$key])) {
            return $this->names[$key];
        }

        return null;
    }

    public function is_empty()
    {
        return empty($this->$names);
    }
}

// ---

$cars = new CarCollection();

$cars->setName('Fit');
$cars->setName('Vitz');
$cars->setName('Swift');

foreach($cars as $car){
    echo $car . "\n";
}

// Output:
// Fit
// Vitz
// Swift
```

### 11 - Visitor

- Intenção: representar uma operação a ser executada nos elementos de uma estrutura de objetos. Permite definir uma nova operação sem mudar as classes dos elementos entre os quais opera.
- Define uma nova operação a uma classe sem alterá-la.

![visitor.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/91ac2621-8e76-4836-a6ff-0dfb7ebecb44/visitor.png)

```php
<?php

class Car
{
    public $color = "Silver";

    // accepting Enhancer as visitor
    public function visitor(Enhancer $enhancer)
    {
        $this->color = $enhancer->enhance( $this );
    }
}

// === A visitor ===
// That can visit into Car object change it
// The benefit is, Car object is now seperated from
// detail algorithm of how it can be enhanced
class Enhancer
{
    // visiting into Car
    public function enhance(Car $car)
    {
        return "Matelic $car->color";
    }
}

// ---

$car = new Car();
echo $car->color . "\n";
// Output: Silver

$car->visitor(new Enhancer());
echo $car->color . "\n";
// Output: Matelic Silver
```