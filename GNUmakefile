
# The repo may contain a Makefile generated by ./configure
# We include Makefile at the end of this GNUmakefile
# Make searches for this file first first per make default search hierarchy
ifneq ($(Makefile),)
	Makefile := defined
endif

# If you see pwd_unknown showing up, this is why. Re-calibrate your system.
PWD ?= pwd_unknown

# Note the different service configs in  docker-compose.yml.
# We override this default for different build/run configs
SERVICE_TARGET := shell

ifeq ($(user),)
# We force root
HOST_USER := root
HOST_UID  := 0
## USER retrieved from env, UID from shell.
HOST_USER ?= $(strip $(if $(USER),$(USER),nodummy))
HOST_UID  ?=  $(strip $(if $(shell id -u),$(shell id -u),4000))
else
# We force root
#HOST_USER := root
#HOST_UID  := 0
## allow override by adding user= and/ or uid=  (lowercase!).
## uid= defaults to 0 if user= set (i.e. root).
HOST_USER = $(user)
HOST_UID = $(strip $(if $(uid),$(uid),0))
endif

#GITHUB CONFIG
GITHUB_USER_NAME=$(git config user.name)
export GITHUB_USER_NAME
GITHUB_USER_EMAIL=$(git config user.email)
export GITHUB_USER_EMAIL


BITCOIN_DATA  := $(HOME)/.bitcoin
export BITCOIN_DATA


# PROJECT_NAME defaults to name of the current directory.
# should not need to be changed if you follow GitOps operating procedures.
PROJECT_NAME := $(notdir $(PWD))
export PROJECT_NAME
REPO := $(notdir $(PWD))
export REPO

#These are referenced here and docker-compose.yml
DOCKERFILE         := $(notdir $(PWD))
export DOCKERFILE

THIS_FILE := $(lastword $(MAKEFILE_LIST))
CMD_ARGUMENTS ?= $(cmd)

ifeq ($(did),)
    D_ID   := $(1)
    export D_ID
else
    D_ID   ?= $(did)
    export D_ID
endif

ifeq ($(port),)
    PUBLIC_PORT   := 80
else
    PUBLIC_PORT   ?= $(port)
endif

# export such that its passed to shell functions for Docker to pick up.
# control alpine version from here
VERSION := 3.11.6
export VERSION
export HOST_USER
export HOST_UID

#VERBOSE=--verbose
VERBOSE=	
export VERBOSE

# all our targets are phony (no files to check).
.PHONY: help slim all init shell build-shell rebuild-shell service logini concat concat-all build-all run-all rerun-all make-statoshi run-statoshi extract concat-slim build-slim rebuild-slim run-slim concat-gui build-gui rebuild-gui run-gui test-gui autogen depends config doc concat package-all package-gui package-slim d-ps d-images d-exec torproxy get-branches

# suppress make's own output
#.SILENT:

help:
	@echo ''
	@echo ''
	@echo '	Docker: make [TARGET] [EXTRA_ARGUMENTS]'
	@echo '	Shell:'
	@echo '		make shell user=root'
	@echo '		make shell user=$(HOST_USER)'
	@echo ''
	@echo '	[TARGET]:'
	@echo ''
	@echo '		run-slim	deploy build-slim product'
	@echo ''
	@echo '	[EXTRA_ARGUMENTS]: push a shell command to the container'
	@echo ''
	@echo '		cmd=:	'
	@echo '		     	    make run-all cmd="bitcoind"'
	@echo '		d=:  	'
	@echo '		     	    make shell   d="--prune=550"'
	@echo ''
	@echo ''

# git fetch branches
git-fetch-branches:
	@echo ''
	bash -c 'git remote set-branches origin "*" &  git fetch -v '
	@echo ''

# Some handy docker commands
d-ps:
	@echo ''
	bash -c 'docker ps'
	@echo ''
d-images:
	@echo ''
	bash -c 'docker images'
	@echo ''
d-exec:
		bash -c './docker/d-exec.sh'
cli-exec:
		bash -c './docker/cli-exec.sh'

#######################
# Backup $HOME/.bitcoin
########################
TIME=$(shell date +%s)
export TIME
#backup:
#	@echo ''
#	bash -c 'mkdir -p $(HOME)/.bitcoin'
##	bash -c 'conf/get_size.sh'
#	bash -c 'tar czv --exclude=*.log --exclude=banlist.dat \
#			--exclude=fee_exstimates.dat --exclude=mempool.dat \
#			--exclude=peers.dat --exclude=.cookie --exclude=database \
#			--exclude=.lock --exclude=.walletlock --exclude=.DS_Store\
#			-f $(HOME)/.bitcoin-$(TIME).tar.gz $(HOME)/.bitcoin'
#	bash -c 'openssl md5 $(HOME)/.bitcoin-$(TIME).tar.gz > $(HOME)/bitcoin-$(TIME).tar.gz.md5'
#	bash -c 'openssl md5 -c $(HOME)/bitcoin-$(TIME).tar.gz.md5'
#	@echo ''

#######################
# Shared volume for datadir persistance
########################
init:
	@echo 'init'
	git config --global core.editor vim
	@echo ''
	install -v ./conf/usr/local/bin/stats /usr/local/bin
	install -v ./conf/usr/local/bin/stats-* /usr/local/bin
	@echo ''
	mkdir -p $(BITCOIN_DATA)
	./conf/config.bitcoin.conf.sh
	install -v ./conf/bitcoin.conf $(BITCOIN_DATA)/bitcoin.conf
	install -v ./conf/additional.conf $(BITCOIN_DATA)/additional.conf
	bash -c ' install -v ./docker/docker-compose.yml .'
	@echo ''
#######################
# Docker file creation...
########################
.PHONY: all
all: init
	@echo 'concat-all'
	bash -c '$(pwd) cat ./docker/header               > $(DOCKERFILE)'
	bash -c '$(pwd) cat ./docker/statoshi.all        >> $(DOCKERFILE)'
	bash -c '$(pwd) cat ./docker/footer              >> $(DOCKERFILE)'
	bash -c '$(pwd) cat ./docker/torproxy             > torproxy'
	bash -c '$(pwd) cat ./docker/shell                > shell'
	make run
	@echo ''
#######################
.PHONY: slim
slim: init
	@echo 'concat-slim'
	bash -c '$(pwd) cat ./docker/header               > $(DOCKERFILE)'
	bash -c '$(pwd) cat ./docker/statoshi.slim       >> $(DOCKERFILE)'
	bash -c '$(pwd) cat ./docker/footer              >> $(DOCKERFILE)'
	bash -c '$(pwd) cat ./docker/torproxy             > torproxy'
	bash -c '$(pwd) cat ./docker/shell                > shell'
	make run
	@echo ''
#######################
build-shell: all
	@echo 'build-shell'
	docker-compose --verbose build shell
	@echo ''
#######################
rebuild-shell: all
	@echo 'rebuild-shell'
	docker-compose --verbose build --no-cache shell
	@echo ''
#######################
shell: build-shell
	@echo 'shell'
ifeq ($(CMD_ARGUMENTS),)
	# no command is given, default to shell
	@echo ''
	docker-compose --verbose -p $(PROJECT_NAME)_$(HOST_UID) run --rm shell sh
	@echo ''
else
	# run the command
	@echo ''
	docker-compose --verbose -p $(PROJECT_NAME)_$(HOST_UID) run --rm shell sh -c "$(CMD_ARGUMENTS)"
	@echo ''
endif
#######################
autogen: all
	@echo 'autogen'
	# here it is useful to add your own customised tests
	@echo ''
	docker-compose --verbose -f docker-compose.yml -p $(PROJECT_NAME)_$(HOST_UID) run --rm statoshi sh -c "cd $(REPO)  && ./autogen.sh && exit"
	@echo ''
#######################
config: autogen
	@echo 'config'
	docker-compose --verbose -f docker-compose.yml -p $(PROJECT_NAME)_$(HOST_UID) run --rm statoshi sh -c "cd $(REPO)  && ./configure --disable-wallet --disable-tests --disable-hardening --disable-man --enable-util-cli --enable-util-tx --with-gui=no --without-miniupnpc --disable-bench && exit"
	@echo ''
#######################
depends: config
	@echo 'depends'
	docker-compose --verbose -f docker-compose.yml -p $(PROJECT_NAME)_$(HOST_UID) run --rm statoshi sh -c "apk add coreutils && exit"
	@echo ''
	docker-compose --verbose -f docker-compose.yml -p $(PROJECT_NAME)_$(HOST_UID) run --rm statoshi sh -c "cd $(REPO) && make -j $(nproc) download -C $(REPO)/depends && exit"
	@echo ''
#######################
test:
	@echo 'test'
	docker-compose --verbose -f docker-compose.yml -p $(PROJECT_NAME)_$(HOST_UID) run -d --rm shell sh -c '\
	echo "I am `whoami`. My uid is `id -u`." && echo "Docker runs!"' \
	&& echo success
	@echo ''
#######################
build:
	@echo 'build'
	docker-compose $(VERBOSE) -f docker-compose.yml build statoshi
	@echo ''
#######################
run: build
	@echo 'run'
ifeq ($(CMD_ARGUMENTS),)
	@echo ''
	docker-compose $(VERBOSE) -f docker-compose.yml -p $(PROJECT_NAME)_$(HOST_UID) run -d --publish $(PUBLIC_PORT):3000 --publish 8125:8125 --publish 8126:8126 --publish 8333:8333 --publish 8332:8332 --rm statoshi sh
	@echo ''
else
	@echo ''
	docker-compose --verbose -f docker-compose.yml -p $(PROJECT_NAME)_$(HOST_UID) run -d --publish $(PUBLIC_PORT):3000 --publish 8125:8125 --publish 8126:8126 --publish 8333:8333 --publish 8332:8332 --rm statoshi sh -c "$(CMD_ARGUMENTS)"
	@echo ''
endif
	@echo 'Give grafana a few minutes to set up...'
	@echo 'http://localhost:$(PUBLIC_PORT)'
#######################
extract: concat
	@echo 'extract'
	#extract TODO CREATE PACKAGE for distribution
	sed '$d' $(DOCKERFILE) | sed '$d' | sed '$d' > $(DOCKERFILE_EXTRACT)
	docker build -f $(DOCKERFILE_EXTRACT) --rm -t $(DOCKERFILE_EXTRACT) .
	docker run --name $(DOCKERFILE_EXTRACT) $(DOCKERFILE_EXTRACT) /bin/true
	docker rm $(DOCKERFILE_EXTRACT)
	rm -f  $(DOCKERFILE_EXTRACT)
#######################
torproxy: concat-all
	@echo 'concat-all'
#REF: https://hub.docker.com/r/dperson/torproxy
	bash -c 'docker run -it -p 8118:8118 -p 9050:9050 -p 9051:9051 -d dperson/torproxy'
#	@echo ''
#	docker-compose --verbose  -f docker-compose.yml -p $(PROJECT_NAME)_$(HOST_UID) run --publish 8118:8118  --publish 9050:9050  --publish 9051:9051 --rm torproxy sh -c "$(CMD_ARGUMENTS)"
#	@echo ''

#######################
get-branches:
	@echo 'get-branches'
		bash -c ./conf/usr/local/bin/get-branches.sh
#######################
clean:
	@echo 'clean'
	@docker-compose -p $(PROJECT_NAME)_$(HOST_UID) down --remove-orphans --rmi all 2>/dev/null \
	&& echo 'Image(s) for "$(PROJECT_NAME):$(HOST_USER)" removed.' \
	|| echo 'Image(s) for "$(PROJECT_NAME):$(HOST_USER)" already removed.'
	rm -f $(DOCKERFILE)*
	rm -f shell
	rm -f gui
	rm -f statoshi
	rm -f stats.build.*
#######################
prune:
	@echo 'prune'
	docker system prune -af
#######################
doc:
	@echo 'doc'
	docker system prune -af
	export HOST_USER=root
	export HOST_UID=0
	bash -c '$(pwd) make user=root'
	bash -c 'cat README > README.md'
	bash -c 'cat ./docker/Docker.md >> README.md'
	bash -c '$(pwd) make user=root  >> README.md'
#######################
package-all: clean build-all
	bash -c 'cat ~/GH_TOKEN.txt | docker login docker.pkg.github.com -u RandyMcMillan --password-stdin'
	bash -c 'docker tag $(PROJECT_NAME):$(HOST_USER) docker.pkg.github.com/bitcoincore-dev/stats.bitcoincore.dev/$(notdir $(PWD)):$(VERSION)'
	bash -c 'docker push docker.pkg.github.com/bitcoincore-dev/stats.bitcoincore.dev/$(notdir $(PWD)):$(VERSION)'
#######################
package-gui: clean build-gui
	bash -c 'cat ~/GH_TOKEN.txt | docker login docker.pkg.github.com -u RandyMcMillan --password-stdin'
	bash -c 'docker tag $(PROJECT_NAME):$(HOST_USER) docker.pkg.github.com/bitcoincore-dev/stats.bitcoincore.dev/$(notdir $(PWD)).gui:$(VERSION)'
	bash -c 'docker push docker.pkg.github.com/bitcoincore-dev/stats.bitcoincore.dev/$(notdir $(PWD)).gui:$(VERSION)'
#######################
package-slim: clean build-slim
	bash -c 'cat ~/GH_TOKEN.txt | docker login docker.pkg.github.com -u RandyMcMillan --password-stdin'
	bash -c 'docker tag $(PROJECT_NAME):$(HOST_USER) docker.pkg.github.com/bitcoincore-dev/stats.bitcoincore.dev/$(notdir $(PWD)).slim:$(VERSION)'
	bash -c 'docker push docker.pkg.github.com/bitcoincore-dev/stats.bitcoincore.dev/$(notdir $(PWD)).slim:$(VERSION)'
#######################
-include Makefile
