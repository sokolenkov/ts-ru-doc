## Отключение исключения неиспользуемого импорта

Не так давно, в _TypeScript_ был реализован механизм предназначеный для исключения импортов для неиспользуемых констркций, но затем выяснилось, что в некоторых случаях, это поведение является нежелатильным. К примеру, известный фраймворк `vuejs` выполняет импорт конструкций в теле тегов `<script></script>`, а использует их в шаблоне описывающем структуру компонентов.

`````ts
<script setup>
import { handler } from "./handlers.ts";
</script>

<button @click="handler">Нажми меня!</button>
`````

Поскольку _TypeScript_ проверяет исключительно блок в котором расположен код, импорт функции-слушателя будет исключон из сборке, что неизбежно приведет в ошибке во время выполнения.

Во избежания подобных ситуаций был введен новый флаг `--preserveValueImports`, который отключает удаление неиспользуемого импорта. Единственный важный момент заключается в том, что его необходимо использовать вместе с флагом `--isolatedModules`, которому задано значение `true`.
