# Cпецификация волшебного мира 2: Описание типов

*Перевод статьи [Tom Harding](http://www.tomharding.me): [Fantas, Eel, and Specification 2: Type Signatures](http://www.tomharding.me/2017/03/08/fantas-eel-and-specification-2/). Опубликовано с разрешения автора.*

[Приветствую, путешественник](https://en.wikiquote.org/wiki/Garth_Marenghi's_Darkplace#Once_Upon_A_Beginning_.5B1.1.5D). Я надеюсь, ты хорошо провёл время с того момента как я опубликовал [первую часть этой серии](https://medium.com/devschacht/cпецификация-волшебного-мира-1-daggy-ef332ae68dd8), и я советую прочитать её, прежде чем двигаться дальше. Предполагая, что ты это сделал, есть еще *одна маленькая вещь*, о которой, я думаю, нам стоит поговорить, прежде чем погружаться с головой в спецификацию: **[вывод типов Дамас-Хиндли-Милнера](https://ru.wikipedia.org/wiki/Вывод_типов)**.

Не паникуй.

## Введение: `Java -> Haskell`

Скорее всего, ты видел языки с явно описанной (*статической*) типизацией раньше. Такие как Java:

```java
public static void main(String[] args) {}
```

Эта строчка говорит нам, что функция `main` принимает массив со значениями типа `String` и возвращает `void` (ничего). Это описание типа функции. Описание Дамас-Хиндли-Милнера просто другой способ описания типов. Знаешь что, давай сохраним символы и просто будем называть это описанием типа.

Когда мы пойдём по примерам, держи одну мысль в уме: **все** функции каррированные. Я писал о [каррировании в JavaScript](http://www.tomharding.me/2016/11/12/curry-on-wayward-son/) до этого, так что посмотрите, если вы не уверенны. **TL;DR**, в то время как мы хотели написать объявление функции `add` в Java-подобном языке вот так:

```
public static int add(int a, int b);
//                     ^ arg  ^ arg
//             ^ return
```

Мы хотели написать *каррированное* описание типа вот так:

```
add :: Int -> Int -> Int
--      ^  arg ^ arg
--                    ^ return
```

По-русски, оно говорит, что *наша функция `add` принимает целое число `x` и возвращает функцию, принимающую целое число `y` и возвращающую целое число (вероятно `x + y`).*

> Сейчас, да, некоторым из вас, возможно, будет интересно узнать о **некаррированых** функциях в JavaScript, таких как функция `add` типа `(Int, Int) -> Int`. Тем не менее, мы говорим, что будет функция с одним аргументом, чей единственный аргумент представляет собой пару (**кортеж**†). JavaScript немножко легкомыслен относительно этого, как мы увидим ещё не раз.

Давайте посмотрим на `zipWith`, немного более сложный пример. Вот возможная реализация в JavaScript:

```js
const zipWith = f => xs => ys => {
  const length = Math.min(
    xs.length, ys.length
  )

  const zs = Array(length)

  for (let i = 0; i < length; i++) {
    zs[i] = f(xs[i])(ys[i])
  }

  return zs
}

// Возвращает [ 5, 7 ]
zipWith(x => y => x + y)([1, 2])([4, 5, 6])
```

Наш красивый `zipWith` принимает значения в каждый аргумент из двух массивов (пока самый короткий не закончится), и применяет их к `f`, возвращающей массив результатов. *Если не понятно как эта функция работает, поиграйтесь с несколькими примерами прежде чем продолжать*. Давайте подумаем о типах:

- Функция `f` должна принимать два аргумента двух типов (назовём их `a` и `b`), и это должны быть соответствующие типам `xs`’ и `ys`’ значения массива.
- Тип, возвращаемый `zipWith`, - массив типа возвращаемого `f`. Так, если `f` возвращает некоторый тип `c`, то `zipWith(f)` должна вернуть массив с `c`.

Как мы можем это записать в виде описания типа? Именно так:

```
zipWith :: (a -> b -> c)
  -> [a] -> [b] -> [c]
```

Мы использовали **переменные типа** для отображения мест, где мы можем получить разные типы (возможно вы знаете это как **полиморфизм**). Вам не нужно называть их `a`, `b` и `c`, вы можете просто называть их `x`, `dog` и `jeff` (но нет). *Единственное правило здесь, это то, что переменные типа всегда начинаются с маленькой буквы и конкретные типы всегда начинаются с заглавной*.

Так как мы можем сохранить в переменную `a` *любой* тип (пока наши `f` и `xs` согласны!), мы можем написать описание для `zipWith`, работающее для *любого* типа `a`. Прикольно, да? Фактически, `zipWith` хороший пример инструмента, который мы видим во всём функциональном коде, *потому что* его переменные делают его таким гибким:

```
// a = Int
// b = String
// c = Bool

// Возвращает [ true, false ]
zipWith(x => y => y.length > x)
  ([3, 5])(['Good', 'Bad'])
```

Здесь нашему `a` присвоен `Int`, `b` — `String` и `c` — `Bool`. Однако мы могли бы так же легко сделать их всех `Int` и запихнуть в `x => y => x + y`! Я надеюсь, вы можете сдержать восхищение. Вот последний пример функции с переменной типа:

```
// Фильтруем массив по предикату.
// filter :: (a -> Bool) -> [a] -> [a]
const filter = p => xs => xs.filter(p)
```

Наша функция работает с *любым* `a` до тех пор, пока функция `p` знает как преобразовать `a` в `Bool`. Ура!

Обратите внимание, что оба `filter` и `zipWith` имеют аргумент, который является *функцией*. Для отображения его в описании, мы приютили его описание (обернув в скобки) в нашем общем описании.

Фух! Вот в принципе и все. Мы разделяем каждый тип `->` так, что в конце возвращаемое значение, а все остальные - аргументы. По сути, это *всё*, что вам нужно, чтобы читать и писать описания типов в [Elm](http://elm-lang.org/) — [идите писать на Elm](http://www.tomharding.me/2016/12/11/the-orrery/).

Для *этой* серии, однако, мы собираемся всё усложнить и ввести еще пару вещей...

## Ограничения типа

`zipWith` и `filter` великолепны, потому что их переменные типа могут быть любого типа. Однако иногда мы не имеем такой роскоши. Возможно, нам придется столкнуться с описаниями вроде этих:

```
equals :: Setoid a => a -> a -> Bool
```

`=>` новое обозначение. Это означает, что описание справа верно, если все условия слева соблюдены. В случае с `equals`, описание `a -> a -> Bool` верно, если `a` является `Setoid`. Не беспокойтесь о том, что такое `Setoid`, разберемся с ним в следующей статье. Сейчас просто думайте о `Setoid` как о типе, для которого мы можем проверить эквивалентны ли две его значения.

Ограничения очень важны. Когда у нас есть описание, включающее в себя переменную типа без ограничений, мы знаем, что функция не может управлять ею в любом случае. Мы знаем, глядя на описание `id :: a -> a`, что она может только лишь возвращать значение, которое получила, потому что мы больше ничего не знаем о `a` - оно может быть числом или функцией, или чем угодно! Это подкрепляет идею, называемую параметризацией, к которой мы будем возвращаться несколько раз в этой серии.

В таких языках, как Haskell, компилятор должен убедиться, что условия слева удовлетворяют во время компиляции, которая ловит целый ряд ошибок! Однако для наших целей, мы увидим, что это просто очень удобная документация.

## Пожалуйста, остановись

Это не будет некоторой интерпертацией JavaScript, не так ли? Потому что JavaScript такой какой есть, мы стараемся создавать наши типы с методами, которые определены на прототипе:

```
// Вместо этого:
equals(first)(second)

// Мы делаем это:
first.equals(second)
```

Это, конечно, аккуратнее. Однако, это портит наши красивые описания, потому что `equals` теперь не функция с двумя аргументами, а метод с одним аргументом, прикреплённый к значению. Помните, однако, что аргумент должен иметь тот же тип, что и объект, к которому `equals` прикреплён. В Fantasy Land вы увидите следующий стиль, используемый для этого выражения:

```
equals :: Setoid a => a ~> a -> Bool
```

Здесь `~>` новый символ. Он означает, что `equals` метод слева от `~>`, а справа — его описание. Вернёмся к предыдущей статье, мы видим метод `List.prototype.toArray`. В стиле Fantasy Land, мы напишем описание для этого метода так:

```
// toArray :: List a ~> [a]
List.prototype.toArray = function () {
  return this.cata({
    Cons: (x, acc) => [
      x, ... acc.toArray()
    ],

    Nil: () => []
  })
}
```

Мы говорим, что `List` со значениями типа `a` имеет метод под названием `toArray`, возвращающий массив значений с типом `a`. Это может быть не очень красиво, дорогой читатель, но это JavaScript. Если вы хотите сделать маленькое упражнение, напишите описание типа для `List.prototype.map`, и убедитесь, что оно простое на сколько это возможно!

## `finish :: Blog ~> Ending`

Я обещаю вам, на этом всё. Это всё, что вам надо знать, чтобы жить полноценной жизнью как функциональный программист. Как только вы привыкнете к такому синтаксису, это просто как езда на велосипеде со странными стрелками и скобками. Если эта статья показалась немного тяжелой, не волнуйтесь: просто вернитесь к ней для справки, если позднее у вас возникнут вопросы по серии.

Независимо от того, готовьтесь. Ни на что не отвлекаемся. Следующая остановка: Fantasy Land.

Берегите себя ♥

- - - -

*\* Важный момент здесь заключается в том, что эквивалентность - это гораздо глубже, чем равенство. Просто попробуйте набрать `(x => x) === (x => x)` в консоли Node; для функций, чтобы быть валидным сетоидом, должно вернуться `true`.*

*† Упорядоченный набор фиксированной длины.*

- - - -

*Слушайте наш подкаст в [iTunes](https://itunes.apple.com/ru/podcast/девшахта/id1226773343) и [SoundCloud](https://soundcloud.com/devschacht), читайте нас на [Medium](https://medium.com/devschacht), контрибьютьте на [GitHub](https://github.com/devSchacht), общайтесь в [группе Telegram](https://t.me/devSchacht), следите в [Twitter](https://twitter.com/DevSchacht) и [канале Telegram](https://t.me/devSchachtChannel), рекомендуйте в [VK](https://vk.com/devschacht) и [Facebook](https://www.facebook.com/devSchacht).*

[Статья на Medium](https://medium.com/devschacht/tom-harding-fantas-eel-and-specification-2-type-signatures-c9b2e45dea71)

- - - -

*В оригинальном названии статьи используется непереводимая игра слов, основанная на схожести звучания названия спецификации Fantasy Land*
