#!/bin/bash
include .makerc

DOCKER_PHPMYADMIN = ${PROJECT_NAME}-server-phpmyadmin
DOCKER_DB = ${PROJECT_NAME}-server-db
DOCKER_BE = ${PROJECT_NAME}-server-be
DOCKER_NETWORK = ${PROJECT_NAME}-server-network
OS := $(shell uname)

ifeq ($(OS),Linux)
	UID = $(shell id -u)
else
	true = $true
	UID = 1000
endif

help: ## Show this help message
	@echo 'usage: make [target]'
	@echo
	@echo 'targets:'
	@egrep '^(.+)\:\ ##\ (.+)' ${MAKEFILE_LIST} | column -t -c 2 -s ':#'

run: ## Start the containers
	docker network create ${DOCKER_NETWORK} || echo $(true)
	U_ID=${UID} docker-compose up -d

stop: ## Stop the containers
	docker-compose stop

restart: ## Restart the containers
	$(MAKE) stop && $(MAKE) run

initialize-project: ## inicialize the project name, rebuilds all containers, aplication and creates and fills the database
	$(MAKE) init-project-name && $(MAKE) build && $(MAKE) install-yarn && $(MAKE) run-fixtures && $(MAKE) restart

build: ## Rebuilds all the containers
	docker-compose stop && U_ID=${UID} docker-compose build && $(MAKE) run

prepare: ## Runs backend commands
	$(MAKE) composer-install

# Backend commands
yarn-watch:
	docker exec -it --user ${UID} ${DOCKER_BE} yarn build
	docker exec -it --user ${UID} ${DOCKER_BE} yarn watch

composer-install: ## Installs composer dependencies
	docker exec --user ${UID} -it ${DOCKER_BE} composer install --no-scripts --no-interaction --optimize-autoloader

migrations: ## Runs the migrations
	docker exec -it --user ${UID} ${DOCKER_BE} bin/console doctrine:migrations:migrate -n

be-logs: ## Tails the Symfony dev log
	docker exec -it --user ${UID} ${DOCKER_BE} tail -f var/log/dev.log
# End backend commands

ssh-be-root: ## ssh's into the be container
	docker exec -it --user 0 ${DOCKER_BE} bash

ssh-be: ## ssh's into the be container
	docker exec -it --user ${UID} ${DOCKER_BE} bash

ssh-db: ## ssh's into the db container
	docker exec -it ${DOCKER_DB} /bin/bash

ssh-phpmyadmin: ## ssh's into the db container
	docker exec -it ${DOCKER_PHPMYADMIN} /bin/bash

code-style: ## Runs php-cs to fix code styling following Symfony rules
	docker exec -it --user ${UID} ${DOCKER_BE} php-cs-fixer fix src --rules=@Symfony
	docker exec -it --user ${UID} ${DOCKER_BE} php-cs-fixer fix tests --rules=@Symfony

generate-ssh-keys: ## Generates SSH keys for the JWT library
	docker exec -it --user ${UID} ${DOCKER_BE} mkdir -p config/jwt
	docker exec -it --user ${UID} ${DOCKER_BE} openssl genrsa -passout pass:b4a42db9c2995ae84a9e1fe8aae5b95f -out config/jwt/private.pem -aes256 4096
	docker exec -it --user ${UID} ${DOCKER_BE} openssl rsa -pubout -passin pass:b4a42db9c2995ae84a9e1fe8aae5b95f -in config/jwt/private.pem -out config/jwt/public.pem

config-db-user:
	docker exec ${DOCKER_DB} sh -c 'exec mysqldump --all-databases -uroot -p root' < /docker/database/configUser.sql

install-yarn:
	docker exec -it --user ${UID} ${DOCKER_BE} symfony composer install
	docker exec -it --user ${UID} ${DOCKER_BE} yarn install
	docker exec -it --user ${UID} ${DOCKER_BE} yarn build
	docker exec -it --user ${UID} ${DOCKER_BE} symfony console sass:build

init-project-name:
	rm -Rf ./var/cache/ ./var/log/ ./public/build ./node_modules ./var ./vendor
	sed -i 's/symfony6base_db/mprod_${PROJECT_NAME}/g' docker-compose.yml
	sed -i 's/symfony6base_db/mprod_${PROJECT_NAME}/g' .env
	{ grep -RiIl 'symfony6base' --exclude=Makefile --exclude-dir={.git,.idea} & echo docker-compose.yml; } | cat | xargs sed -i 's/symfony6base/${PROJECT_NAME}/g'

run-fixtures:
	docker exec -it --user ${UID} ${DOCKER_BE} symfony console doctrine:migration:migrate --no-interaction
	docker exec -it --user ${UID} ${DOCKER_BE} symfony console app:load-data --no-interaction