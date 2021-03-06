## Дополнение Ё - Как создаётся Rust и “Nightly Rust”

Это дополнение рассказывает как создаётся Rust, и как это влияет на Вас как на разработчика.

### Стабильность без стагнации

Как язык, Rust *много* заботиться о стабильности Вашего кода. Мы хотим чтобы Rust был прочным фундаментом, вашей опорой, и если бы все постоянно менялось, это было бы невозможно. В то же время, если мы не можем экспериментировать с различными возможностями, мы не можем обнаружить важные проблемы до релиза, когда мы не можем их изменить.

Нашим решением проблемы является “стабильность без стагнации”, и наш руководящий принцип: Вы никогда не должны бояться перехода на новую стабильную версию Rust. Каждое обновление должно быть безболезненным, но также должно добавлять новые функции, меньше дефектов и более быструю скорость компиляции.

### Ту-ту! Каналы выпуска и поездка на поезде

Разработка языка Rust работает по принципу *расписания поездов*. То есть, вся разработка совершается в ветке `master` Rust репозитория. Выпуски следуют модели последовательного выпуска продукта (software release train), которая была использована Cisco IOS и другими программными продуктами. Есть три *канала выпуска* Rust:

- Ночной (Nightly)
- Бета (Beta)
- Стабильный (Stable)

Большинство Rust разработчиков используют стабильную версию, но те кто хотят попробовать экспериментальные новые функции, должны использовать Nightly или Beta.

Приведём пример, как работает процесс разработки и выпуска новых версий. Давайте предположим, что команда Rust работает над версией Rust 1.5. Его релиз состоялся в декабре 2015 года, но это даст реалистичность номера версии. Была добавлена новая функциональность в Rust: новые коммиты в ветку `master`. Каждую ночь выпускается новая ночная версия Rust. Каждый день является днём выпуска ночной версии и эти выпуски создаются нашей структурой автоматически. По мере того как идёт время, наши выпуски выглядят так:

```text
nightly: * - - * - - *
```

Каждые шесть недель наступает время подготовки новой Beta версии! Ветка `beta` Rust репозитория ответвляется от ветки `master`, используемой версией Nightly. Теперь мы имеем два выпуска:

```text
nightly: * - - * - - *
                     |
beta:                *
```

Многие пользователи Rust не используют активно бета-версию, но тестируют бета-версию в их системе CI для помощи Rust обнаружить проблемы обратной совместимости. В это время каждую ночь выпускается новая версия Nightly:

```text
nightly: * - - * - - * - - * - - *
                     |
beta:                *
```

Предположим, что была найдена регрессия. Хорошо, что мы можем протестировать бета-версию перед тем как регрессия попала в стабильную версию! Исправление отправляется в ветку `master`, поэтому версия nightly исправлена и затем исправление также направляется в ветку `beta`, и происходит новый выпуск бета-версии:

```text
nightly: * - - * - - * - - * - - * - - *
                     |
beta:                * - - - - - - - - *
```

Через шесть недель после выпуска бета-версии, наступает время для выпуска стабильной версии! Ветка `stable` создаётся из ветки `beta`:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |
beta:                * - - - - - - - - *
                                       |
stable:                                *
```

Ура! Rust 1.5 выпущена! Но мы также забыли про одну вещь: так как прошло шесть недель, мы должны выпустить бета-версию *следующей*  версии Rust 1.6. Поэтому после ответвления ветки `stable` из ветки `beta`, следующая версия `beta` ответвляется снова от `nightly`:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |                         |
beta:                * - - - - - - - - *       *
                                       |
stable:                                *
```

Это называется “модель поезда” (train model), потому что каждые шесть недель выпуск “покидает станцию”, но ему все ещё нужно пройти канал beta, чтобы попасть в стабильную версию.

Rust выпускается каждые шесть недель, как часы. Если вы знаете дату одного выпуска Rust, вы знаете дату выпуска следующего: это шесть недель позднее. Хорошим аспектом выпуска версий каждые шесть недель является то, что следующий поезд прибывает скоро. Если какая-то функция не попадает в релиз, не надо волноваться: ещё один выпуск произойдёт очень скоро! Это помогает снизить давление в случае если функция возможно не отполирована к дате выпуска.

Благодаря этому процессу, вы всегда можете посмотреть следующую версию Rust и убедиться, что на неё легко будет перейти: если бета-выпуск будет работать не так как ожидалось, вы можете сообщить об этом разработчикам и он будет исправлен перед выпуском стабильной версии! Поломки в бета-версии случаются относительно редко, но `rustc` все ещё является частью программного обеспечения, поэтому дефекты все ещё существуют.

### Нестабильные функции

У этой модели выпуска есть ещё один плюс: нестабильные функции. Rust использует технику называемую “флаги функционала” (feature flags) для определения функций, которые были включены в выпуске. Если новая функция находится в активной разработке, она попадает в ветку `master`, и поэтому попадает в ночную версию, но с *флагом функции* (feature flag). Если как пользователь, вы хотите попробовать работу такой функции, находящейся в разработке, вы должны использовать ночную версию Rust и указать в вашем исходном коде определённый флаг.

Если вы используете бета или стабильную версию Rust, Вы не можете использовать флаги функций. Этот ключевой момент позволяет использовать на практике новые возможности перед их стабилизацией. Это может использоваться желающими идти в ногу со временем, а другие могут использовать стабильную версию и быть уверенными что их код не сломается. Стабильность без стагнации.

Эта книга содержит информацию только о стабильных возможностях, так как разрабатываемые возможности продолжают меняться в процессе и несомненно они будут отличаться в зависимости от того, когда эта книга написана и когда эти возможности будут включены в стабильные сборки. Вы можете найти информацию о возможностях ночной версии в интернете.

### Rustup и роль ночной версии Rust

Rustup делает лёгким изменение между различными каналами Rust, на глобальном или локальном для проекта уровне. По умолчанию устанавливается стабильная версия Rust. Для установки ночной версии выполните команду:

```console
$ rustup toolchain install nightly
```

Вы можете также увидеть все установленные *инструменты разработчика (toolchains)* (версии Rust и ассоциированные компоненты) с помощью `rustup`. Это пример вывода у одного из авторов Rust с компьютером на Windows:

```powershell
> rustup toolchain list
stable-x86_64-pc-windows-msvc (default)
beta-x86_64-pc-windows-msvc
nightly-x86_64-pc-windows-msvc
```

Как видите, стабильный набор инструментов (toolchain) используется по умолчанию. Большинство пользователей Rust используют стабильные версии большую часть времени. Возможно, вы захотите использовать стабильную большую часть времени, но использовать каждую ночную версию в конкретном проекте, потому что заботитесь о передовых возможностях. Для этого вы можете использовать команду `rustup override` в каталоге этого проекта, чтобы установить ночной набор инструментов, должна использоваться команда `rustup`, когда вы находитесь в этом каталоге:

```console
$ cd ~/projects/needs-nightly
$ rustup override set nightly
```

Теперь каждый раз, когда вы вызываете `rustc` или `cargo` внутри *~/projects/needs-nightly*, `rustup` будет следить за тем, чтобы вы используете ночную версию Rust, а не стабильную по умолчанию. Это очень удобно, когда у вас есть множество Rust проектов!

### Процесс RFC и команды

Итак, как вы узнаете об этих новых возможностях? Модель разработки Rust следует *процессу запроса комментариев (RFC - Request For Comments)*. Если хотите улучшить Rust, вы можете написать предложение, которое называется RFC.

Любой может написать RFC для улучшения Rust, предложения рассматриваются и обсуждаются командой Rust, которая состоит из множества тематических подгрупп. На [веб-сайте Rust](https://www.rust-lang.org/governance) есть полный список команд, который включает команды для каждой области проекта: дизайн языка, реализация компилятора, инфраструктура, документация и многое другое. Соответствующая команда читает предложение и комментарии, пишет некоторые собственные комментарии и в конечном итоге, приходит к согласию принять или отклонить эту возможность.

Если новая возможность принята и кто-то может реализовать её, то задача открывается в репозитории Rust. Человек реализующий её, вполне может не быть тем, кто предложил эту возможность! Когда реализация готова, она попадает в `master` ветвь с флагом функции, как мы обсуждали в разделе ["Нестабильных функциях"](#unstable-features)<!--  -->.

Через некоторое время, разработчики Rust использующие ночные выпуски, смогут опробовать новую возможность, члены команды обсудят её, как она работает в ночной версии и решат, должна ли она попасть в стабильную версию Rust или нет. Если принимается решение двигать её вперёд, ограничение функции с помощью флага убирается и функция теперь считается стабильной! Она едет в новую  стабильную версию Rust.
