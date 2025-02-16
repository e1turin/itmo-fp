# Лабораторные работы по курсу «Функциональное программирование»

## Материалы по учебному курсу

- лекционные материалы: 
   - http://fp.edu.swampbuds.me/ 
      - [web.archive.org](https://web.archive.org/web/20240904134050/http://fp.edu.swampbuds.me/)
- билеты на экзамен:
   - https://aalexuser.github.io/Functional-programming-exam

## Дополнительные материалы про Idris

1. Записи лекций мини-курса «Функциональное программирование на языке Idris»
   (Виталий Брагилевский) – https://www.youtube.com/playlist?list=PLEqoHzpnmTfD8ocGHDAMUfxTtchqSvrWn
   - курс посвящен изучению **первой** версии языка, вторая несколько отличается
2. Туториал по **второй** версии языка от создателя пакетного менеджера
   `pack` – https://github.com/stefan-hoeck/idris2-tutorial. Особенно оплезные
   - Установка через `pack` (самый простой способ, но работает только на Linux и/или в Docker)
     – [Getting Started with pack and Idris2](https://github.com/stefan-hoeck/idris2-tutorial/blob/main/src/Appendices/Install.md)
   - Настройка работы в Neovim (на Windows трудно сейчас установить LSP, так что проще использовать VS Code в devcontainer'е)
     – [Interactive Editing in Neovim](https://github.com/stefan-hoeck/idris2-tutorial/blob/main/src/Appendices/Neovim.md)
3. Официальный туториал – https://idris2.readthedocs.io/en/latest/tutorial/index.html
   - Важное отличие второй версии Idris от первой – это [QTT aka Multipliplicities](https://idris2.readthedocs.io/en/latest/tutorial/multiplicities.html)
     - Доклад [«Idris 2: Quantitative Types in Action - Edwin Brady»](https://youtu.be/0uA-tKR6Ah4)
4. Инструкция по работе с пакетныйм менеджером `pack` – https://github.com/stefan-hoeck/idris2-pack
5. Документация к языку на официальном сайте (полезно смотреть документацию к API)
   – https://www.idris-lang.org/pages/documentation.html


# Комментарии

## Пакетный менеджер idris2-pack

Пакетный менеджер позволяет одной командой создать модуль с инициализацией git-репозитори и `.gitignore`. Но чтобы норм использовать свои пакеты нужно немного переработать структуру. Можно использовать `pack new lib <lib-name>`, а потом перетащить файлы и изменить пути.

Структура проекта должна выглядеть так:
- repository
   - my-package-1
      - build -- temporary folder with binaries, ignored in git
      - src
         - PublicModule1.idr:
            ```idris
            module PublicModule1
            ...
            ```
         - PublicModule2.idr
         - PrivateModule.idr
      - test
         - build -- temporary folder with testing binaries, ignored in git
         - src
            - Main.idr:
               ```idris
               module Main

               import PublicModule1 -- module for testing
               import Hedgehog      -- property-based testing lib
               ...
               ```
         - my-package-1-test.ipkg:
            ```ipkg
            ...
            depends = my-package-1 -- testing module
                     , hedgehog    -- property-based testing lib
            ...
            ```
      - my-package-1.ipkg:
         ```ipkg
         ...
         modules = PublicMadule1  -- not specified PrivateModule.idr
                  , PublicModule2
         ...
         ```
   - my-package-2
   - ... -- another packages
   - .gitignore:
      ```gitignore
      **/build/**/*
      *.*~
      ```
   - pack.toml:
      ```toml
      [custom.all.my-package-1]
      type = "local"
      path = "my-package-1"                # path to directory
      ipkg = "my-package-1.ipkg"           # path to .ipkg from `path`
      test = "test/my-package-1-test.ipkg" # path to test's .ipkg from `paht`

      ... -- another packages
      ```

### idris2-lsp interop

idris2-lsp использует для решения зависимостей пакеты установленные в его дефолтные пути, поэтому возникают проблемы, когда пакет установлен через pack в локальный *"репозиторий"*, что LSP не может найти нужную зависимость указанную в `.ipkg` файле. Например, такое возникает в файлах тестов пакета `my-package-1-test`, которые зависят от `my-package-1`. Для решения проще всего установить пакет в системный репозиторий:

```sh
idris2 --install my-package-1/my-package-1.ipkg
```

Потом можно перезапустить LSP или сбросить буфер файла, мб перезапустить редактор. Кроме того, может потребоваться для всех изменений в зависимости выполнять переустановку.

По какой-то причине `idris2-lsp` не воспринимает пакеты, которые установил
`pack` в свои пути (`$PACK_DIR`) Похоже, что LSP и pack не получается так
просто совместить, по этому поводу даже есть ишью:

- https://github.com/idris-community/idris2-lsp/issues/219 и
- https://github.com/stefan-hoeck/idris2-pack/issues/292.

Там предложили в качестве временной меры добавить в переменные окружения для LSP
пути используемые в pack (см.
[idris2-lsp_fix-pack](https://github.com/e1turin/itmo-fp/blob/docker/idris2-lsp_fix-pack)).
- > P.S. такое решение по какой-то причине не работает в Docker :(

## Stype Guide

Для Idris2 есть неофициальный Style Guide:
- https://github.com/stefan-hoeck/idris2-style-guide (от создателя `pack`)
- или другой https://github.com/expede/idris-styleguide.

Так же можно использовать готовый `.editorconfig` (с расширением VS Code
`EditorConfig.EditorConfig`) из репозитория Idris2:
```sh
wget https://raw.githubusercontent.com/idris-lang/Idris2/refs/heads/main/.editorconfig
# or
curl -L -O https://raw.githubusercontent.com/idris-lang/Idris2/refs/heads/main/.editorconfig
# or
echo -e 'GET /idris-lang/Idris2/refs/heads/main/.editorconfig HTTP/1.0\nHost: raw.githubusercontent.com\n\n' | openssl s_client -quiet -connect raw.githubusercontent.com:443 2>/dev/null | sed '0,/^\s*$/d' > .editorconfig
```