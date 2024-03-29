GIT_TAG               := $(shell git describe --exact-match --tags --abbrev=0 2> /dev/null || echo untagged)
ifeq ($(GIT_TAG),untagged)
# "SNAPSHOT" is "latest" for Java
VERSION               := 0.0.0-SNAPSHOT
else
# remove the "v" prefix, not allowed
VERSION               := $(GIT_TAG:v=)
endif

# work dir
WD                    := $(shell echo "`pwd`/client")

DOCKER = docker run --rm -v $(WD):/wd --workdir /wd
MVN = $(DOCKER) -v $(HOME)/.m2:/root/.m2 -e JAVA_SDK_MAVEN_PASSWORD=${JAVA_SDK_MAVEN_PASSWORD} maven:3-openjdk-8 mvn -s settings.xml
CHOWN = chown -R $(shell id -u):$(shell id -g)

publish: generate
	# https://help.github.com/en/packages/using-github-packages-with-your-projects-ecosystem/configuring-apache-maven-for-use-with-github-packages
	$(MVN) deploy -DskipTests -DaltDeploymentRepository=github::default::https://maven.pkg.github.com/argoproj/argo-workflows

generate:
	rm -Rf $(WD)
	mkdir -p $(WD)
	cp settings.xml $(WD)/settings.xml
	cat ../../api/openapi-spec/swagger.json | \
		sed 's/io.k8s.api.core.v1.//' | \
		sed 's/io.k8s.apimachinery.pkg.apis.meta.v1.//' \
		> $(WD)/swagger.json
	$(DOCKER) openapitools/openapi-generator-cli:v5.2.1 \
		generate \
		-i /wd/swagger.json \
		-g java \
		-o /wd \
		-p hideGenerationTimestamp=true \
		-p dateLibrary=java8 \
		--api-package io.argoproj.workflow.apis \
		--invoker-package io.argoproj.workflow \
		--model-package io.argoproj.workflow.models \
		--skip-validate-spec \
		--group-id io.argoproj.workflow \
		--artifact-id argo-client-java \
		--import-mappings Time=java.time.Instant \
		--import-mappings Affinity=io.kubernetes.client.openapi.models.V1Affinity \
		--import-mappings ConfigMapKeySelector=io.kubernetes.client.openapi.models.V1ConfigMapKeySelector \
		--import-mappings Container=io.kubernetes.client.openapi.models.V1Container \
		--import-mappings ContainerPort=io.kubernetes.client.openapi.models.V1ContainerPort \
		--import-mappings EnvFromSource=io.kubernetes.client.openapi.models.V1EnvFromSource \
		--import-mappings EnvVar=io.kubernetes.client.openapi.models.V1EnvVar \
		--import-mappings HostAlias=io.kubernetes.client.openapi.models.V1HostAlias \
		--import-mappings Lifecycle=io.kubernetes.client.openapi.models.V1Lifecycle \
		--import-mappings ListMeta=io.kubernetes.client.openapi.models.V1ListMeta \
		--import-mappings LocalObjectReference=io.kubernetes.client.openapi.models.V1LocalObjectReference \
		--import-mappings ObjectMeta=io.kubernetes.client.openapi.models.V1ObjectMeta \
		--import-mappings ObjectReference=io.kubernetes.client.openapi.models.V1ObjectReference \
		--import-mappings PersistentVolumeClaim=io.kubernetes.client.openapi.models.V1PersistentVolumeClaim \
		--import-mappings PodDisruptionBudgetSpec=io.kubernetes.client.openapi.models.V1beta1PodDisruptionBudgetSpec \
		--import-mappings PodDNSConfig=io.kubernetes.client.openapi.models.V1PodDNSConfig \
		--import-mappings PodSecurityContext=io.kubernetes.client.openapi.models.V1PodSecurityContext \
		--import-mappings Probe=io.kubernetes.client.openapi.models.V1Probe \
		--import-mappings ResourceRequirements=io.kubernetes.client.openapi.models.V1ResourceRequirements \
		--import-mappings SecretKeySelector=io.kubernetes.client.openapi.models.V1SecretKeySelector \
		--import-mappings SecurityContext=io.kubernetes.client.openapi.models.V1SecurityContext \
		--import-mappings Toleration=io.kubernetes.client.openapi.models.V1Toleration \
		--import-mappings Volume=io.kubernetes.client.openapi.models.V1Volume \
		--import-mappings VolumeDevice=io.kubernetes.client.openapi.models.V1VolumeDevice \
		--import-mappings VolumeMount=io.kubernetes.client.openapi.models.V1VolumeMount \
		--generate-alias-as-model
	# https://vsupalov.com/docker-shared-permissions/#set-the-docker-user-when-running-your-container
	$(CHOWN) $(WD) || sudo $(CHOWN) $(WD)
	# replace the generated pom.xml, because that has too many dependencies
	sed 's/0.0.0-VERSION/$(VERSION)/' pom.xml > $(WD)/pom.xml

