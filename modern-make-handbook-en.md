# Modern Make Handbook (v. 0.5-en)

[![](https://i.imgur.com/a1MOR8T.png)](https://makefile.site)




[TOC]


<div style="page-break-after: always;"></div>
## 1. Making your library of shortcuts

– What does the cat say when it's hungry?

– Meow!



– What does the dog way when it's warning?

– Woof-woof!



– And what does Bob say when he's about to deploy his project?

– `ansible-playbook -i inventory/production --tags "deploy" app-server.yml -vvv --become-user=app --extra-vars=extra.txt --vault-password-file="~/.ansible/vault.txt"`


This is what we're about to change here.

Our way of changing is easy and simple, just like organizing your closet.

We will open a `Makefile` and put your magical spell in there:

```make
deploy:
	ansible-playbook -i inventory/production --tags "deploy" app-server.yml -vvv --become-user=app --extra-vars=extra.txt --vault-password-file="~/.ansible/vault.txt"
```



Next time Bob only needs to say `make deploy` and the magic happens!

Pretty cool, huh? Now Bob:

- does not have to spend another minute to remember the syntax
- will never use a wrong flag while typing the deploy command
- is not going to freak out next time, when Ansible decides to rename half of their flags



Let's do the same to the rest of our commands.

Почему-то то что изначально задумывалось простым, например `rails server`, часто обрастает дополнительными изнуряющими подробностями: `bundle exec bin/rails server -p 3001 RAILS_ENV=development`. 

At some point you start adding details to your simple commands and your `rails server` turns into something like `bundle exec bin/rails server -p 3001 RAILS_ENV=development`.

And we already know what can we do here:

```make
server:
	bundle exec bin/rails server -p 3001 RAILS_ENV=development
```

Moving on:

```make
logs:
	tail -f log/development.log
```

You've got the idea. A good thing about it is that you don't have to do this all at once. Whenever you find yourself struggling to remember that long command just add it to the `Makefile`!

<div style="page-break-after: always;"></div>
## 2. Overcoming Make weirdness

Bob finally made it to his tests and adds a new line to his `Makefile` to run them conveniently:

```make
test:
	MINITEST_REPORTER=SpecReporter bundle exec bin/rails test
```

But once he tries his new command, a weird thing happens:

```
$: make test
make: `test' is up to date.
```

– "After all it looks like your magical `make` is not *that* magical" - he thinks.


Let's get a few decades back. The original idea of `make` was to generate files, and back these days `make test` would mean that you want to generate a file (or a folder) with the name `test`.

`make` is clever enough to check whether such file (or folder) already exists to avoid doing extra work. But this is not what Bob meant!



To overcome this cleverness of `make` Bob needs to add one magical line into our `Makefile`:

`.PHONY: test`

And if there are several such targets, it's enough to enumerate them one by one using space character:

`.PHONY: app test log doc`



Another surprise comes when you used spaces instead of tabs for indentation:

```
$: make test
Makefile:13: *** missing separator.  Stop.
```

This does not look right. If your text editor can recognize different file formats, it probably already figured out that it should use tabs. Otherwise, you'll need to tweak it a little.

Now, when Bob went down this rabbit hole and learned all the tricks, he can move on.

<!--
<div style="page-break-after: always;"></div>
## 3. Running multiple commands at once

Хотим прогнать тесты, и если все ок, то задеплоить? No problemos:

`make test deploy`

Да, команды можно составлять в длинные цепочки, и если какая-то из команд фэйлится, то остальные запущены не будут.



## 4. Subcommands

В какой-то момент в команде решили что негоже деплоить без запуска тестов и прогон тестов просто захардкодили внутрь команды деплой: 

```make
deploy: test
	ansible-playbook -i inventory/production --tags 'deploy' # ...
```

Т.е. при использовании `make deploy` до деплоя дойдет дело только если пройдут тесты.

<div style="page-break-after: always;"></div>
## 5. Aliases

Семен добавил в Makefile команду для прогона миграций, а Вова все никак не может запомнить как она называется. Иногда он пишет `make dbmigrate`, иногда `make db_migrate`, иногда по привычке вообще `make db:migrate`.

Увы, с последним ничего не поделать. Двоеточия в названиях комманд не поддерживаются. Зато можно смело нафигачить себе алиасов на остальные варианты! Чем Вова и занялся.

Для этого не нужно копипастить нашу длинную команду несколько раз, достаточно записать вызов оригинала после двоеточия:

```make
db-migrate:
	bundle exec bin/rails db:migrate

db_migrate: db-migrate
dbmigrate: db-migrate
```

Придумать годное название для шортката сходу бывает непросто. В таких случая как раз можно насоздавать сразу несколько алиасов, и оставить тот который приживется со временем.

<div style="page-break-after: always;"></div>
## 6. Multiline commands

Со временем Вове надоело набирать такую длинную команду, и он заменил ее на `make db` (не забыв добавить `db` в `PHONY:`)

И все бы классно, но в какой-то момент разработчики на проекте договорились не коммитить в проект `db/schema.rb` (который авто-обновляется после прогона миграций), а это значит что каждый раз после прогона миграций приходилось выполнять команду `make schema-reset`:

```make
schema-reset:
	git checkout HEAD -- db/schema.rb
```

К счастью, никто не запрещает запускать несколько команд под одним шорткатом, и команды Make можно вызывать из Makefile:

```make
db:
	bundle exec bin/rails db:migrate
	make schema-reset

schema-reset:
	git checkout HEAD -- db/schema.rb
```

Единственный минус в том, что Make по умолчанию многословен, и печатает каждую команду прежде чем выполнить:

```
$: make db
bundle exec bin/rails db:migrate 
# ...
make schema-reset
git checkout HEAD -- db/schema.rb
```

К счатью это легко забороть.

<div style="page-break-after: always;"></div>
## 7. Suppressing output

Все что нужно сделать чтобы Make не выводил саму команду, а просто её выполнял, это добавить перед ней символ "@".

Например, так:

```make
hello:
	@echo "Привет, Вова!"
```

Соответственно лишний вывод "make schema-reset" прячем так:

```make
db:
	bundle exec bin/rails db:migrate
	@make schema-reset
```

Ура! 

<div style="page-break-after: always;"></div>
## 8. Ignoring errors

При деплое на staging разрабы тоже решили прогонять тесты:

```make
staging-deploy: 
	@make test
	ansible-playbook -i inventory/staging --tags 'deploy' #...
```

Правда быстро выяснилось, что иногда надо задеплоить, даже если тесты падают! 

Чтобы не выпиливать тесты из сценария, но деплоить несмотря на их результат, можно использовать магический префикс "-":

```make
staging-deploy: 
	-@make test
	ansible-playbook -i inventory/staging --tags 'deploy' #...
```


## 9. Running command only if another one fails

А можно было поступить по другому.

```make
staging-deploy: 
	@make test || echo "Опять Вова поломал тесты!!"
	ansible-playbook -i inventory/staging --tags 'deploy' #...
```

Это даже не фишка Make, это обычный Bash scripting. В результате программа будет каждый раз журить Вову если тесты упали, но и от деплоя отказываться не будет.

<div style="page-break-after: always;"></div>
## 10. Passing arguments

Однажды Вове понадобилось стянуть дамп базы со стэйджинга на свой комп. Пришлось напрячь остатки пямяти и разродиться скриптом:

```make
staging-fetch-dump:
	scp app@staging-server.dev:/path/to/app/db/dump.tgz ./
```

Только вот неплохо бы его сделать чуть более полезным. Вдруг понадобится какой-то еще файл стягивать.

По такому случаю можно передать название файла в качестве аргумента:

```make
staging-fetch:
	scp app@staging-server.dev:/path/to/app/$(F)/ ./
```

Вызов команды теперь будет выглядеть так:

`make staging-fetch F=db/dump.tgz`

<div style="page-break-after: always;"></div>
## 11. Seamless arguments

Однако в случае когда аргумент всего один, было бы классно избавиться от необходимости запоминать название этого самого аргумента и вызывать команду прямо так: `make staging-fetch db/dump.tgz`

Этого можно добиться, но только с помочью черной магии. Надо добавить в Makefile вот такую конструкцию:

```make
ARGS = $(filter-out $@,$(MAKECMDGOALS))
%:
  @:
```

Шорткат при таких раскладах выглядит вот так:

```
staging-fetch:
	scp app@staging-server.dev:/path/to/app/$(ARGS)/ ./
```

Но магия на то и черная, что у неё есть неприятный спецеффект – уже после того как все успешно выполнится, прилетает вот такое сообщение:

```
make: *** No rule to make target 'db/dump.tgz'.  Stop.
```

Можете почитать [подробности того как это работает](https://stackoverflow.com/a/6273809/1334666), чуть ниже вроде даже показывают как победить проблему с ошибкой, но мне это победить не удалось.

Короче говоря, Вова смирился, и решил что можно заплатить такую цену за такую фичу, ну а вам решать самим.

<div style="page-break-after: always;"></div>
## 12. Advanced scripting

Внезапно админы запилили все так, что теперь на каждый фичебранч поднимается по отдельному стэйджингу.

И все бы классно, но теперь в наши крутые шорткаты для работы со стэйджингом придется добавлять по еще одной переменной - имени сервера:

```make
ssh:
	ssh app@$(S)

staging-fetch:
	scp app@$(S):/path/to/app/$(F)/ ./
```

... [TO BE CONTINUED IN PRO VERSION](https://gum.co/makefile-ru): ...

1. [Advanced scripting](https://gum.co/makefile-ru)
2. [Putting things in order](https://gum.co/makefile-ru)
3. [Naming conventions](https://gum.co/makefile-ru)
4. [Full workflow automation](https://gum.co/makefile-ru)
5. [Guiding principles](https://gum.co/makefile-ru)











[![](https://i.imgur.com/MhU79hR.png)](https://makefile.site#yay)

