# Спецификация Fantasy Land

[![Build Status](https://travis-ci.org/fantasyland/fantasy-land.svg?branch=master)](https://travis-ci.org/fantasyland/fantasy-land) [![Join the chat at https://gitter.im/fantasyland/fantasy-land](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/fantasyland/fantasy-land?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

(aka "Спецификация Алгебраический JavaScript")

<img src="logo.png" width="200" height="200" />

Этот проект определяет взаимодействие общих алгебраических структур:

* [Setoid](#setoid)
* [Semigroup](#semigroup)
* [Monoid](#monoid)
* [Functor](#functor)
* [Contravariant](#contravariant)
* [Apply](#apply)
* [Applicative](#applicative)
* [Alt](#alt)
* [Plus](#plus)
* [Alternative](#alternative)
* [Foldable](#foldable)
* [Traversable](#traversable)
* [Chain](#chain)
* [ChainRec](#chainrec)
* [Monad](#monad)
* [Extend](#extend)
* [Comonad](#comonad)
* [Bifunctor](#bifunctor)
* [Profunctor](#profunctor)

<img src="figures/dependencies.png" width="888" height="289" />

## Общее

Алгебра представляет собой набор значений, набор операторов, которые зависимы и должны подчиняться некоторым законам.

Каждая алгебра Fantasy Land – это отдельная спецификация. Алгебра может иметь зависимости от реализаций других алгебр.

## Терминология

1. "значение" любые JavaScript значения, включая структуры определенные ниже.
2. "эквивалент" - подходящее определение эквивалентности для заданного значения. Определение должно гарантировать, что эти два значения могут быть безопасно переставлены в программе, что подтверждает абстракцию. Например:
- Два списка эквивалентны, если они эквивалентны по всем показателям.
- Два обычных JavaScript объекта, представленные как словари, эквивалентны, если они эквивалентны по всем ключам.
- Два промиса эквивалентны, когда они возвращают эквивалентные значения.
- Две функции эквивалентны, если они дают эквивалентные результаты для эквивалентных входных данных.

## Префиксы имен методов

Для тех типов данных, которые должны быть совместимы с Fantasy Land, значения должны иметь определенные свойства. Эти свойства имеют префикс `fantasy-land/`.

Например:

```js
//  MyType#fantasy-land/map :: MyType a ~> (a -> b) -> MyType b
MyType.prototype['fantasy-land/map'] = ...
```

Далее в этом документе имена без префиксов используются только для уменьшения шума.

Для удобства вы можете использовать пакет `fantasy-land`:

```js
var fl = require('fantasy-land')

// ...

MyType.prototype[fl.map] = ...

// ...

var foo = bar[fl.map](x => x + 1)
```

## Представление типа

Определенные виды поведения определяются с точки зрения представления типа. Другие поведения не требуют представления. Таким образом, определенные алгебры требуют тип для предоставления значения-уровня представления (с определенными свойствами). Тип Identity, например, может обеспечить `Id` в качестве своего представителя типа: `Id :: TypeRep Identity`.

Если тип предоставляет представителя типа, каждый представитель типа должен иметь свойство `constructor`, которое является ссылкой на представителя типа.

## Алгебры

### Setoid

1. `a.equals(a) === true` (рефлексивность)
2. `a.equals(b) === b.equals(a)` (симметрия)
3. Если `a.equals(b)` и  `b.equals(c)`, то `a.equals(c)` (транзитивность)

#### Метод `equals`

```hs
equals :: Setoid a => a ~> a -> Boolean
```

Значение Setoid должно предоставить метод `equals`. Метод `equals` принимает один аргумент:

    a.equals(b)

1. `b` должен быть значением того же Setoid

    1. Если `b` не тот же Setoid, поведение `equals` не определено (рекомендуется возвращать `false`)

2. `equals` должен возвращать логическое значение (`true` или `false`)

### Semigroup

1. `a.concat(b).concat(c)` эквивалентно `a.concat(b.concat(c))` (ассоциативность)

#### Метод `concat`

```hs
concat :: Semigroup a => a ~> a -> a
```

Значение Semigroup должно предоставить метод `concat`. Метод `concat` принимает один аргумент:

    s.concat(b)

1. `b` должен быть значением того же Semigroup

    1. Если `b` не тот же Semigroup, поведение `concat` не определено

2. `concat` должен возвращать значение того же Semigroup

### Monoid

Значение, которое реализует спецификацию Monoid, должно также реализовать спецификацию [Semigroup](#semigroup).

1. `m.concat(M.empty())` эквивалентно `m` (точный справа)
2. `M.empty().concat(m)` эквивалентно `m` (точный слева)

#### Метод `empty`

```hs
empty :: Monoid m => () -> m
```

Значение Monoid должно предоставить метод `empty` в её [представлении типа](#Представление-типа):

    M.empty()

Получив значение `m`, можно получить представление типа через свойство `constructor`:

    m.constructor.empty()

1. `empty` должен возвращать значение того же Monoid

### Functor

1. `u.map(a => a)` эквивалентно `u` (точный)
2. `u.map(x => f(g(x)))` эквивалентно `u.map(g).map(f)` (композиция)

#### Метод `map`

```hs
map :: Functor f => f a ~> (a -> b) -> f b
```

Значение Functor должно предоставить метод `map`. Метод `map` принимает один аргумент:

    u.map(f)

1. `f` должен быть функцией

    1. Если `f` не функция, поведение `map` не определено
    2. `f` может вернуть любое значение
    3. Значения возвращенные `f` проверять не надо

2. `map` должен возвращать значение того же Functor

### Contravariant

1. `u.contramap(a => a)` эквивалентно `u` (точный)
2. `u.contramap(x => f(g(x)))` эквивалентно `u.contramap(f).contramap(g)` (композиция)

#### Метод `contramap`

```hs
contramap :: Contravariant f => f a ~> (b -> a) -> f b
```

Значение Contravariant должно предоставить метод `contramap`. Метод `contramap` принимает один аргумент:

    u.contramap(f)

1. `f` должен быть функцией

    1. Если `f` не функция, поведение `contramap` не определено
    2. `f` может вернуть любое значение
    3. Значения возвращенные `f` проверять не надо

2. `contramap` должен возвращать значение того же Contravariant

### Apply

Значение, которое реализует спецификация Apply, должно также реализовывать спецификацию [Functor](#functor).

1. `v.ap(u.ap(a.map(f => g => x => f(g(x)))))` эквивалентно `v.ap(u).ap(a)` (композиция)

#### Метод `ap`

```hs
ap :: Apply f => f a ~> f (a -> b) -> f b
```

Значение Apply должно предоставить метод `ap`. Метод `ap` принимает один аргумент:

    a.ap(b)

1. `b` должен быть функцией

    1. Если `b` не является функцией, поведение `ap` не определено

2. `a` должен быть любым значением Apply

3. `ap` должны применять функцию Apply `b` со значением Apply `a`

    1. Значения возвращенные этой функцией проверять не надо

### Applicative

Значение, которое реализует спецификация Applicative, должно также реализовывать спецификацию [Apply](#apply).

1. `v.ap(A.of(x => x))` эквивалентно `v` (точный)
2. `A.of(x).ap(A.of(f))` эквивалентно `А.(Ф(Х))` (гомоморфизм)
3. `A.of(y).ap(u)` эквивалентно `u.ap(A.of(f => f(y)))` (перестановка)

#### Метод `of`

```hs
of :: Applicative f => a -> f a
```

Значение Applicative должно предоставить метод `of` в её [представлении типа](#Представление-типа). Функция `of` принимает один аргумент:

    F.of(a)

Получив значение `f`, можно получить представление типа через свойство `constructor`:

    f.constructor.of(a)

1. `of` должен вернуть значение того же Applicative

    1. Значения возвращенные `a` проверять не надо

### Alt

Значение, которое реализует спецификацию Alt, должно также реализовать спецификацию [Functor](#functor).

1. `a.alt(b).alt(c)` эквивалентно `a.alt(b.alt(c))` (ассоциативность)
2. `a.alt(b).map(f)` эквивалентно `a.map(f).alt(b.map(f))` (распределённость)

#### Метод `alt`

```hs
alt :: Alt f => f a ~> f a -> f a
```

Значение Alt должно предоставить метод `alt`. Метод `alt` принимает один аргумент:

    a.alt(b)

1. `b` должен быть значением того же Alt

    1. Если `b` не тот же Alt, поведение `alt` не определено
    2. `a` и `b` могут содержать любое значение того же типа
    3. Значения содержащиеся в `a` и `b` проверять не надо

2. `alt` должен вернуть значение того же Alt

### Plus

Значение, которое реализует спецификацию Plus, также должно реализовать спецификацию [Alt](#alt).

1. `x.alt(A.zero())` эквивалентно `x` (точный справа)
2. `A.zero().alt(x)` эквивалентно `x` (точный слева)
3. `A.zero().map(f)` эквивалентно `A.zero()` (упразднение)

#### Метод `zero`

```hs
zero :: Plus f => () -> f a
```

Значение Plus должно предоставить метод `zero` в её [представлении типа](#Представление-типа):

    A.zero()

Получив значение `x`, можно получить представление типа через свойство `constructor`:

    x.constructor.zero()

1. `zero` должен возвращать значение того же Plus

### Alternative

Значение, которое реализует спецификацию Alternative, должно также реализовать спецификации [Applicative](#applicative) и [Plus](#plus).

1. `x.ap(f.alt(g))` эквивалентно `x.ap(f).alt(x.ap(g))` (распределённость)
2. `x.ap(A.zero())` эквивалентно `A.zero()` (упразднение)

### Foldable

1. `u.reduce` эквивалентно `u.reduce((acc, x) => acc.concat([x]), []).reduce`

#### Метод `reduce`

```hs
reduce :: Foldable f => f a ~> ((b, a) -> b, b) -> b
```

Значение Foldable должно предоставить метод `reduce`. Метод `reduce` принимает два аргумента:

    u.reduce(f, x)

1. `f` должен быть двоичной функцией

    1. Если `f` не функция, поведение `reduce` не определено
    2. Первый аргумент `f` должен быть такого же типа, как `x`
    3. `f` должен возвращать значение такого же типа, как `x`
    4. Значения возвращенные `f` проверять не надо

2. `x` - это первоначальное аккумулятивное значение для преобразования

    1. `x` проверять не надо

### Traversable

Значение, которое реализует спецификацию Traversable, должно также реализовывать спецификации [Functor](#functor) и [Foldable](#foldable).

1. `t(u.traverse(F, x => x))` эквивалентно `u.traverse(G, t)` для любого `t` такого, что `t(a).map(f)` эквивалентно `t(a.map(f))` (нормализованность)

2. `u.traverse(F, F.of)` эквивалентно `F.of(u)` для любой Applicative `F` (точный)

3. `u.traverse(Compose, x => new Compose(x))` эквивалентно `new Compose(u.traverse(F, x => x).map(x => x.traverse(G, x => x)))` для `Compose`, определенных ниже, и любых Applicatives `F` и `G` (композиция)

```js
var Compose = function(c) {
  this.c = c;
};

Compose.of = function(x) {
  return new Compose(F.of(G.of(x)));
};

Compose.prototype.ap = function(f) {
  return new Compose(this.c.ap(f.c.map(u => y => y.ap(u))));
};

Compose.prototype.map = function(f) {
  return new Compose(this.c.map(y => y.map(f)));
};
```

#### Метод `traverse`

```hs
traverse :: Applicative f, Traversable t => t a ~> (TypeRep f, a -> f b) -> f (t b)
```

Значение Traversable должно предоставить метод `traverse`. Метод `traverse` принимает два аргумента:

    u.traverse(A, f)

1. `A` должен быть [представлением типа](#Представление-типа) Applicative

2. `f` должен быть функцией, которая возвращает значение

    1. Если `f` не функция, поведение `traverse` не определено
    2. `f` должен возвращать значение типа, представленного `A`

3. `traverse` должен возвращать значение типа представленного `A`

### Chain

Значение, которое реализует спецификацию Chain, должно также реализовывать спецификацию [Apply](#apply).

1. `m.chain(f).chain(g)` эквивалентно `m.chain(x => f(x).chain(g))` (ассоциативность)

#### Метод `chain`

```hs
chain :: Chain m => m a ~> (a -> m b) -> m b
```

Значение Chain должно предоставить метод `chain`. Метод `chain` принимает один аргумент:

    m.chain(f)

1. `f` должен быть функцией, которая возвращает значение

    1. Если `f` не функция, поведение `chain` не определено
    2. `f` должен возвращать значение того же Chain

2. `chain` должен возвращать значение того же Chain

### ChainRec

Значение, которое реализует спецификацию ChainRec, должно реализовать спецификацию [Chain](#chain).

1. `M.chainRec((next, done, v) => p(v) ? d(v).map(done) : n(v).map(next), i)` эквивалентно `(function step(v) { return p(v) ? d(v) : n(v).chain(step); }(i))` (эквивалентность)
2. Использование `M.chainRec(f, i)` должно быть максимально подобным самостоятельному вызову `f`

#### Метод `chainRec`

```hs
chainRec :: ChainRec m => ((a -> c, b -> c, a) -> m c, a) -> m b
```

Значение ChainRec должно предоставить метод `chainRec` в его [представлении типа](#Представление-типа). Метод `chainRec` принимает два аргумента:

    M.chainRec(f, i)

Получив значение `m`, можно получить представление типа через свойство `constructor`:

    m.constructor.chainRec(f, i)

1. `f` должен быть функцией, которая возвращает значение

    1. Если `f` не функция, поведение `chainRec` не определено
    2. `f` принимает три аргумента `next`, `done`, `value`

        1. `next` - это функция, которая принимает один аргумент того же типа, как `i` и может вернуть любое значение
        2. `done` - это функция, которая принимает один аргумент и возвращает тот же тип, возвращаемое значение `next`
        3. `value` - это значение такого же типа, как `i`

    3. `f` должен возвращать значение того же ChainRec, который содержит значение, возвращаемое из `done` или `next`

2. `chainRec` должен возвращать значение того же ChainRec, который содержит то же значение, что и аргумент `done`

### Monad

Значение, которое реализует спецификацию Monad, должно также реализовать спецификации [Applicative](#applicative) and [Chain](#chain).

1. `M.of(a).chain(f)` эквивалентно `f(a)` (точный слева)
2. `m.chain(M.of)` эквивалентно `m` (точный справа)

### Extend

Значение, которое реализует спецификацию Extend, должно реализовать спецификацию [Functor](#functor).

1. `w.extend(g).extend(f)` эквивалентно `w.extend(_w => f(_w.extend(g)))`

#### Метод `extend`

```hs
extend :: Extend w => w a ~> (w a -> b) -> w b
```

Значение Extend должно предоставить метод `extend`. Метод `extend` принимает один аргумент:

     w.extend(f)

1. `f` должен быть функцией, которая возвращает значение

    1. Если `f` не функция, поведение `extend` неопределено
    2. `f` должен возвращать значение типа `v`, для некоторой переменной `v`, содержащейся в `w`
    3. Значения возвращенные `f` проверять не надо

2. `extend` должен возвращать значение того же Extend

### Comonad

Значение, которое реализует спецификацию Comonad, должно реализовать спецификацию [Extend](#extend).

1. `w.extend(_w => _w.extract())` эквивалентно `w` (точный слева)
2. `w.extend(f).extract()` эквивалентно `f(w)` (точный справа)

#### Метод `extract`

```hs
extract :: Comonad w => w a ~> () -> a
```

Значение Comonad должно предоставить метод `extract`. Метод `extract` не принимает никаких аргументов:

    w.extract()

1. `extract` должен возвращать значение типа `v`, для некоторой переменной `v`, содержащиеся в `w`

    1. `v` должен иметь тот же тип, что возвращает `f` в `extend`

### Bifunctor

Значение, которое реализует спецификацию Bifunctor, должно также реализовать спецификацию [Functor](#functor).

1. `p.bimap(a => a, b => b)` эквивалентно `р` (точный)
2. `p.bimap(a => f(g(a)), b => h(i(b))` эквивалентно `p.bimap(g, i).bimap(f, h)` (композиция)

#### Метод `bimap`

```hs
bimap :: Bifunctor f => f a c ~> (a -> b, c -> d) -> f b d
```

Значение Bifunctor должно предоставить метод `bimap`. Метод `bimap` принимает два аргумента:

    c.bimap(f, g)

1. `f` должен быть функцией, которая возвращает значение

    1. Если `f` не функция, поведение `bimap` не определено
    2. `f` может вернуть любое значение
    3. Значение, возвращенное `f`, проверять не надо

2. `g` должен быть функцией, которая возвращает значение

    1. Если `g` не функция, поведение `promap` не определено
    2. `g` может возвращать любое значение
    3. Значение, возвращенное `g`, проверять не надо

3. `bimap` должен возвращать значение того же Bifunctor

### Profunctor

Значение, которое реализует спецификацию Profunctor, должно также реализовать спецификацию [Functor](#functor).

1. `p.promap(a => a, b => b)` эквивалентно `р` (точный)
2. `p.promap(a => f(g(a)), b => h(i(b)))` эквивалентно `p.promap(f, i).promap(g, h)` (композиция)

#### Метод `promap`

```hs
promap :: Profunctor p => p b c ~> (a -> b, c -> d) -> p a d
```

Значение Profunctor должно предоставить метод `promap`. Метод `profunctor` принимает два аргумента:

    c.promap(f, g)

1. `f` должен быть функцией, которая возвращает значение

    1. Если `f` не функция, поведение `promap` не определено
    2. `f` может вернуть любое значение
    3. Значение, возвращенное `f`, проверять не надо

2. `g` должен быть функцией, которая возвращает значение

    1. Если `g` не функция, поведение `promap` не определено
    2. `g` может возвращать любое значение
    3. Значение, возвращенное `g`, проверять не надо

3. `promap` должен возвращать значение того же Profunctor

## Реализации

При создании типов данных, которые удовлетворяют нескольким алгебрам, авторы могут выбрать для реализации определенных методов другие методы. Реализации:


  - [`map`][] может быть реализован с помощью [`ap`][] и [`of`][]:

    ```js
    function(f) { return this.ap(this.of(f)); }
    ```

  - [`map`][]может быть реализован с помощью [`chain`][] и [`of`][]:

    ```js
    function(f) { return this.chain(a => this.of(f(a))); }
    ```

  - [`map`][] может быть реализован с помощью [`bimap`]:

    ```js
    function(f) { return this.bimap(a => a, f); }
    ```

  - [`map`][] может быть реализован с помощью [`promap`]:

    ```js
    function(f) { return this.promap(a => a, f); }
    ```

  - [`ap`][] может быть реализован с помощью [`chain`][]:

    ```js
    function(m) { return m.chain(f => this.map(f)); }
    ```

  - [`reduce`][] может быть реализован так:

    ```js
    function(f, acc) {
      function Const(value) {
        this.value = value;
      }
      Const.of = function(_) {
        return new Const(acc);
      };
      Const.prototype.map = function(_) {
        return this;
      };
      Const.prototype.ap = function(b) {
        return new Const(f(b.value, this.value));
      };
      return this.traverse(x => new Const(x), Const.of).value;
    }
    ```

  - [`map`][] может быть реализован так:

    ```js
    function(f) {
      function Id(value) {
        this.value = value;
      };
      Id.of = function(x) {
        return new Id(x);
      };
      Id.prototype.map = function(f) {
        return new Id(f(this.value));
      };
      Id.prototype.ap = function(b) {
        return new Id(this.value(b.value));
      };
      return this.traverse(x => Id.of(f(x)), Id.of).value;
    }
    ```

Если тип данных реализует метод, который *может* быть реализован, его поведение должно быть эквивалентно реализации (или реализациям).

## Примечания

1. Если есть больше, чем один способ реализации методов и законов, реализация должна выбрать один и представить обертку для других применений.
2. Не рекомендуется перегружать спецификации методов. Можно легко в результате получить сломанное и неправильное поведение.
3. Рекомендуется выбрасывать предупреждение на незадокументированное применение.
4. Контейнер `Id`, реализующий множество методов, доступен в `internal/id.js`.

[`ap`]: #ap-method
[`bimap`]: #bimap-method
[`chain`]: #chain-method
[`concat`]: #concat-method
[`empty`]: #empty-method
[`equals`]: #equals-method
[`extend`]: #extend-method
[`extract`]: #extract-method
[`map`]: #map-method
[`of`]: #of-method
[`promap`]: #promap-method
[`reduce`]: #reduce-method
[`sequence`]: #sequence-method

## Альтернативы

Существует также [Спецификация Static Land](https://github.com/rpominov/static-land)
