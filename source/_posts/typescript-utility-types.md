---
title: TypeScript 中各种实用工具类型的功能示例
toc: true
categories:
  - Front-end Study
tags:
  - JavaScript
date: 2024-03-20 16:18:34
---


### Partial

Partial 工具类型可以将接口中的所有属性转换为可选属性。

```typescript
interface User {
  id: number;
  name: string;
  age: number;
}

type PartialUser = Partial<User>;

const partialUser: PartialUser = { id: 1 };
```

<!-- more -->

### Required

Required 工具类型可以将接口中的所有属性转换为必选属性。

```typescript
interface Config {
  apiKey?: string;
  endpoint?: string;
}

type RequiredConfig = Required<Config>;

const requiredConfig: RequiredConfig = { apiKey: '123', endpoint: 'example.com' };
```

### Readonly

Readonly 工具类型可以将接口中的所有属性转换为只读属性。

```typescript
interface Point {
  x: number;
  y: number;
}

type ReadonlyPoint = Readonly<Point>;

const readonlyPoint: ReadonlyPoint = { x: 10, y: 20 };
// readonlyPoint.x = 5; // Error: Cannot assign to 'x' because it is a read-only property
```

### Record

Record 工具类型可以创建具有指定键类型和值类型的对象。

```typescript
type Fruit = 'Apple' | 'Banana' | 'Orange';
type Price = number;

const fruitPrices: Record<Fruit, Price> = {
  Apple: 1,
  Banana: 2,
  Orange: 3,
};
```

### Pick

Pick 工具类型可以从接口中选择指定的属性。

```typescript
interface Car {
  make: string;
  model: string;
  year: number;
}

type CarInfo = Pick<Car, 'make' | 'model'>;

const carInfo: CarInfo = { make: 'Toyota', model: 'Camry' };
```

### Omit

Omit 工具类型可以从接口中排除指定的属性。

```typescript
interface Book {
  title: string;
  author: string;
  pages: number;
}

type BookSummary = Omit<Book, 'pages'>;

const bookSummary: BookSummary = { title: 'Sample Book', author: 'John Doe' };
```

### Exclude

Exclude 工具类型可以从联合类型中排除指定的类型。

```typescript
type Color = 'Red' | 'Green' | 'Blue';
type PrimaryColor = Exclude<Color, 'Green' | 'Blue'>;

const primaryColor: PrimaryColor = 'Red';
```

### Extract

Extract 工具类型可以从联合类型中提取指定的类型。

```typescript
type Color = 'Red' | 'Green' | 'Blue';
type SecondaryColor = Extract<Color, 'Green' | 'Blue'>;

const secondaryColor: SecondaryColor = 'Green';
```

### NonNullable

NonNullable 工具类型可以从类型中排除 null 和 undefined。

```typescript
type NullableString = string | null | undefined;
type NonNullString = NonNullable<NullableString>;

const nonNullString: NonNullString = 'Hello';
```

### Parameters

Parameters 工具类型可以获取函数类型的参数类型组成的元组。

```typescript
function greet(name: string, age: number) {
  console.log(`Hello, ${name}! You are ${age} years old.`);
}

type GreetParams = Parameters<typeof greet>;

const params: GreetParams = ['Alice', 30];
```

### ConstructorParameters

ConstructorParameters 工具类型可以获取构造函数类型的参数类型组成的元组。

```typescript
class Person {
  constructor(public name: string, public age: number) {}
}

type PersonConstructorParams = ConstructorParameters<typeof Person>;

const constructorParams: PersonConstructorParams = ['Bob', 25];
```

### ReturnType

ReturnType 工具类型可以获取函数类型的返回值类型。

```typescript
function add(a: number, b: number): number {
  return a + b;
}

type AddResult = ReturnType<typeof add>;

const result: AddResult = 5;
```

### InstanceType

InstanceType 工具类型可以获取构造函数类型的实例类型。

```typescript
class Product {
  constructor(public name: string, public price: number) {}
}

type ProductInstance = InstanceType<typeof Product>;

const product: ProductInstance = new Product('Phone', 500);
```
