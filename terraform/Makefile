TFSUBDIRS:=	$(foreach DIR,$(shell find providers -maxdepth 1 -mindepth 1 -type d),$(subst providers/,,$(DIR)))

GO_MOD_TIDY_TARGETS:=	$(foreach DIR,$(TFSUBDIRS), $(subst $(DIR),go-mod-tidy-vendor.$(DIR),$(DIR)))
GO_BUILD_TARGETS:=	$(foreach DIR,$(TFSUBDIRS), $(subst $(DIR),go-build.$(DIR),$(DIR)))
GO_CLEAN_TARGETS:=	$(foreach DIR,$(TFSUBDIRS), $(subst $(DIR),go-clean.$(DIR),$(DIR)))
TERRAFORM_PROVIDER_TARGETS := $(foreach DIR,$(TFSUBDIRS), bin/terraform-provider-$(DIR).zip)

LDFLAGS:= "-s -w"

.PHONY: all
all: go-build

.PHONY: go-mod-tidy-vendor
go-mod-tidy-vendor: $(GO_MOD_TIDY_TARGETS)
# The -compat=1.17 is needed for the openstack provider. The provider uses golang.org/x/mod, which go 1.17 selects
# as v0.3.0 but go 1.16 selects as v0.4.2.
$(GO_MOD_TIDY_TARGETS): go-mod-tidy-vendor.%:
	cd providers/$* && go mod tidy -compat=1.17 && go mod vendor

.PHONY: go-build
go-build: $(GO_BUILD_TARGETS) go-build-terraform
$(GO_BUILD_TARGETS): go-build.%: bin/terraform-provider-%.zip

$(TERRAFORM_PROVIDER_TARGETS): bin/terraform-provider-%.zip: providers/%/go.mod
	cd providers/$*; \
	if [ -f main.go ]; then path="."; else path=./vendor/`grep _ tools.go|awk '{ print $$2 }'|sed 's|"||g'`; fi; \
	go build -ldflags $(LDFLAGS) -o ../../bin/terraform-provider-$* "$$path"; \
	zip -1j ../../bin/terraform-provider-$*.zip ../../bin/terraform-provider-$*;

.PHONY: go-build-terraform
go-build-terraform: bin/terraform

bin/terraform: terraform/go.mod
	cd terraform; \
	go build -ldflags $(LDFLAGS) -o ../bin/terraform ./vendor/github.com/hashicorp/terraform

.PHONY: go-clean
go-clean: go-clean-providers go-clean-terraform

$(GO_CLEAN_TARGETS): go-clean.%:
	rm -f bin/terraform-provider-$*
	rm -f bin/terraform-provider-$*.zip

go-clean-providers:
	rm -f bin/terraform-provider-*
	
go-clean-terraform:
	rm -f bin/terraform

.PHONY: clean
clean: go-clean
