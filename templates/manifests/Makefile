EXECUTABLES = check-jsonschema find jinja2 kubectl kubectx yamllint
K := $(foreach exec,$(EXECUTABLES),\
        $(if $(shell which $(exec)),some string,$(error "No $(exec) in PATH")))

CONTEXT     ?= $(shell kubectx --current)
ENVIRONMENT ?= $(shell kubectx --current)
TMPDIR      := $(shell mktemp -d)

NAMESPACES             = $(patsubst ./templates/%,./manifests/%,$(patsubst %.j2,%,$(shell find ./templates/ -name 'namespace.yaml.j2' -type f -print)))
PERSISTENTVOLUMECLAIMS = $(patsubst ./templates/%,./manifests/%,$(patsubst %.j2,%,$(shell find ./templates/ -name 'persistentvolumeclaim.yaml.j2' -type f -print)))
ONEPASSWORDITEMS       = $(patsubst ./templates/%,./manifests/%,$(patsubst %.j2,%,$(shell find ./templates/ -name 'onepassworditem.yaml.j2' -type f -print)))
CONFIGMAPS             = $(patsubst ./templates/%,./manifests/%,$(patsubst %.j2,%,$(shell find ./templates/ -name 'configmap.yaml.j2' -type f -print)))
SECRETS                = $(patsubst ./templates/%,./manifests/%,$(patsubst %.j2,%,$(shell find ./templates/ -name 'secret.yaml.j2' -type f -print)))
SERVICEACCOUNTS        = $(patsubst ./templates/%,./manifests/%,$(patsubst %.j2,%,$(shell find ./templates/ -name 'serviceaccount.yaml.j2' -type f -print)))
ROLES                  = $(patsubst ./templates/%,./manifests/%,$(patsubst %.j2,%,$(shell find ./templates/ -name 'role.yaml.j2' -type f -print)))
ROLEBINDINGS           = $(patsubst ./templates/%,./manifests/%,$(patsubst %.j2,%,$(shell find ./templates/ -name 'rolebinding.yaml.j2' -type f -print)))
DEPLOYMENTS            = $(patsubst ./templates/%,./manifests/%,$(patsubst %.j2,%,$(shell find ./templates/ -name 'deployment.yaml.j2' -type f -print)))
SERVICES               = $(patsubst ./templates/%,./manifests/%,$(patsubst %.j2,%,$(shell find ./templates/ -name 'service.yaml.j2' -type f -print)))
MANIFESTS_DIRTY        = $(NAMESPACES) $(PERSISTENTVOLUMECLAIMS) $(ONEPASSWORDITEMS)
MANIFESTS_CLEAN        = $(CONFIGMAPS) $(SECRETS) $(SERVICEACCOUNTS) $(ROLES) $(ROLEBINDINGS) $(DEPLOYMENTS) $(SERVICES)

.PHONY: all manifests install uninstall clean
.DELETE_ON_ERROR: $(MANIFESTS_DIRTY) $(MANIFESTS_CLEAN)

all: $(ENVIRONMENT).yaml install

manifests: $(MANIFESTS_DIRTY) $(MANIFESTS_CLEAN)
	@find ./ -type d -name manifests | while read i; do echo find $$i -depth -empty -delete; find $$i -depth -empty -delete; done

install: $(MANIFESTS_DIRTY) $(MANIFESTS_CLEAN)
	@for i in $^; do find -wholename ./$$i -not -empty | while read i; do echo kubectl --context $(CONTEXT) apply -f $$i; kubectl --context $(CONTEXT) apply -f $$i; done; done

uninstall: $(MANIFESTS_CLEAN)
	@for i in $^; do find -wholename ./$$i -not -empty | while read i; do echo kubectl --context $(CONTEXT) delete --ignore-not-found -f $$i; kubectl --context $(CONTEXT) delete --ignore-not-found -f $$i; done; done
	
purge: uninstall $(MANIFESTS_DIRTY)
	@for i in $^; do find -wholename ./$$i -not -empty | while read i; do echo kubectl --context $(CONTEXT) delete --ignore-not-found -f $$i; kubectl --context $(CONTEXT) delete --ignore-not-found -f $$i; done; done
	
clean:
	rm -f $(MANIFESTS_DIRTY) $(MANIFESTS_CLEAN)
	@find ./ -type d -name manifests | while read i; do echo find $$i -depth -empty -delete; find $$i -depth -empty -delete; done

manifests/%.yaml: templates/%.yaml.j2 $(ENVIRONMENT).yaml
	@mkdir -p $(dir $@)
	jinja2 --extension=jinja2_ansible_filters.AnsibleCoreFiltersExtension --extension=jinja2_getenv_extension.GetenvExtension --strict $< $(ENVIRONMENT).yaml > $@
	yamllint $@
	csplit --elide-empty-files -f $(TMPDIR)/$(subst /,-,$(basename $@))- --suffix-format="%02d.yaml" -q $@ "/---/" "{*}"
	@for i in $(TMPDIR)/$(subst /,-,$(basename $@))-*.yaml; do \
		if [ -s $$i ]; then \
			echo check-jsonschema --schemafile schemas/$(notdir $(basename $(basename $<))).json $$i; \
			check-jsonschema --schemafile schemas/$(notdir $(basename $(basename $<))).json $$i; \
		fi; \
	done
	@for i in $(TMPDIR)/$(subst /,-,$(basename $@))-*.yaml; do \
		if [ -s $$i ]; then \
			rm -f $$i; \
		fi; \
	done
