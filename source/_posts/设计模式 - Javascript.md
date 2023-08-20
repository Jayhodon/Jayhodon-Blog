---
title: 设计模式 - Javascript
date: 2023-07-18 14:07:56
tags: [前端,Javascript]
categories: [前端,Javascript]
summary: '该博文是js的内功进阶修炼 - JavaScript设计模式'
---



> 设计模式（Design pattern），在项目开发过程中面临的一般问题的解决方案，包括JS常见设计模式以及应用场景，适合有JS基础的同学进阶学习。



### 创建型模式（Creational Patterns）

​	

#### 单例模式 (Singleton Pattern)



> **简介：** 单例模式是一种创建型设计模式，确保一个类只有一个实例，并提供一个全局访问点来获取该实例。这在需要控制资源访问、配置管理以及避免重复创建对象时非常有用。

​	

**示例代码：**

```javascript
class Singleton {
  constructor() {
    if (!Singleton.instance) {
      // 在这里可以进行一些初始化操作
      this.data = [];
      Singleton.instance = this;
    }
    return Singleton.instance;
  }

  addData(item) {
    this.data.push(item);
  }

  getData() {
    return this.data;
  }
}

// 使用单例
const instance1 = new Singleton();
instance1.addData('Item 1');

const instance2 = new Singleton();
instance2.addData('Item 2');

console.log(instance1.getData()); // 输出: ["Item 1", "Item 2"]
console.log(instance1 === instance2); // 输出: true，它们是同一个实例

```

​	

> **代码解读：**
>
> - 我们定义了一个名为 `Singleton` 的类，构造函数中检查是否已经存在实例，如果不存在则进行初始化并保存实例。
> - `addData` 和 `getData` 方法用于操作和获取数据。
> - 使用 `new` 操作符创建对象实例时，始终返回同一个实例。

​	

> **用途：** 
>
> 1. **全局资源管理：** 单例模式常用于全局配置、日志记录、数据库连接池等场景中，确保只有一个实例用于管理全局资源。
> 2. **延迟加载：** 单例模式可以用于延迟对象的创建，直到首次使用时才创建对象，节省资源。
> 3. **避免竞态条件：** 在多线程环境中，单例模式可以用于避免竞态条件，确保只有一个实例被创建。



​	单例模式在需要确保只有一个实例存在的情况下非常有用，同时提供了一个全局访问点以方便访问该实例。

​	

#### 工厂模式 (Factory Pattern)




> **简介：** 工厂模式是一种创建型设计模式，用于封装对象的创建过程，以便根据需要创建不同类型的对象。它将对象的创建与使用分离，使代码更具灵活性和可维护性。

​	

**示例代码：**

```javascript
// 定义产品类
class Product {
  constructor(name) {
    this.name = name;
  }

  getDescription() {
    return `This is a ${this.name}.`;
  }
}

// 定义工厂类
class ProductFactory {
  createProduct(name) {
    return new Product(name);
  }
}

// 使用工厂创建不同类型的产品
const factory = new ProductFactory();
const product1 = factory.createProduct('Widget');
const product2 = factory.createProduct('Gadget');

console.log(product1.getDescription()); // 输出: "This is a Widget."
console.log(product2.getDescription()); // 输出: "This is a Gadget."

```
​	



> **代码解读：**
> 
> - 我们定义了一个产品类 `Product`，它有一个构造函数和一个 `getDescription` 方法。
> - 工厂类 `ProductFactory` 包含一个 `createProduct` 方法，用于创建产品对象。
> - 使用工厂类创建不同类型的产品对象，客户端代码只需要指定产品的名称，而不需要知道产品的创建细节。

​	

> **用途：**
> 
> 1. **隐藏对象创建细节：** 工厂模式允许隐藏对象的具体创建方式和初始化过程，使客户端代码不需要关心对象的创建细节。
> 2. **统一接口：** 工厂模式提供了一个统一的接口，用于创建不同类型的对象，使客户端代码可以通过相同的方式创建不同对象。
> 3. **降低耦合性：** 客户端代码与具体对象的创建过程解耦，使代码更容易维护和扩展。如果需要更改对象的创建方式，只需修改工厂类而不影响客户端代码。
> 4. **对象池管理：** 工厂模式可以用于创建和管理对象池，以提高性能和资源利用率。例如，数据库连接池可以使用工厂模式来管理连接对象。


   工厂模式通常在需要创建多个具有相似属性和方法的对象实例时使用，以减少重复的代码并提高代码的可维护性。


​	
#### 抽象工厂模式 (Abstract Factory Pattern)




> **简介：** 抽象工厂模式是一种创建型设计模式，用于创建一组相关或相互依赖的对象，而无需指定其具体类。它提供一个接口，用于创建产品的家族，而不需要知道具体产品的类。

​	

**示例代码：**

```javascript
// 抽象工厂接口
class AbstractFactory {
  createProductA() {}
  createProductB() {}
}

// 具体工厂类
class ConcreteFactory1 extends AbstractFactory {
  createProductA() {
    return new ProductA1();
  }
  createProductB() {
    return new ProductB1();
  }
}

class ConcreteFactory2 extends AbstractFactory {
  createProductA() {
    return new ProductA2();
  }
  createProductB() {
    return new ProductB2();
  }
}

// 抽象产品类
class AbstractProductA {
  operationA() {}
}

class AbstractProductB {
  operationB() {}
}

// 具体产品类
class ProductA1 extends AbstractProductA {
  operationA() {
    return 'Operation A1';
  }
}

class ProductB1 extends AbstractProductB {
  operationB() {
    return 'Operation B1';
  }
}

class ProductA2 extends AbstractProductA {
  operationA() {
    return 'Operation A2';
  }
}

class ProductB2 extends AbstractProductB {
  operationB() {
    return 'Operation B2';
  }
}

// 使用抽象工厂创建产品
function clientCode(factory) {
  const productA = factory.createProductA();
  const productB = factory.createProductB();

  console.log(productA.operationA()); // 输出: "Operation A1" 或 "Operation A2"
  console.log(productB.operationB()); // 输出: "Operation B1" 或 "Operation B2"
}

// 使用具体工厂1
const factory1 = new ConcreteFactory1();
clientCode(factory1);

// 使用具体工厂2
const factory2 = new ConcreteFactory2();
clientCode(factory2);

```

​	

> **代码解读：**
> 
> - 我们定义了一个抽象工厂接口 `AbstractFactory`，它包含两个方法 `createProductA()` 和 `createProductB()`。
> - 有两个具体工厂类 `ConcreteFactory1` 和 `ConcreteFactory2`，它们分别实现了抽象工厂接口，负责创建一组相关的产品。
> - 抽象产品类 `AbstractProductA` 和 `AbstractProductB` 定义了产品的接口。
> - 具体产品类 `ProductA1`、`ProductB1`、`ProductA2` 和 `ProductB2` 实现了抽象产品类，提供具体的产品。

​	

**用途：**

> 1. **创建一组相关对象：** 抽象工厂模式用于创建一组相关的产品，例如创建不同风格的用户界面组件，如按钮、文本框、和下拉框。
> 2. **确保产品一致性：** 它确保创建的产品是一致的，因为每个具体工厂都负责创建一组产品，这些产品之间具有一致的风格和行为。
> 3. **隐藏对象创建细节：** 抽象工厂模式隐藏了具体对象的创建细节，客户端代码只需要与抽象工厂接口交互，而不需要知道具体的产品类。
> 4. **支持产品族扩展：** 可以轻松扩展产品族，只需创建新的具体工厂类，无需修改现有客户端代码。



​	抽象工厂模式通常在需要创建一组相关产品，同时保持这些产品一致性的情况下使用。

​	

#### 建造者模式 (Builder Pattern)



> ​	**简介：** 建造者模式是一种创建型设计模式，用于分步构建一个复杂对象。它将对象的构建过程与其表示分离，允许同一构建过程创建不同的表示。

​	

```javascript
// 产品类
class Product {
  constructor() {
    this.parts = [];
  }

  addPart(part) {
    this.parts.push(part);
  }

  listParts() {
    return this.parts.join(', ');
  }
}

// 抽象建造者
class Builder {
  buildPartA() {}
  buildPartB() {}
  getResult() {}
}

// 具体建造者
class ConcreteBuilder extends Builder {
  constructor() {
    super();
    this.product = new Product();
  }

  buildPartA() {
    this.product.addPart('Part A');
  }

  buildPartB() {
    this.product.addPart('Part B');
  }

  getResult() {
    return this.product;
  }
}

// 指挥者
class Director {
  constructor(builder) {
    this.builder = builder;
  }

  construct() {
    this.builder.buildPartA();
    this.builder.buildPartB();
  }
}

// 使用建造者构建产品
const builder = new ConcreteBuilder();
const director = new Director(builder);

director.construct();
const product = builder.getResult();

console.log('Product Parts:', product.listParts()); // 输出: "Product Parts: Part A, Part B"

```

​	

> **代码解读：**
> 
> - 我们首先定义了一个产品类 `Product`，它包含一个数组用于存储部件。
> - 抽象建造者 `Builder` 定义了构建产品的步骤，包括 `buildPartA` 和 `buildPartB` 方法，以及获取最终产品的 `getResult` 方法。
> - 具体建造者 `ConcreteBuilder` 继承了抽象建造者，并实现了具体的构建步骤。
> - 指挥者 `Director` 接受一个建造者，并通过调用建造者的方法来构建产品。
> - 最后，我们使用具体建造者来构建产品，并获取最终产品。

​	

> **用途：** 
> 
> 1. **构建复杂对象：** 建造者模式用于创建复杂对象，其中构建过程包含多个步骤，不同步骤可以生成不同的表示。
> 2. **分离构建和表示：** 它允许将对象的构建过程与其表示分离，从而使代码更加灵活，可以使用不同的构建过程来创建不同的对象。
> 3. **避免重叠构造器：** 当有多个构造器参数时，可以使用建造者模式来避免创建多个重叠的构造器。



​	建造者模式常用于创建复杂的对象，例如创建配置对象、文档对象、以及图形对象等。

​	

####  原型模式 (Prototype Pattern)




> ​	**简介：**原型模式是一种创建型设计模式，用于创建对象的新实例，同时保持原始对象的原型。这允许通过复制现有对象来创建新对象，而无需重新创建它们。

​	

```javascript
// 原型对象
class Prototype {
  constructor() {
    this.property1 = 'Default Value';
  }

  clone() {
    // 创建新对象并复制原型属性
    const clone = new this.constructor();
    clone.property1 = this.property1;
    return clone;
  }
}

// 具体原型对象
class ConcretePrototype extends Prototype {
  constructor() {
    super();
    this.property1 = 'Custom Value';
  }
}

// 使用原型创建新对象
const prototype = new ConcretePrototype();
const clone1 = prototype.clone();
const clone2 = prototype.clone();

console.log(clone1.property1); // 输出: "Custom Value"
console.log(clone2.property1); // 输出: "Custom Value"

```

​	

> **代码解读：**
> 
> - 我们首先定义了一个原型对象 `Prototype`，它包含一个属性 `property1` 和一个 `clone` 方法，用于创建新对象并复制原型属性。
> - 具体原型对象 `ConcretePrototype` 继承了原型对象，并重写了属性 `property1`。
> - 我们创建一个具体原型对象 `prototype`，然后通过调用 `clone` 方法来创建新对象 `clone1` 和 `clone2`。

​	

> **用途：** 
> 
> 1. **创建对象副本：** 原型模式用于创建对象的副本，新对象包含与原始对象相同的属性和方法。
> 2. **避免构造函数调用：** 原型模式创建对象时不会调用构造函数，可以提高性能和避免可能的副作用。
> 3. **支持动态配置：** 可以在原型对象上设置默认属性，然后通过复制原型来创建对象，并根据需要更改属性值。



​	原型模式通常在需要创建多个类似对象实例，且这些对象共享某些初始属性或状态时使用。



### 结构型模式（Structural Patterns）

​	

####  装饰器模式 (Decorator Pattern)




> ​	**简介：**装饰器模式是一种结构型设计模式，用于动态地将新行为附加到对象上，而无需修改其代码。它允许你在运行时通过将对象包装在装饰器对象中来添加新的功能。

​	

```javascript
// 基础组件
class Coffee {
  cost() {
    return 5;
  }

  getDescription() {
    return 'Coffee';
  }
}

// 装饰器基类
class CoffeeDecorator {
  constructor(coffee) {
    this.coffee = coffee;
  }

  cost() {
    return this.coffee.cost();
  }

  getDescription() {
    return this.coffee.getDescription();
  }
}

// 具体装饰器类
class MilkDecorator extends CoffeeDecorator {
  cost() {
    return this.coffee.cost() + 2;
  }

  getDescription() {
    return this.coffee.getDescription() + ', Milk';
  }
}

class SugarDecorator extends CoffeeDecorator {
  cost() {
    return this.coffee.cost() + 1;
  }

  getDescription() {
    return this.coffee.getDescription() + ', Sugar';
  }
}

// 使用装饰器
const plainCoffee = new Coffee();
console.log('Plain Coffee:', plainCoffee.getDescription(), 'Cost:', plainCoffee.cost());

const coffeeWithMilk = new MilkDecorator(plainCoffee);
console.log('Coffee with Milk:', coffeeWithMilk.getDescription(), 'Cost:', coffeeWithMilk.cost());

const coffeeWithMilkAndSugar = new SugarDecorator(coffeeWithMilk);
console.log('Coffee with Milk and Sugar:', coffeeWithMilkAndSugar.getDescription(), 'Cost:', coffeeWithMilkAndSugar.cost());

```
​	


> **代码解读：**
> 
> - 我们有一个基础组件 `Coffee`，它有 `cost` 和 `getDescription` 方法。
> - 装饰器基类 `CoffeeDecorator` 包装了基础组件，并重写了相同的方法以添加额外的功能。
> - 具体装饰器类 `MilkDecorator` 和 `SugarDecorator` 继承自装饰器基类，它们重写了 `cost` 和 `getDescription` 方法来添加牛奶和糖的功能。
> - 我们可以创建一个基础组件对象，然后通过将装饰器对象包装在它上面来添加额外的功能。


​	
> **用途：** 
> 
> 1. **动态扩展对象功能：** 装饰器模式允许你在不修改现有对象的情况下，动态地添加新功能。
> 2. **避免类爆炸：** 它避免了通过创建多个子类来添加功能，从而减少了类的数量。
> 3. **开放封闭原则：** 装饰器模式遵循开放封闭原则，允许你添加新功能而不修改已有代码。



​	装饰器模式通常在需要在运行时添加或删除对象的功能时使用。

​	

#### 适配器模式 (Adapter Pattern)




> ​	**简介：**适配器模式是一种结构型设计模式，用于将一个类的接口转换成客户端所期望的另一个接口。它允许不兼容的接口能够一起工作。

​	

```javascript
// 老接口
class OldSystem {
  operationA() {
    return 'Operation A from Old System';
  }
}

// 新接口
class NewSystem {
  operationB() {
    return 'Operation B from New System';
  }
}

// 适配器类
class Adapter {
  constructor(newSystem) {
    this.newSystem = newSystem;
  }

  operationA() {
    return this.newSystem.operationB();
  }
}

// 使用适配器
const newSystem = new NewSystem();
const adapter = new Adapter(newSystem);

console.log(newSystem.operationB()); // 输出: "Operation B from New System"
console.log(adapter.operationA());    // 输出: "Operation B from New System"

```

​	

> **代码解读：**
> 
> - 我们有一个老系统 `OldSystem`，它有一个名为 `operationA` 的方法。
> - 我们引入了一个新系统 `NewSystem`，它有一个名为 `operationB` 的方法。
> - 为了使老系统的 `operationA` 方法能够使用新系统的功能，我们创建了一个适配器 `Adapter`，它包装了新系统的实例并将 `operationA` 映射到新系统的 `operationB`。

​	

> **用途：** 
> 
> 1. **兼容新旧代码：** 适配器模式允许在新旧系统之间创建一个桥梁，以便它们可以一起工作，而无需修改已有代码。
> 2. **接口转换：** 当需要将一个接口转换成另一个接口时，适配器模式可以用于实现这种转换。
> 3. **解耦性：** 适配器模式可以降低两个不同接口之间的耦合性，使系统更加灵活和可维护。



​		适配器模式通常在需要将已有接口与新接口协同工作，或者在整合不同系统时使用。

​	

####  代理模式 (Proxy Pattern)



> ​	**简介：**代理模式是一种结构型设计模式，它允许你提供一个代理对象来控制对另一个对象的访问。代理通常用于在访问对象之前或之后执行一些额外的操作，例如延迟加载、权限控制、日志记录等。

​	

```javascript
// 真实主题
class RealSubject {
  request() {
    console.log('RealSubject: Handling request.');
  }
}

// 代理
class Proxy {
  constructor(realSubject) {
    this.realSubject = realSubject;
  }

  request() {
    if (this.checkAccess()) {
      this.realSubject.request();
      this.logAccess();
    }
  }

  checkAccess() {
    // 模拟权限检查
    console.log('Proxy: Checking access.');
    return true;
  }

  logAccess() {
    // 模拟日志记录
    console.log('Proxy: Logging access.');
  }
}

// 客户端代码
const realSubject = new RealSubject();
const proxy = new Proxy(realSubject);

proxy.request();

```
​	


> **代码解读：**
> 
> - 我们有一个真实主题 `RealSubject`，它实现了一个 `request` 方法。
> - 代理类 `Proxy` 包装了真实主题，并在访问真实主题之前执行了权限检查和日志记录。
> - 客户端代码通过代理对象来访问真实主题。


​	
> **用途：** 
> 
> 1. **远程代理：** 代理模式可以用于实现远程对象的代理，允许客户端访问远程服务器上的对象。
> 2. **虚拟代理：** 代理模式可以用于实现虚拟代理，延迟加载对象以节省资源。
> 3. **权限控制：** 代理模式可以用于实现权限控制，只有具有适当权限的用户才能访问某些对象。
> 4. **日志记录：** 代理模式可以用于记录对象的访问历史和行为，以便调试和监控。



​	代理模式通常在需要在访问对象之前或之后执行某些操作，或者在访问对象时控制访问的情况下使用。

​	

#### 桥接模式 (Bridge Pattern)



> ​	**简介：**桥接模式是一种结构型设计模式，用于将一个抽象与其实现部分分离，使它们可以独立地变化。它将一个大类或一组类分成两个独立的层次结构，使抽象和实现可以独立扩展。

​	

```javascript
// 实现接口
class Implementor {
  operation() {}
}

// 具体实现类A
class ConcreteImplementorA extends Implementor {
  operation() {
    return 'Concrete Implementor A Operation';
  }
}

// 具体实现类B
class ConcreteImplementorB extends Implementor {
  operation() {
    return 'Concrete Implementor B Operation';
  }
}

// 抽象类
class Abstraction {
  constructor(implementor) {
    this.implementor = implementor;
  }

  operation() {
    return this.implementor.operation();
  }
}

// 具体抽象类X
class ConcreteAbstractionX extends Abstraction {
  operation() {
    return `Concrete Abstraction X: ${this.implementor.operation()}`;
  }
}

// 具体抽象类Y
class ConcreteAbstractionY extends Abstraction {
  operation() {
    return `Concrete Abstraction Y: ${this.implementor.operation()}`;
  }
}

// 客户端代码
const implementorA = new ConcreteImplementorA();
const implementorB = new ConcreteImplementorB();

const abstractionX = new ConcreteAbstractionX(implementorA);
const abstractionY = new ConcreteAbstractionY(implementorB);

console.log(abstractionX.operation()); // 输出: "Concrete Abstraction X: Concrete Implementor A Operation"
console.log(abstractionY.operation()); // 输出: "Concrete Abstraction Y: Concrete Implementor B Operation"

```

​	

> **代码解读：**
> 
> - 我们有一个实现接口 `Implementor`，它包含一个 `operation` 方法。
> - 具体实现类 `ConcreteImplementorA` 和 `ConcreteImplementorB` 分别实现了 `Implementor` 接口，并提供具体的操作。
> - 抽象类 `Abstraction` 包含一个 `Implementor` 实例，并提供了一个 `operation` 方法，该方法调用 `Implementor` 实例的操作。
> - 具体抽象类 `ConcreteAbstractionX` 和 `ConcreteAbstractionY` 继承自 `Abstraction`，并可以修改 `Implementor` 实例的操作结果。


​	
> **用途：** 
> 
> 1. **分离抽象和实现：** 桥接模式将抽象和实现分离，允许它们独立变化，从而提高了代码的灵活性。
> 2. **多维度变化：** 当一个类有多个独立的变化维度时，桥接模式可以避免类的指数性增长。



​	桥接模式通常在需要处理多个变化维度且希望避免创建多个类的情况下使用。

​	

#### 组合模式 (Composite Pattern)



> ​	**简介：**组合模式是一种结构型设计模式，用于将对象组合成树状结构以表示部分-整体层次结构。它使客户端可以统一地处理单个对象和组合对象。

​	

```javascript
// 抽象组件
class Component {
  constructor(name) {
    this.name = name;
  }

  operation() {}
}

// 叶子组件
class Leaf extends Component {
  operation() {
    console.log(`Leaf ${this.name} is operated.`);
  }
}

// 复合组件
class Composite extends Component {
  constructor(name) {
    super(name);
    this.children = [];
  }

  operation() {
    console.log(`Composite ${this.name} is operated.`);
    for (const child of this.children) {
      child.operation();
    }
  }

  add(child) {
    this.children.push(child);
  }

  remove(child) {
    const index = this.children.indexOf(child);
    if (index !== -1) {
      this.children.splice(index, 1);
    }
  }
}

// 使用组合模式
const root = new Composite('Root');
const branch1 = new Composite('Branch 1');
const branch2 = new Composite('Branch 2');

const leaf1 = new Leaf('Leaf 1');
const leaf2 = new Leaf('Leaf 2');
const leaf3 = new Leaf('Leaf 3');

branch1.add(leaf1);
branch1.add(leaf2);

branch2.add(leaf3);

root.add(branch1);
root.add(branch2);

// 调用操作
root.operation();

```

​	

> **代码解读：**
> 
> - 我们首先定义了抽象组件类 `Component`，它包含一个 `name` 属性和一个 `operation` 方法。
> - 叶子组件 `Leaf` 继承自 `Component`，实现了具体的 `operation` 方法。
> - 复合组件 `Composite` 也继承自 `Component`，它包含一个 `children` 数组来存储子组件，并实现了 `operation`、`add` 和 `remove` 方法。
> - 我们创建了一个组合结构，包括根节点、分支节点和叶子节点，然后通过 `add` 方法将它们组合在一起。
> - 最后，我们调用根节点的 `operation` 方法，它会递归执行所有子组件的 `operation` 方法。


​	
> **用途：** 
> 
> 1. **表示部分-整体层次结构：** 组合模式用于表示具有部分-整体层次结构的对象，允许客户端一致地处理单个对象和组合对象。
> 2. **简化客户端代码：** 客户端不需要知道具体的组合结构，只需与抽象组件类交互。
> 3. **支持递归操作：** 组合模式支持递归操作，可以对整个组合结构执行操作。



​	组合模式通常在需要构建具有层次结构的对象，并希望客户端能够一致地处理不同层次的对象时使用。



### 行为型模式（Behavioral Patterns）

​	

#### 观察者模式 (Observer Pattern)



> ​	**简介：**观察者模式是一种行为型设计模式，它定义了一种一对多的依赖关系，让多个观察者对象可以自动收到被观察对象的状态变化通知并进行相应的处理。

​	

**示例代码：**

```javascript
// 主题 (被观察者)
class Subject {
  constructor() {
    this.observers = [];
  }

  addObserver(observer) {
    this.observers.push(observer);
  }

  removeObserver(observer) {
    const index = this.observers.indexOf(observer);
    if (index !== -1) {
      this.observers.splice(index, 1);
    }
  }

  notifyObservers() {
    for (const observer of this.observers) {
      observer.update(this);
    }
  }
}

// 观察者接口
class Observer {
  update(subject) {}
}

// 具体观察者
class ConcreteObserverA extends Observer {
  update(subject) {
    console.log('Observer A received the update from subject.');
  }
}

class ConcreteObserverB extends Observer {
  update(subject) {
    console.log('Observer B received the update from subject.');
  }
}

// 客户端代码
const subject = new Subject();
const observerA = new ConcreteObserverA();
const observerB = new ConcreteObserverB();

subject.addObserver(observerA);
subject.addObserver(observerB);

subject.notifyObservers();

```

​	

> **代码解读：**
> 
> - 我们首先定义了一个主题（被观察者） `Subject`，它包含了一个观察者数组 `observers`，并提供了添加、移除、通知观察者的方法。
> - 观察者接口 `Observer` 定义了一个 `update` 方法，具体的观察者需要实现这个方法。
> - 具体观察者 `ConcreteObserverA` 和 `ConcreteObserverB` 继承自 `Observer`，并实现了 `update` 方法来处理主题的通知。
> - 客户端代码创建了一个主题对象，然后添加了两个观察者，最后通过 `notifyObservers` 方法通知观察者。

​	
​
> **用途：** 
> 
> 1. **解耦性：** 观察者模式可以将主题与观察者解耦，使主题不需要知道观察者的具体实现。
> 2. **一对多通知：** 它允许一个主题通知多个观察者，每个观察者可以根据自己的需求对通知做出响应。
> 3. **动态变化：** 观察者模式支持动态地添加或移除观察者，主题和观察者之间的关系可以随时变化。



​	观察者模式通常在需要实现发布-订阅系统、事件处理、用户界面组件更新等场景中使用。

​	

#### 策略模式 (Strategy Pattern)




>  **简介：**策略模式是一种行为型设计模式，它定义了一系列算法，将每个算法封装成一个策略，使这些策略可以互相替换。客户端可以选择不同的策略来实现不同的行为。

​	

**示例代码：**

```javascript
// 策略接口
class PaymentStrategy {
  pay(amount) {}
}

// 具体策略类：信用卡支付
class CreditCardPayment extends PaymentStrategy {
  pay(amount) {
    console.log(`Paid $${amount} with Credit Card.`);
  }
}

// 具体策略类：PayPal支付
class PayPalPayment extends PaymentStrategy {
  pay(amount) {
    console.log(`Paid $${amount} with PayPal.`);
  }
}

// 具体策略类：现金支付
class CashPayment extends PaymentStrategy {
  pay(amount) {
    console.log(`Paid $${amount} in Cash.`);
  }
}

// 环境类
class ShoppingCart {
  constructor(paymentStrategy) {
    this.items = [];
    this.paymentStrategy = paymentStrategy;
  }

  addItem(item) {
    this.items.push(item);
  }

  checkout() {
    let total = 0;
    for (const item of this.items) {
      total += item.price;
    }
    this.paymentStrategy.pay(total);
  }
}

// 客户端代码
const cart = new ShoppingCart(new CreditCardPayment());

cart.addItem({ name: 'Item 1', price: 25 });
cart.addItem({ name: 'Item 2', price: 30 });

cart.checkout();

```

​	

> **代码解读：**
> 
> - 我们首先定义了一个策略接口 `PaymentStrategy`，它包含一个 `pay` 方法。
> - 具体策略类 `CreditCardPayment`、`PayPalPayment` 和 `CashPayment` 分别继承自 `PaymentStrategy`，并实现了 `pay` 方法来完成具体的支付方式。
> - 环境类 `ShoppingCart` 接受一个支付策略，并包含一个 `items` 数组来存储购物车中的物品。
> - 客户端代码创建了一个购物车对象，选择了信用卡支付策略，然后添加了两个物品并进行结算。

​	

> **用途：** 
> 
> 1. **多算法选择：** 策略模式用于在运行时选择算法，使客户端能够灵活地选择不同的策略。
> 2. **解耦性：** 它将算法的实现与客户端代码分离，提高了代码的可维护性。
> 3. **替代继承：** 策略模式可以替代多层继承，避免类爆炸问题。



​	策略模式通常在需要动态地切换算法或在一个类有多种行为变体的情况下使用。

​	

#### 命令模式 (Command Pattern)



> **简介：**命令模式是一种行为型设计模式，它将请求封装成一个对象，以便可以对请求的参数化、队列化、记录日志、撤销等操作，从而使系统更加灵活。

​	

**示例代码：**

```javascript
// 命令接口
class Command {
  execute() {}
  undo() {}
}

// 具体命令类：打开电视
class TVOnCommand extends Command {
  constructor(tv) {
    super();
    this.tv = tv;
  }

  execute() {
    this.tv.turnOn();
  }

  undo() {
    this.tv.turnOff();
  }
}

// 具体命令类：关闭电视
class TVOffCommand extends Command {
  constructor(tv) {
    super();
    this.tv = tv;
  }

  execute() {
    this.tv.turnOff();
  }

  undo() {
    this.tv.turnOn();
  }
}

// 接收者：电视
class TV {
  turnOn() {
    console.log('TV is ON');
  }

  turnOff() {
    console.log('TV is OFF');
  }
}

// 遥控器（调用者）
class RemoteControl {
  constructor() {
    this.commands = [];
  }

  addCommand(command) {
    this.commands.push(command);
  }

  pressButton() {
    for (const command of this.commands) {
      command.execute();
    }
  }

  undoButton() {
    for (const command of this.commands.reverse()) {
      command.undo();
    }
  }
}

// 客户端代码
const tv = new TV();
const tvOnCommand = new TVOnCommand(tv);
const tvOffCommand = new TVOffCommand(tv);

const remoteControl = new RemoteControl();
remoteControl.addCommand(tvOnCommand);
remoteControl.addCommand(tvOffCommand);

remoteControl.pressButton();  // 打开电视
remoteControl.undoButton();   // 关闭电视

```
​	


> **代码解读：**
> 
> - 我们首先定义了一个命令接口 `Command`，它包含 `execute` 和 `undo` 方法。
> - 具体命令类 `TVOnCommand` 和 `TVOffCommand` 分别继承自 `Command`，并封装了打开和关闭电视的操作。
> - 接收者 `TV` 包含了实际的电视操作。
> - 遥控器 `RemoteControl` 包含了命令列表，并提供了 `pressButton` 和 `undoButton` 方法来执行和撤销命令。

​	

> **用途：** 
> 
> 1. **将请求封装成对象：** 命令模式将请求和其参数封装成一个对象，可以用于参数化、队列化、撤销等操作。
> 2. **解耦性：** 客户端不需要知道命令的具体实现，从而降低了命令发送者和接收者之间的耦合度。
> 3. **支持撤销操作：** 命令模式支持撤销操作，通过 `undo` 方法可以还原之前的操作。



​	命令模式通常在需要将请求参数化、支持撤销操作或实现命令队列等场景中使用。

​	

#### 发布-订阅模式 (Publish-Subscribe Pattern)




> ​	**简介：**发布-订阅模式是一种行为型设计模式，它定义了对象间的一种一对多的依赖关系，当一个对象的状态发生变化时，所有依赖于它的对象都会得到通知并自动更新。
​	


**示例代码：**

```javascript
// 发布者 (发布事件的对象)
class Publisher {
  constructor() {
    this.subscribers = [];
  }

  subscribe(subscriber) {
    this.subscribers.push(subscriber);
  }

  unsubscribe(subscriber) {
    const index = this.subscribers.indexOf(subscriber);
    if (index !== -1) {
      this.subscribers.splice(index, 1);
    }
  }

  notify(data) {
    for (const subscriber of this.subscribers) {
      subscriber.update(data);
    }
  }
}

// 订阅者 (订阅事件的对象)
class Subscriber {
  constructor(name) {
    this.name = name;
  }

  update(data) {
    console.log(`${this.name} received update with data: ${data}`);
  }
}

// 客户端代码
const publisher = new Publisher();

const subscriberA = new Subscriber('Subscriber A');
const subscriberB = new Subscriber('Subscriber B');
const subscriberC = new Subscriber('Subscriber C');

publisher.subscribe(subscriberA);
publisher.subscribe(subscriberB);
publisher.subscribe(subscriberC);

publisher.notify('Message 1');
publisher.unsubscribe(subscriberB);

publisher.notify('Message 2');

```

​	

> **代码解读：**
> 
> - 我们首先定义了一个发布者 `Publisher`，它包含了一个订阅者数组 `subscribers`，并提供了订阅、取消订阅和通知的方法。
> - 订阅者 `Subscriber` 有一个 `name` 属性，并实现了 `update` 方法来处理接收到的通知。
> - 客户端代码创建了一个发布者对象，然后添加了三个订阅者，并发送两条通知消息。

​	

> **用途：** 
> 
> 1. **解耦性：** 发布-订阅模式用于解耦发布者和订阅者，使它们可以独立演化。
> 2. **事件处理：** 它常用于事件处理机制，让多个对象监听并响应事件。
> 3. **消息通信：** 发布-订阅模式可用于消息通信，不同部分之间可以通过事件进行通信。



​	发布-订阅模式通常在需要实现松散耦合、事件驱动、多对多通信等场景中使用。

​	

#### 模块模式 (Module Pattern)



> ​	**简介：**模块模式是一种设计模式，用于将代码组织成可维护和可重用的模块，同时隐藏了内部实现的细节。它通过使用闭包来创建私有变量和函数，提供了一种封装数据和行为的方法。


​	
**示例代码：**

```javascript
// 模块
const myModule = (function () {
  // 私有变量
  let privateVar = 0;

  // 私有函数
  function privateFunction() {
    console.log('Private Function');
    privateVar++;
  }

  // 公有接口（可以访问私有成员）
  return {
    publicVar: 100,

    publicFunction: function () {
      console.log('Public Function');
      privateFunction();
    },

    getPrivateVar: function () {
      return privateVar;
    }
  };
})();

// 客户端代码
console.log(myModule.publicVar);        // 访问公有变量
myModule.publicFunction();               // 调用公有函数
console.log(myModule.getPrivateVar());   // 获取私有变量

```

​	

> **代码解读：**
> 
> - 我们创建了一个匿名的立即执行函数，它返回一个包含公有接口的对象，这个对象可以访问模块内部的私有变量和函数。
> - 模块内部定义了私有变量 `privateVar` 和私有函数 `privateFunction`，它们无法在模块外部直接访问。
> - 公有接口包含了公有变量 `publicVar`、公有函数 `publicFunction` 和一个获取私有变量的方法 `getPrivateVar`。

​	

> **用途：** 
> 
> 1. **封装数据和行为：** 模块模式用于将数据和行为封装在一个模块内，隐藏了内部实现的细节，提供了一个清晰的公共接口。
> 2. **单例模式：** 模块模式可以用于创建单例对象，确保只有一个实例存在。
> 3. **避免全局污染：** 使用模块模式可以避免将变量和函数添加到全局作用域，减少了命名冲突的可能性。



​	模块模式通常在需要封装一组相关的变量和函数，并将其隐藏在模块内部的情况下使用，以提高代码的可维护性和可复用性。

​	

#### 模板方法模式 (Template Method Pattern)



> ​	**简介：**模板方法模式是一种行为型设计模式，它定义了一个算法的骨架，将具体步骤的实现延迟到子类。模板方法使子类可以在不改变算法结构的情况下重新定义算法的某些步骤。

​	

**示例代码：**

```javascript
// 模板类
class Game {
  constructor() {}

  initialize() {
    console.log('Game initialized.');
  }

  start() {
    this.initialize();
    this.play();
    this.end();
  }

  play() {}

  end() {
    console.log('Game ended.');
  }
}

// 具体游戏类：Basketball
class BasketballGame extends Game {
  play() {
    console.log('Playing basketball.');
  }
}

// 具体游戏类：Soccer
class SoccerGame extends Game {
  play() {
    console.log('Playing soccer.');
  }
}

// 客户端代码
const basketball = new BasketballGame();
basketball.start();

const soccer = new SoccerGame();
soccer.start();

```
​	


> **代码解读：**
> 
> - 我们首先定义了一个抽象的游戏类 `Game`，它包含了一个模板方法 `start`，以及初始化方法 `initialize`、游戏方法 `play` 和结束方法 `end`。
> - `start` 方法按照固定的顺序调用了 `initialize`、`play` 和 `end` 方法，这个顺序定义了游戏的骨架。
> - 具体游戏类 `BasketballGame` 和 `SoccerGame` 继承自 `Game` 并实现了 `play` 方法，从而定义了具体的游戏行为。

​	

> **用途：** 
> 
> 1. **定义算法骨架：** 模板方法模式用于定义算法的骨架，将算法的步骤抽象出来，并允许子类提供具体实现。
> 2. **代码复用：** 它提供了一种重用代码的方式，将通用部分放在模板方法中，将变化的部分留给子类。
> 3. **防止滥用继承：** 模板方法模式避免了滥用继承，因为它允许在不改变算法结构的情况下重定义部分步骤。



​	模板方法模式通常在需要定义算法骨架、有多个子类共享一些公共行为、避免代码重复等情况下使用。

​	

#### 迭代器模式 (Iterator Pattern)


> ​	**简介：**迭代器模式是一种行为型设计模式，用于提供一种顺序访问集合对象元素的方法，而不需要暴露集合的内部表示。它允许客户端访问集合对象中的元素，而不必了解底层数据结构。


​	
**示例代码：**

```javascript
// 迭代器接口
class Iterator {
  constructor(collection) {
    this.collection = collection;
    this.index = 0;
  }

  hasNext() {}

  next() {}
}

// 具体迭代器：数组迭代器
class ArrayIterator extends Iterator {
  hasNext() {
    return this.index < this.collection.length;
  }

  next() {
    return this.hasNext() ? this.collection[this.index++] : null;
  }
}

// 集合类
class Collection {
  constructor() {
    this.items = [];
  }

  addItem(item) {
    this.items.push(item);
  }

  getIterator() {
    return new ArrayIterator(this.items);
  }
}

// 客户端代码
const collection = new Collection();
collection.addItem('Item 1');
collection.addItem('Item 2');
collection.addItem('Item 3');

const iterator = collection.getIterator();

while (iterator.hasNext()) {
  console.log(iterator.next());
}

```

​	

> **代码解读：**
> 
> - 我们首先定义了一个迭代器接口 `Iterator`，它包含了 `hasNext` 和 `next` 方法。
> - 具体迭代器 `ArrayIterator` 继承自 `Iterator`，实现了针对数组的迭代逻辑。
> - 集合类 `Collection` 包含一个数组 `items` 和一个获取迭代器的方法 `getIterator`。
> - 客户端代码创建了一个集合对象，添加了三个元素，然后获取了一个迭代器并使用 `while` 循环遍历元素。

​	

> **用途：** 
> 
> 1. **封装集合遍历逻辑：** 迭代器模式封装了集合遍历的逻辑，使客户端代码不必了解底层数据结构。
> 2. **支持多种集合类型：** 它可以为不同类型的集合提供通用的遍历方式，无需修改客户端代码。
> 3. **提供一致的接口：** 迭代器模式为不同集合提供了一致的接口，简化了客户端代码。



​	迭代器模式通常在需要遍历集合对象、隐藏集合的内部结构、支持不同类型的集合等场景中使用。

​	

#### 责任链模式 (Chain of Responsibility Pattern)




>  **简介：**责任链模式是一种行为型设计模式，用于构建一个对象链，每个对象都有机会处理请求，但请求会在链上传递直到有一个对象处理它为止。责任链模式将请求发送者与接收者解耦，允许多个对象都有机会处理请求。

​	

**示例代码：**

```javascript
// 处理请求的抽象处理者
class Handler {
  constructor() {
    this.successor = null;
  }

  setSuccessor(successor) {
    this.successor = successor;
  }

  handleRequest(request) {}
}

// 具体处理者 A
class ConcreteHandlerA extends Handler {
  handleRequest(request) {
    if (request === 'A') {
      console.log('Handler A is handling the request.');
    } else if (this.successor !== null) {
      this.successor.handleRequest(request);
    }
  }
}

// 具体处理者 B
class ConcreteHandlerB extends Handler {
  handleRequest(request) {
    if (request === 'B') {
      console.log('Handler B is handling the request.');
    } else if (this.successor !== null) {
      this.successor.handleRequest(request);
    }
  }
}

// 客户端代码
const handlerA = new ConcreteHandlerA();
const handlerB = new ConcreteHandlerB();

handlerA.setSuccessor(handlerB);

handlerA.handleRequest('A'); // Handler A is handling the request.
handlerA.handleRequest('B'); // Handler B is handling the request.
handlerA.handleRequest('C'); // (No handler can handle the request.)

```

​	

> **代码解读：**
> 
> - 我们首先定义了一个抽象的处理者类 `Handler`，它包含一个指向下一个处理者的引用 `successor` 和一个 `handleRequest` 方法。
> - 具体处理者类 `ConcreteHandlerA` 和 `ConcreteHandlerB` 继承自 `Handler`，并实现了自己的 `handleRequest` 方法来处理请求。
> - 客户端代码创建了两个具体处理者对象，然后通过 `setSuccessor` 方法设置它们的处理顺序，最后调用第一个处理者的 `handleRequest` 方法来处理请求。

​	

> **用途：** 
> 
> 1. **解耦发送者和接收者：** 责任链模式将请求的发送者和接收者解耦，允许多个对象都有机会处理请求，从而降低了它们之间的依赖关系。
> 2. **动态构建责任链：** 可以动态地添加、删除或修改处理者，以满足不同的请求处理需求。
> 3. **避免硬编码逻辑：** 责任链模式可以避免硬编码请求的处理逻辑，使代码更加灵活和可维护。



​	责任链模式通常在需要动态决定请求的处理顺序、多个对象有机会处理请求、避免请求发送者和接收者之间紧密耦合的情况下使用。




​	
####  状态模式 (State Pattern)



> ​	**简介：**状态模式是一种行为型设计模式，它允许对象在内部状态发生改变时改变它的行为。这种模式将状态封装成独立的类，并将上下文对象委托给当前状态对象，以便在状态发生变化时切换到不同的状态。

​	

**示例代码：**

```javascript
// 状态接口
class State {
  handle(context) {}
}

// 具体状态类：开机状态
class PowerOnState extends State {
  handle(context) {
    console.log('TV is ON');
    context.setState(new PowerOffState());
  }
}

// 具体状态类：关机状态
class PowerOffState extends State {
  handle(context) {
    console.log('TV is OFF');
    context.setState(new PowerOnState());
  }
}

// 上下文类
class TVContext {
  constructor() {
    this.state = new PowerOffState();
  }

  setState(state) {
    this.state = state;
  }

  request() {
    this.state.handle(this);
  }
}

// 客户端代码
const tv = new TVContext();

tv.request(); // TV is OFF
tv.request(); // TV is ON
tv.request(); // TV is OFF

```

​	

> **代码解读：**
> 
> - 我们首先定义了一个状态接口 `State`，它包含一个 `handle` 方法。
> - 具体状态类 `PowerOnState` 和 `PowerOffState` 继承自 `State`，并分别实现了 `handle` 方法来处理开机和关机状态。
> - 上下文类 `TVContext` 包含一个当前状态对象，并提供了 `setState` 和 `request` 方法。`setState` 方法用于切换当前状态，而 `request` 方法用于发出请求，委托给当前状态来处理。

​	

> **用途：**
>
> 1. **封装状态：** 状态模式将每个状态封装成一个独立的类，使状态的变化对上下文对象透明，提高了代码的可维护性。
> 2. **减少条件语句：** 它避免了大量的条件语句，使代码更加清晰和可读。
> 3. **支持扩展性：** 可以轻松地添加新的状态类，扩展系统的行为。



​	状态模式通常在需要对象在不同状态下执行不同操作的场景中使用，它有助于将状态转换和状态处理的逻辑封装起来，提高了代码的可扩展性和可维护性。

