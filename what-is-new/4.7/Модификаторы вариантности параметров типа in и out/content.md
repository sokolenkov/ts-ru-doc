## Модификаторы вариантности параметров типа in и out

Идентификация данных осуществляется при помощи типов и предназначена для предотвращения попадания в операции неподходящих для них значений. Каждая операция имеет свое низкоуровневое описание содержащее, в том числе и тип, к которому должно принадлежать пригодное для неё значение. Сверка типа значения с этим типом называется процессом выявления совместимости. Совместимость осуществляется по правилам вариантности, которые бывают четырех видов. Но прежде, чем рассмотреть каждый из них, ненадолго отвлечемся на _TypeScript_.
Поскольку _TypeScript_ реализует _структурную типизацию_, тип проще всего представить в виде обычного листка бумаги, который может содержать имена (идентификаторы) ассоциированные с какими-либо другими типами. Идентификатор + ассоциированный с ним тип = признак типа. Именно на основе этих признаков и осуществляется процесс выявления совместимости речь о которой пойдет сразу после того освещения ещё одной очень простой темы - иерархии наследования.
Представляя иерархию наследования в голове сразу вырисовывается картина из мира номинативной типизации, что неосознанно выбивает из колеи структурной. Поэтому сразу стоит сосредоточиться на интересующей нас детали - логическом обозначении иерархических отношений. Дело в том, что иерархия направлена сверху вниз, а значит более базовый тип расположен выше, чем его подтип. Таким образом, в логических выражениях представляющих иерархию базовые типы обозначаются, как большие (`>`) по отношению к своим подтипам. И наоборот. То есть, `SuperType > SubType` и `SubType < SuperType`. Но упомяну ещё раз, в структурной типизации нет понятия иерархия наследования, поскольку при сравнении берутся в расчет признаки типов, а не ссылки (`ref`). Но чтобы не забивать особо голову просто возьмем за правило, что тип, который обладает всеми признаками другого типа и кроме этого содержит дополнительные, будет считаться подтипом, а значит, в логических выражениях будет обозначаться меньшим (`<`).

`````ts
interface A {
    f0: boolean;
}
interface B {
    f0: boolean;
    f1: number;
}
interface С {
    f0: boolean;
    f1: number;
}

// A > B или B < A
// A > C или C < A
// B = C или C = B
`````

Это очень просто и это знают все, но повторить все равно стоило, так как именно логические выражения нам помогут разобраться в количестве видов вариантов совместимости. Теперь, на основе полученной информации давайте рассмотрим варианты по которым может происходить проверка на совместимость.

И так, вариантов всего четыре и каждый из них имеет собственное название. Но чтобы было более понятно рассмотрим сценарий, когда переменной принадлежащей к типу `A` может быть присвоено значение с типом `B`.

`Ковариантность` предполагает, что проверка на совместимость завершится успехом в случаи, когда `B < A` или `B = A` (в номинативной типизации `B` подтип `A` ). `Контрвариантность` предполагает, что `B > A` или `B = A` (в номинативной бы это звучало, как базовый тип можно совместим с подтипом или самим собой, но не наоборот). `Инвариантность`, это когда совместимы исключительно при условии `B = A`. `Бивариантность` подразумевает `B < A`, `B > A` или `B = A`, то есть - все предыдущие варианты в одном.

А теперь к сути дела. В _TypeScript_ все типы проверяются на совместимость по ковариантным правилам, за исключением параметров функций, которые контрвариантны. Поскольку различные правила на сложных рекурсивных типах требуют дорогостоящие вычисления, _TypeScript_ реализует механизм явного аннотирования параметров типа при с помощью необязательных модификаторов `in` и `out`.

`in` указывает, что параметр типа ковариантен, а `out` контрвариантен. Но стоит сделать акцент на том, что с помощью этих модификаторов невозможно изменить правила по которым _TypeScript_ производит вычисления совместимости, а можно лишь их конкретизировать.

`````ts
type Setter<T> = (param: T) => void;
type Getter<T> = () => T;

/**
 * Стандартный код.
 * При сравнении двух сеттеров параметры в сигнатуре будут проверятся по контрвариантным правилам, а для геттеров возвращаемые типы по ковариантным.
 */
`````
`````ts
type Setter<in T> = (param: T) => void;
type Getter<out T> = () => T;

/**
 * [Код с модификаторами]
 * Правила будут идентичны предыдущему примеру. Разница лишь в явной конкретизации.
 */
`````
`````ts
type Setter<out T> = (param: T) => void; // [0]
type Getter<in T> = () => T; // [1]

/**
 * [Код с модификаторами]
 * К тому же, нельзя изменить поведение, то есть - нельзя поменять модификаторы местами!
 */

/**
 * [0]
 * Type 'Setter<sub-T>' is not assignable to type 'Setter<super-T>' as implied by variance annotation.
 *  Types of parameters 'param' and 'param' are incompatible.
 *   Type 'super-T' is not assignable to type 'sub-T'.ts(2636)
 */

/**
 * [1]
 * Type 'Getter<super-T>' is not assignable to type 'Getter<sub-T>' as implied by variance annotation.
 *  Type 'super-T' is not assignable to type 'sub-T'.ts(2636)
 */
`````

Проще всего воспринимать эти модификаторы, как указание на то, что тип будет использоваться во входных параметрах (`<in T>`) или выходных (`<out T>`) или и то и другое одновременно `<in out T>`.

`````ts
/**
 * Указание на то, что тип используется во входных и в выходных параметрах. 
 */
type Func<in out T> = (param: T) => T;
`````
