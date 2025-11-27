# etcd for CKA Practice (macOS/Linux)
ETCD_VER := v3.5.21
DIR := /tmp/etcd
URL := https://storage.googleapis.com/etcd

# Auto-detect platform
UNAME_S := $(shell uname -s)
UNAME_M := $(shell uname -m)
ifeq ($(UNAME_S),Darwin)
  PLATFORM := darwin-$(if $(filter arm64,$(UNAME_M)),arm64,amd64)
  EXT := zip
else
  PLATFORM := linux-$(if $(filter aarch64,$(UNAME_M)),arm64,amd64)
  EXT := tar.gz
endif

.PHONY: install start stop test backup restore clean

install:
	@rm -rf $(DIR) && mkdir -p $(DIR)
	@curl -sL $(URL)/$(ETCD_VER)/etcd-$(ETCD_VER)-$(PLATFORM).$(EXT) -o /tmp/etcd.$(EXT)
ifeq ($(EXT),zip)
	@unzip -q /tmp/etcd.$(EXT) -d /tmp && mv /tmp/etcd-$(ETCD_VER)-$(PLATFORM)/* $(DIR)/
else
	@tar xzf /tmp/etcd.$(EXT) -C $(DIR) --strip-components=1
endif
	@rm -f /tmp/etcd.$(EXT)
	@$(DIR)/etcd --version | head -1

start:
	@$(DIR)/etcd --data-dir=/tmp/etcd-data >/tmp/etcd.log 2>&1 & sleep 1
	@echo "etcd started (log: /tmp/etcd.log)"

stop:
	@pkill -f '$(DIR)/etcd' 2>/dev/null || true

test:
	@$(DIR)/etcdctl put foo bar && $(DIR)/etcdctl get foo

backup:
	@$(DIR)/etcdctl snapshot save /tmp/etcd-backup.db
	@$(DIR)/etcdctl snapshot status /tmp/etcd-backup.db --write-out=table

restore: stop
	@rm -rf /tmp/etcd-restore
	@$(DIR)/etcdutl snapshot restore /tmp/etcd-backup.db --data-dir=/tmp/etcd-restore
	@echo "Restored. Start with: $(DIR)/etcd --data-dir=/tmp/etcd-restore"

clean: stop
	@rm -rf $(DIR) /tmp/etcd-data /tmp/etcd-restore /tmp/etcd-backup.db /tmp/etcd.log
