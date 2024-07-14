---
# You can also start simply with 'default'
# theme: seriph
theme: default
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides (markdown enabled)
title: Typescript Generic
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# apply unocss classes to the current slide
class: text-center
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# https://sli.dev/guide/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations#slide-transitions
transition: slide-up
# enable MDC Syntax: https://sli.dev/guide/syntax#mdc-syntax
mdc: true
lineNumbers: true
---

# Typescript Generic

Hippo Lin

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    Press Space for next page <carbon:arrow-right class="inline"/>
  </span>
</div>

---

# 泛型解決了什麼？

## 靈活性

我們希望所寫出來的工具可以重複應用在**多種場景**，而不是單純只能應付單一情況。

### 不靈活的情況

```ts {1-3|5-7}
function identity(arg: string): string {
  return arg;
}

function identity(arg: any): any {
  return arg;
}
```

- 上述的第一種程式碼就只能夠傳入字串並且輸出字串，缺乏彈性。
- 第二種宣告方式雖然擁有彈性，但卻失去型別的意義。

---

# Hello, Generic World

```ts {1,3|2|1-3|5,7|6|5-7|all}
function identity(arg: string): string {
  return arg;
}

function identityGeneric<Type>(arg: Type): Type {
  return arg;
}
```

- 透過generic `Type` 的方法，我們可以透過參數的型別，去決定、操作回傳的型別，提升了靈活性同時又可以保有型別的限制與保障。

---

# 如果我們希望可以在`identityGeneric`的函數中使用`.length`的方法呢？

```ts {monaco}
function loggingIdentity<Type>(arg: Type): Type {
  console.log(arg.length);
  return arg;
}
```

- 當我們這樣做時，編譯器會給我們一個錯誤:<span class="text-red">`Property 'length' does not exist on type 'Type'.`</span>。
- 請記住，我們之前說過這些類型變數代表任所有類型，因此使用此函數的人可能會傳入`number, symbol ...etc`，而<span class="text-green">可能沒有`.length`這種方法</span>。

---

# 解決方法

```ts {1,3|2|all}
function loggingIdentity<Type>(arg: Type[]): Type[] {
  console.log(arg.length);
  return arg;
}
```

- 透過`Type[]`宣告成Array，讓TS知道`arg`是陣列而不是其他沒有`.length`的變數。

### 另一種宣告方式

```ts {monaco}
interface Lengthwise {
  length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
  console.log(arg.length); // Now we know it has a .length property, so no more error
  return arg;
}
```

---

# 什麼時候適合使用Generic(泛型)?

```ts {monaco}
interface Animal {
  name: string;
}

interface Human {
  firstName: string;
  lastName: string;
}

export const getDisplayName = (item: Animal): { displayName: string } => {
  return { displayName: item.name };
};

const resultAnimal = getDisplayName({ name: "dog" });

const resultHuman = getDisplayName({ firstName: "Albert", lastName: "Lin" });
```

- 當你想寫一個可以處理多種數據類型的組件或函數時，大部分的情況下泛型會是一個比較好的選擇。
- 不過也有可能因為使用泛型太過複雜、型別很單一的情況，這時候就應該考量到可讀性。

---

# 承接上一題：

```ts {all|10|12|14-19}
interface Animal {
  name: string;
}

interface Human {
  firstName: string;
  lastName: string;
}

export const getDisplayName = <T extends Animal | Human>(
  item: T
): T extends Human ? { humanName: string } : { animalName: string } => {
  if ("name" in item)
    return { animalName: item.name } as T extends Human
      ? { humanName: string }
      : { animalName: string };
  return { animalName: item.firstName + item.lastName } as T extends Human
    ? { humanName: string }
    : { animalName: string };
};

const resultAnimal = getDisplayName({ name: "dog" });

const resultHuman = getDisplayName({ firstName: "Albert", lastName: "Lin" });
```

---

# 實際上怎麼跟React Component結合呢？

## 這是常見的Component宣告方式

```tsx
interface TableProps {
  items: { id: string }[];
  renderItem: (item: { id: string }) => React.ReactNode;
}

export const Table = (props: TableProps) => {
  return null;
};

const Component = () => {
  return (
    <Table items={[{ id: "1" }]} renderItem={(item) => <div>{item.id}</div>} />
  );
};
```

---

# 實際上怎麼跟React Component結合呢？

## 這是<span class="text-orange border-b">使用了泛型</span>的Component宣告方式

```ts
interface TableProps<TItem> {
  items: TItem[];
  renderItem: (item: TItem) => React.ReactNode;
}

export function Table<TItem>(props: TableProps<TItem>) {
return null;
}

const Component = () => {
return (
<Table items={[{ id: "1" }]} renderItem={(item) => <div>{item.id}</div>} />);
};

```

## [codeSandBox](https://codesandbox.io/p/sandbox/ts-generic-sywntm?layout=%257B%2522sidebarPanel%2522%253A%2522EXPLORER%2522%252C%2522rootPanelGroup%2522%253A%257B%2522direction%2522%253A%2522horizontal%2522%252C%2522contentType%2522%253A%2522UNKNOWN%2522%252C%2522type%2522%253A%2522PANEL_GROUP%2522%252C%2522id%2522%253A%2522ROOT_LAYOUT%2522%252C%2522panels%2522%253A%255B%257B%2522type%2522%253A%2522PANEL_GROUP%2522%252C%2522contentType%2522%253A%2522UNKNOWN%2522%252C%2522direction%2522%253A%2522vertical%2522%252C%2522id%2522%253A%2522clyl2hkl100063b6j4ebq6fbg%2522%252C%2522sizes%2522%253A%255B100%252C0%255D%252C%2522panels%2522%253A%255B%257B%2522type%2522%253A%2522PANEL_GROUP%2522%252C%2522contentType%2522%253A%2522EDITOR%2522%252C%2522direction%2522%253A%2522horizontal%2522%252C%2522id%2522%253A%2522EDITOR%2522%252C%2522panels%2522%253A%255B%257B%2522type%2522%253A%2522PANEL%2522%252C%2522contentType%2522%253A%2522EDITOR%2522%252C%2522id%2522%253A%2522clyl2hkl100023b6jrr8adgok%2522%257D%255D%257D%252C%257B%2522type%2522%253A%2522PANEL_GROUP%2522%252C%2522contentType%2522%253A%2522SHELLS%2522%252C%2522direction%2522%253A%2522horizontal%2522%252C%2522id%2522%253A%2522SHELLS%2522%252C%2522panels%2522%253A%255B%257B%2522type%2522%253A%2522PANEL%2522%252C%2522contentType%2522%253A%2522SHELLS%2522%252C%2522id%2522%253A%2522clyl2hkl100033b6jv5xat56z%2522%257D%255D%252C%2522sizes%2522%253A%255B100%255D%257D%255D%257D%252C%257B%2522type%2522%253A%2522PANEL_GROUP%2522%252C%2522contentType%2522%253A%2522DEVTOOLS%2522%252C%2522direction%2522%253A%2522vertical%2522%252C%2522id%2522%253A%2522DEVTOOLS%2522%252C%2522panels%2522%253A%255B%257B%2522type%2522%253A%2522PANEL%2522%252C%2522contentType%2522%253A%2522DEVTOOLS%2522%252C%2522id%2522%253A%2522clyl2hkl100053b6j97n0f92t%2522%257D%255D%252C%2522sizes%2522%253A%255B100%255D%257D%255D%252C%2522sizes%2522%253A%255B52.95348264828944%252C47.04651735171056%255D%257D%252C%2522tabbedPanels%2522%253A%257B%2522clyl2hkl100023b6jrr8adgok%2522%253A%257B%2522tabs%2522%253A%255B%257B%2522id%2522%253A%2522clyl2hkl000013b6j0946kbmz%2522%252C%2522mode%2522%253A%2522permanent%2522%252C%2522type%2522%253A%2522FILE%2522%252C%2522filepath%2522%253A%2522%252Fsrc%252Findex.tsx%2522%257D%255D%252C%2522id%2522%253A%2522clyl2hkl100023b6jrr8adgok%2522%252C%2522activeTabId%2522%253A%2522clyl2hkl000013b6j0946kbmz%2522%257D%252C%2522clyl2hkl100053b6j97n0f92t%2522%253A%257B%2522tabs%2522%253A%255B%257B%2522id%2522%253A%2522clyl2hkl100043b6j8rs3kjr9%2522%252C%2522mode%2522%253A%2522permanent%2522%252C%2522type%2522%253A%2522UNASSIGNED_PORT%2522%252C%2522port%2522%253A0%252C%2522path%2522%253A%2522%252F%2522%257D%255D%252C%2522id%2522%253A%2522clyl2hkl100053b6j97n0f92t%2522%252C%2522activeTabId%2522%253A%2522clyl2hkl100043b6j8rs3kjr9%2522%257D%252C%2522clyl2hkl100033b6jv5xat56z%2522%253A%257B%2522tabs%2522%253A%255B%255D%252C%2522id%2522%253A%2522clyl2hkl100033b6jv5xat56z%2522%257D%257D%252C%2522showDevtools%2522%253Atrue%252C%2522showShells%2522%253Afalse%252C%2522showSidebar%2522%253Atrue%252C%2522sidebarPanelSize%2522%253A13.263888888888886%257D)

---

# Key Remover

```ts {monaco}
const makeKeyRemover =
  <Key extends string>(keys: Key[]) =>
  <Obj extends Record<Key, any>>(obj: Obj): Omit<Obj, Key> => {
    const result = { ...obj };
    for (const key of keys) {
      delete result[key];
    }
    return result as Omit<Obj, Key>;
  };

const keyRemover = makeKeyRemover(["a", "b"]);
const newObject = keyRemover({ a: 1, b: 2, c: 3 });

console.log(newObject); // { c: 3 }
```

---

# Key Remover

```ts {1-3|2|3|8}
const makeKeyRemover =
  <Key extends string>(keys: Key[]) =>
  <Obj extends Record<Key, any>>(obj: Obj): Omit<Obj, Key> => {
    const result = { ...obj };
    for (const key of keys) {
      delete result[key];
    }
    return result as Omit<Obj, Key>;
  };

const keyRemover = makeKeyRemover(["a", "b"]);
const newObject = keyRemover({ a: 1, b: 2, c: 3 });

console.log(newObject); // { c: 3 }
```
