---
title: 程序设计的 SOLID 原则
date: 2022-07-18 14:14:29
tags: [typescript,程序设计,SOLID]
categories: typescript
cover: https://6973-isundae-3gbm0y3523031b2a-1258186980.tcb.qcloud.la/blog/202209041400219.jpg
---

# 程序设计的 SOLID 原则🎯

SOLID 原则其实是用来指导软件设计的，它一共分为五条设计原则，分别是：

1. 单一职责原则（SRP）
2. 开闭原则（OCP）
3. 里氏替换原则（LSP）
4. 接口隔离原则（ISP）
5. 依赖倒置原则（DIP）

## 单一职责原则（SRP）🎯

> 核心思想：类的职责应该单一，不要承担过多的职责。

    

    先看下面这段代码，为 Book 创建了一个类，但是类中却承担了多个职责，比如把书保存为一个文件：

```typescript
class Book {
  public title: string;
  public author: string;
  public description: string;
  public pages: number;

  // constructor and other methods

  public saveToFile(): void {
    // some fs.write method to save book to file
  }
}
```

    遵循单一职责原则，应该创建两个类，分别负责不同的事情：

```typescript
class Book {
  public title: string;
  public author: string;
  public description: string;
  public pages: number;

  // constructor and other methods
}

class Persistence {
  public saveToFile(book: Book): void {
    // some fs.write method to save book to file
  }
}
```

> 好处：降低类的复杂度、提高可读性、可维护性、扩展性、最大限度的减少潜在的副作用。

## 开闭原则（OCP）🎯

> 核心思想：类应该对扩展开放，但对修改关闭。简单理解就是当别人要修改软件功能的时候，不能让他修改我们原有代码，尽量让他在原有的基础上做扩展。

    

    下面这段写的不太好的代码，我们单独封装了一个 AreaCalculator 类来负责计算 Rectangle 和 Circle 类的面积。想象一下，如果我们后续要再添加一个形状，我们要创建一个新的类，同时我们也要去修改 AreaCalculator 来计算新类的面积，这违反了开闭原则。

```typescript
class Rectangle {
  public width: number;
  public height: number;

  constructor(width: number, height: number) {
    this.width = width;
    this.height = height;
  }
}

class Circle {
  public radius: number;

  constructor(radius: number) {
    this.radius = radius;
  }
}

class AreaCalculator {
  public calculateRectangleArea(rectangle: Rectangle): number {
    return rectangle.width * rectangle.height;
  }

  public calculateCircleArea(circle: Circle): number {
    return Math.PI * (circle.radius * circle.radius);
  }
}
```

    为了遵循开闭原则，我们只需要添加一个名为 Shape 的接口，每个形状类（矩形、圆形等）都可以通过实现它来依赖该接口。通过这种方式，可以将 AreaCalculator 类简化为一个带有参数的函数，每当创建一个新的形状类，都必须实现这个函数，这样就不需要修改原有的类了：

```typescript
interface Shape {
  calculateArea(): number;
}

class Rectangle implements Shape {
  public width: number;
  public height: number;

  constructor(width: number, height: number) {
    this.width = width;
    this.height = height;
  }

  public calculateArea(): number {
    return this.width * this.height;
  }
}

class Circle implements Shape {
  public radius: number;

  constructor(radius: number) {
    this.radius = radius;
  }

  public calculateArea(): number {
    return Math.PI * (this.radius * this.radius);
  }
}

class AreaCalculator {
  public calculateArea(shape: Shape): number {
    return shape.calculateArea();
  }
}
```

## 里氏替换原则（LSP）🎯

> 核心思想：在使用基类的的地方可以任意使用其子类，能保证子类完美替换基类。简单理解就是所有父类能出现的地方，子类就可以出现，并且替换了也不会出现任何错误。

    要求子类的所有相同方法，都必须遵循父类的约定，否则当父类替换为子类时就会出错。

    先来看看下面这段代码，Square 类扩展了 Rectangle 类。但是这个扩展没有任何意义，因为通过覆盖宽度和高度属性来改变了原有的逻辑。

```typescript
class Rectangle {
  public width: number;
  public height: number;

  constructor(width: number, height: number) {
    this.width = width;
    this.height = height;
  }

  public calculateArea(): number {
    return this.width * this.height;
  }
}

class Square extends Rectangle {
  public _width: number;
  public _height: number;

  constructor(width: number, height: number) {
    super(width, height);

    this._width = width;
    this._height = height;
  }
}
```

    遵循里氏替换原则，不需要覆盖基类的属性，而是直接删除掉 Square 类并，将它的逻辑带到 Rectangle 类，而且也不改变其用途。

```typescript
class Rectangle {
  public width: number;
  public height: number;

  constructor(width: number, height: number) {
    this.width = width;
    this.height = height;
  }

  public calculateArea(): number {
    return this.width * this.height;
  }

  public isSquare(): boolean {
    return this.width === this.height;
  }
}
```

    好处：增强程序的健壮性，即使增加了子类，原有的子类还可以继续运行。

## 接口隔离原则（ISP）🎯

> 核心思想：类间的依赖关系应该建立在最小的接口上。简单理解就是接口的内容一定要尽可能地小，能有多小就多小。我们要为各个类建立专用的接口，而不要试图去建立一个很庞大的接口供所有依赖它的类去调用。

    看下面的代码，有一个名为 Troll 的类，它实现了一个名为 Character 的接口，但是 Troll 既不会游泳也不会说话，所以它似乎不太适合实现我们的接口：

```typescript
interface Character {
  shoot(): void;
  swim(): void;
  talk(): void;
  dance(): void;
}

class Troll implements Character {
  public shoot(): void {
    // some method
  }

  public swim(): void {
    // a troll can't swim
  }

  public talk(): void {
    // a troll can't talk
  }

  public dance(): void {
    // some method
  }
}
```

    遵循接口隔离原则，删除 Character 接口并将它的功能拆分为四个接口，然后 Troll 类只需要依赖于我们实际需要的这些接口。

```typescript
interface Talker {
  talk(): void;
}

interface Shooter {
  shoot(): void;
}

interface Swimmer {
  swim(): void;
}

interface Dancer {
  dance(): void;
}

class Troll implements Shooter, Dancer {
  public shoot(): void {
    // some method
  }

  public dance(): void {
    // some method
  }
}
```

## 依赖倒置原则（DIP）🎯

> 核心思想：依赖一个抽象的服务接口，而不是去依赖一个具体的服务执行者，从依赖具体实现转向到依赖抽象接口，倒置过来。

    看看下面这段代码，有一个 SoftwareProject 类，它初始化了 FrontendDeveloper 和 BackendDeveloper 类：

```ts
class FrontendDeveloper {
  public writeHtmlCode(): void {
    // some method
  }
}

class BackendDeveloper {
  public writeTypeScriptCode(): void {
    // some method
  }
}

class SoftwareProject {
  public frontendDeveloper: FrontendDeveloper;
  public backendDeveloper: BackendDeveloper;

  constructor() {
    this.frontendDeveloper = new FrontendDeveloper();
    this.backendDeveloper = new BackendDeveloper();
  }

  public createProject(): void {
    this.frontendDeveloper.writeHtmlCode();
    this.backendDeveloper.writeTypeScriptCode();
  }
}
```

    遵循依赖倒置原则，创建一个 Developer 接口，由于 FrontendDeveloper 和 BackendDeveloper 是相似的类，它们都依赖于 Developer 接口。

    不需要在 SoftwareProject 类中以单一方式初始化 FrontendDeveloper 和 BackendDeveloper，而是将它们作为一个列表来遍历它们，分别调用每个 develop() 方法。

```ts
interface Developer {
  develop(): void
}

class FrontendDeveloper implements Developer {
  public develop(): void {
    this.writeHtmlCode()
  }

  private writeHtmlCode(): void {
    // some method
  }
}

class BackendDeveloper implements Developer {
  public develop(): void {
    this.writeTypeScriptCode()
  }

  private writeTypeScriptCode(): void {
    // some method
  }
}

class SoftwareProject {
  public developers: Developer[] = []

  public createProject(): void {
    this.developers.forEach((developer: Developer) => {
      developer.develop()
    })
  }
}

const softwareProject = new SoftwareProject()
softwareProject.developers = [new FrontendDeveloper(), new BackendDeveloper()]
softwareProject.createProject()
```

或者

```typescript
interface Developer {
  develop(): void
}

class FrontendDeveloper implements Developer {
  public develop(): void {
    this.writeHtmlCode()
  }

  private writeHtmlCode(): void {
    // some method
  }
}

class BackendDeveloper implements Developer {
  public develop(): void {
    this.writeTypeScriptCode()
  }

  private writeTypeScriptCode(): void {
    // some method
  }
}

class SoftwareProject {
  private developers: Developer[] = []
  constructor(...developers: Developer[]) {
    this.developers.push(...developers)
  }

  public createProject(): void {
    this.developers.forEach((developer: Developer) => {
      developer.develop()
    })
  }
}

const softwareProject = new SoftwareProject(
  new FrontendDeveloper(),
  new BackendDeveloper()
).createProject()
```

    好处：实现模块间的松耦合，更利于多模块并行开发。