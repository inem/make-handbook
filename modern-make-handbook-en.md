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

<div style="page-break-after: always;"></div>
## 3. Running multiple commands at once

What if we want to run the tests, and if they pass deploy our code? No problemos:

`make test deploy`

You've guessed right, you can specify multiple targets when running `make` and all of them will be executed in provided order. If one of them fails, the rest is not be executed.



## 4. Subcommands

At some point your team decides that tests should always be run before deploying. To make sure everyone does it, they hardcoded this inside of the `deploy` target:

```make
deploy: test
	ansible-playbook -i inventory/production --tags 'deploy' # ...
```

This means that each time you run `make deploy`, `make test` is called first. And only if it succeeds, `deploy` will be executed.

<div style="page-break-after: always;"></div>
## 5. Aliases

Alice added a new target to the `Makefile` to run database migrations, but Bob keeps forgetting its name. He tries `make dbmigrate`, `make db_migrate`, even `make db:migrate` that always used to work before.

Unfortunately the latter will not work, because `make` does not support colon signs in target names. However you can have aliases for the rest!

To do this Bob does not have to copy the long command to each of these targets. All what he needs to do is to specify the name of the original target after a colon sign:

```make
db-migrate:
	bundle exec bin/rails db:migrate

db_migrate: db-migrate
dbmigrate: db-migrate
```

Sometimes coming up with a handy name for a shortcut can be tricky. In such cases you can create multiple aliases and keep the most used one after a while.

<div style="page-break-after: always;"></div>
## 6. Multiline commands

One day Bob decided to automate his workflow even further and replaced his long command with a shorter version `make db`, and also added `db` to `.PHONY:`.

So far so good, but on the next day the team decided to move `db/schema.rb` outside of their revision control system. But this file is updated automatically, so they also had to start running `make schema-reset` every time they run a migration.

```make
schema-reset:
	git checkout HEAD -- db/schema.rb
```

Luckily there is a way to run multiple targets in one command by calling `make` from within the `Makefile`:

```make
db:
	bundle exec bin/rails db:migrate
	make schema-reset

schema-reset:
	git checkout HEAD -- db/schema.rb
```

The downside of this approach is that by default `make` is verbose and outputs each command before executing it:

```
$: make db
bundle exec bin/rails db:migrate 
# ...
make schema-reset
git checkout HEAD -- db/schema.rb
```

The good news are that this is very easy to fix!



<div style="page-break-after: always;"></div>
## 7. Suppressing output

To stop `make` from printing out commands before execution just prefix it with `@`, i.e.

```make
hello:
	@echo "Hi, Bob!"
```

In case of our story with `make schema-reset` all we need to do is:

```make
db:
	bundle exec bin/rails db:migrate
	@make schema-reset
```

Yay!

<!--
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

