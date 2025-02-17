# Copyright (c) Aptos
# SPDX-License-Identifier: Apache-2.0

test: clean lint test-code-gen test-api-spec clean

lint:
	npx @redocly/openapi-cli lint doc/openapi.yaml --skip-rule no-empty-servers

test-code-gen:
	echo '{"generator-cli": {"version": "5.2.1"}}' > openapitools.json # v5.3 has bug, ping the version to 5.2.1
	npx @openapitools/openapi-generator-cli generate -g rust -i doc/openapi.yaml -o /tmp/aptos_api_client --package-name aptos_api_client
	cd /tmp/aptos_api_client && cargo build

clean:
	- pkill aptos-node
	- rm -rf /tmp/aptos_api_client
	- rm -f openapitools.json
	- rm -rf .hypothesis

test-api-spec:
	- pkill aptos-node
	cargo build -p aptos-node
	./../target/debug/aptos-node --test --lazy &

	curl https://raw.githubusercontent.com/vishnubob/wait-for-it/master/wait-for-it.sh > /tmp/wait-for-it.sh
	chmod +x /tmp/wait-for-it.sh
	/tmp/wait-for-it.sh -t 300 localhost:8080

	schemathesis run --method GET \
		--show-errors-tracebacks \
		--code-sample-style=curl \
		--store-network-log=./../target/schemathesis-network-log.yaml \
		--checks all \
		--base-url http://localhost:8080 \
		http://localhost:8080/openapi.yaml


serve:
	cd doc && python -m http.server 8888

.PHONY: test lint test-code-gen test-api-spec clean serve
