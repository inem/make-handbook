# Modern Make Handbook (v. 0.5-en)

[![](https://i.imgur.com/a1MOR8T.png)](https://makefile.site)




[TOC]


<div style="page-break-after: always;"></div>
## 1. Making your library of shortcuts

– What cats say when they’re hungry?

– Meow!



– What dogs say when they smell danger?

– Woof-woof!



– And what does Bob say when he’s deploying project?

– `ansible-playbook -i inventory/production --tags “deploy” app-server.yml -vvv --become-user=app --extra-vars=extra.txt --vault-password-file="~/.ansible/vault.txt"`


This is what we’re going to fix.

It is not gonna be hard. It will be easy and pleasant process. Just like organizing a closet.

Open `Makefile` and put your mega-command in there:

```make
deploy:
	ansible-playbook -i inventory/production --tags “deploy” app-server.yml -vvv --become-user=app --extra-vars=extra.txt --vault-password-file="~/.ansible/vault.txt"
```


Next time Bob will have to say just `make deploy.

Cool, huh?  It means that

- Bob saves time because he doesn't have to remember all the details of the deploy command
- Bob will never make a mistake in the deploy command
- Bob is not going to freak out, when Ansible renames half of their flags


Let’s move on.


Often, things that supposed to be simple, like `rails server`,  owergrow with addtional debilitating details: `bundle exec bin/rails server -p 3001 RAILS_ENV=development`. 

Luckily we already know what to do::

```make
server:
	bundle exec bin/rails server -p 3001 RAILS_ENV=development
```

and:

```make
logs:
	tail -f log/development.log
```

Savor it: `make deploy`, `make server`, `make logs`

Of course, Bob commits the Makefile to the repository, so that his colleagues can use the shortcuts too!



<div style="page-break-after: always;"></div>
## 2. Overcoming Make weirdness

Inspired, Bob keeps cleaning up:

```make
test:
	MINITEST_REPORTER=SpecReporter bundle exec bin/rails test
```

Then some weirdness happens:

```
$: make test
make: `test’ is up to date.
```

— "Looks like something is very wrong with your Make!” - he thinks.

Original purpose of Make is to automate complex builds for C/C++ projects. So, the semantics of `make test` assumes that `test` directory should be generated as a result.

If such directory exists, Make assumes there's no need to execute anything. This is exactly what happened, since every Rails project has `test` directory.

To persuade Make, Bob has to add one magical rule to `Makefile`:

`.PHONY: test`

If there are multiple commands like this,  we can add them all:

`.PHONY: app test log doc`


Another surprise is that Make is very picky about indentation. It refuses to work if you use spaces:

```
$: make test
Makefile:13: *** missing separator. Stop.
```

If your editor detects the file format correctly, you don't have to do anything. If not, then just configure it accordingly.


Now Bob is now warned, he knows how to avoid common problems, so we can move on.

<div style="page-break-after: always;"></div>
## 3. Running multiple commands at once

What if we want to run the tests, and if they pass, then deploy our code? No problemo:

`make test deploy`

Yup, you can combine commands in long chains. If one fails, the rest of them going to be skipped.


## 4. Subcommands

At some point, Bob's team decides that tests execution should be a part of deployment process, so they just hardcoded `test` command into deploy instructions:

```make
deploy: test
	ansible-playbook -i inventory/production --tags ‘deploy’ # ...
```
This means that each time you run `make deploy`, `make test` is called first. And only if it succeeds, `deploy` will be executed.

<div style="page-break-after: always;"></div>
## 5. Aliases

Alice adds a new command to run migrations, but Bob keeps forgetting how it is called. He run `make dbmigrate`, `make db_migrate`, and even `make db:migrate`, but kept getting error: 

```
make: *** No rule to make target '*'.  Stop.
```

We can fix this problem with aliases! Unfortunately, the latter will not work, because `make` command names can't contain colons. But for the rest of the typos we can do it easily, even without copying and pasting:

```make
db-migrate:
	bundle exec bin/rails db:migrate

db_migrate: db-migrate
dbmigrate: db-migrate
```

Sometimes it is hard to come up with handy name for a shortcut. In this case just create a bunch of aliases and keep the most used one after a while.

<div style="page-break-after: always;"></div>
## 6. Multiline commands

After a while, the team decided to replace `make db-migrate` with just `make db`, which is impossible to forget.
Of course, they added `db` to `.PHONY:`, because Rails projects have `db`directory as well.

So far so good, but on the next day the team decides that they not going to commit `db/schema.rb` anymore, but delegate it to CI system. The problem is that Rails generates the new version of `schema.rb` every time you run migrations.

Not a big problem actually:

```make
schema-reset:
	git checkout HEAD -- db/schema.rb
```


Luckily, we can run multiple commands under one shortcut, and you can nest make shortcuts:

```make
db:
	bundle exec bin/rails db:migrate
	make schema-reset

schema-reset:
	git checkout HEAD -- db/schema.rb
```

Make prints each command before executing it, so the aoutput is a bit verbose:

```
$: make db
bundle exec bin/rails db:migrate 
# ...
make schema-reset
git checkout HEAD -- db/schema.rb
```

The good news is that it is very easy to fix!



<div style="page-break-after: always;"></div>
## 7. Making Make less verbose

When you don't want a command to be printed, and just want Make to execute it, prepend it with `@`:

```make
hello:
	@echo “Hi, Bob!”
```

So, in our case:

```make
db:
	bundle exec bin/rails db:migrate
	@make schema-reset
```

Yay!


<div style="page-break-after: always;"></div>
## 8. Ignoring errors

Bob’s team decides to run tests before deploying to staging as well:

```make
staging-deploy: 
	@make test
	ansible-playbook -i inventory/staging --tags ‘deploy’ #...
```

However, sometimes you have to deploy to staging even if tests are failing.

To keep tests running, but still deploy even if they do not pass, we add another magical prefix: `-`.

```make
staging-deploy: 
	-@make test
	ansible-playbook -i inventory/staging --tags ‘deploy’ #...
```


## 9. Running command only if another one fails

There is one more way to achieve the same effect:

```make
staging-deploy: 
	@make test || echo “Looks like Bob broke tests again!!”
	ansible-playbook -i inventory/staging --tags ‘deploy’ #...
```

It's even not a feature of Make - it's a regular shell scripting. It will condemn Bob for broken tests every time they fail, but will deploy the project anyway.

If we don’t want lower Bob's self-esteem, we can do it like this:

```make
staging-deploy: 
	@make test || true
	ansible-playbook -i inventory/staging --tags ‘deploy’ #...
```

<div style="page-break-after: always;"></div>
## 10. Passing arguments

Bob had to download a database dump from the staging server to his local machine. He decides to add it as a shortcut:

```make
staging-fetch-dump:
	scp app@staging-server.dev:/path/to/app/db/dump.tgz ./
```

A good start, but how about to make it more useful, so we could use it to download any file?

We can pass a filename as argument:

```make
staging-fetch:
	scp app@staging-server.dev:/path/to/app/$(F)/ ./
```

And we call it like this:

`make staging-fetch F=db/dump.tgz`


<div style="page-break-after: always;"></div>
## 11. Seamless arguments

What if we could simplify the command by skipping the name of the argument, since we have only one here?

Could we just do `make staging-fetch db/dump.tgz`?

The answer is yes! We can do it with a bit of black magic. We have to add the following statement into the `Makefile`:

```make
ARGS = $(filter-out $@,$(MAKECMDGOALS))
%:
	@:
```

Your shortcut will turn into something that looks like this:

```
staging-fetch:
	scp app@staging-server.dev:/path/to/app/$(ARGS)/ ./
```

We called it "balck magic" for a reason. This trick has one annoying side effect. After the script is executed, you get an error message like this:

```
make: *** No rule to make target ‘db/dump.tgz’. Stop.
```

Here is a [post on StackOverflow](https://stackoverflow.com/a/6273809/1334666) that explains how it works and even contains a fix for this problem. Although, Bob couldn‘t make it work, but he decided that this is the price he is ready to pay.


<div style="page-break-after: always;"></div>
## 12. Advanced scripting

Another day DevOps team announces that for now on each feature branch will be deployed to a separate staging server.

Sounds great, but now for all our staging-related shortcuts we have to specify additional variable – URL of the server:

```make
ssh:
  ssh app@$(S)

staging-fetch:
  scp app@$(S):/path/to/app/$(F)/ ./
```

... [TO BE CONTINUED IN PRO VERSION](https://gum.co/makefile-ru): ...

1. [Advanced scripting](https://gum.co/makefile-ru)
2. [Putting things in order](https://gum.co/makefile-ru)
3. [Naming conventions](https://gum.co/makefile-ru)
4. [Full workflow automation](https://gum.co/makefile-ru)
5. [Guiding principles](https://gum.co/makefile-ru)











[![](https://i.imgur.com/MhU79hR.png)](https://makefile.site#yay)

