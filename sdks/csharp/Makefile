GIT_TAG               := $(shell git describe --exact-match --tags --abbrev=0 2> /dev/null || echo untagged)
ifeq ($(GIT_TAG),untagged)
# "SNAPSHOT" is "latest" for Java
VERSION               := 0.0.0-SNAPSHOT
else
# remove the "v" prefix, not allowed
VERSION               := $(GIT_TAG:v=)
endif

# work dir
WD := $(shell echo "`pwd`/client")

DOCKER = docker run --rm -v $(WD):/wd --workdir /wd
CHOWN = chown -R $(shell id -u):$(shell id -g)

publish: generate
	dotnet build client

generate:
	rm -Rf $(WD)
	mkdir -p $(WD)
	cat ../../api/openapi-spec/swagger.json | \
		sed 's/io.k8s.api.core.v1.//' | \
		sed 's/io.k8s.apimachinery.pkg.apis.meta.v1.//' \
		> $(WD)/swagger.json
	cp ../../LICENSE $(WD)/LICENSE
	$(DOCKER) openapitools/openapi-generator-cli:v6.2.1 \
		generate \
		--input-spec /wd/swagger.json \
		--generator-name csharp-netcore \
		--output /wd \
		--additional-properties targetFramework=net6.0 \
		--additional-properties packageVersion=${VERSION} \
		--additional-properties packageName="Argo.Workflows" \
		--additional-properties projectName="argo-workflows" \
		--additional-properties hideGenerationTimestamp=true \
		--additional-properties nullableReferenceTypes=true \
		--additional-properties netCoreProjectFile=true \
		--additional-properties returnICollection=true \
		--additional-properties useDateTimeOffset=true \
		--additional-properties disallowAdditionalPropertiesIfNotPresent=false \
		--remove-operation-id-prefix \
		--library httpclient \
		--model-name-prefix '' \
		--model-name-suffix '' \
		--artifact-id argo-csharp-client \
		--artifact-version ${VERSION} \
 		--global-property packageName=Argo.Workflows \
		--generate-alias-as-model
	# https://vsupalov.com/docker-shared-permissions/#set-the-docker-user-when-running-your-container
	$(CHOWN) $(WD) || sudo $(CHOWN) $(WD)
