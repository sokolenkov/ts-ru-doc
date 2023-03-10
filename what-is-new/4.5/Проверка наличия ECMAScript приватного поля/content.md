## Проверка наличия ECMAScript приватного поля

_TYpeScript_ реализовал предложенный _ECMAScript_ механизм проверки наличия приватного поля. Теперь, в теле класса при помощи оператора `in` можно производить проверки на присутствие одноименных приватных полей в других экземплярах. Но стоит сделать акцент, что идентификаторы проверяемых полей должны быть идентичны идентификаторам приватных полей объявленых в самом классе.


`````ts
class Person {
    #name: string;
    constructor(name: string) {
        this.#name = name;
    }

    /**
     * [*] Допустимы проверки только на одноименные
     * приватные поля объявленные в текущем классе!
     */
    equals(other: unknown) {
        return other &&
            typeof other === "object" &&
            #name in other && // [*]
            this.#name === other.#name;
    }
}
`````

Поскольку с помощью оператора `instanceof` невозможно точно выявить принадлежность экземпляра из-за осуществления её на всем дереве иерархии...

`````ts
class A {

}
class B extends A {

}
class C extends B {

}
const c = new C();


console.log(c instanceof A); // [*]

/**
 * [*] Невозможно однозначно интерпретировать экземпляр
 * при помощи оператора instanceof поскольку проверка
 * осуществляется по всей иерархии наследования.
 */
`````

... а строковое представление класса не гарантирует место его объявления...


`````ts
console.log(c.constructor.name === `A`); // [*]

/**
 * [*] это ./a/A.ts или ./b/A.ts?
 */
`````

... механизм определения класса по приватным полям обозначается, как _эргонамичная проверка бренда_, поскольку только приватные поля отличают экземпляры в одном иерархическом древе.
