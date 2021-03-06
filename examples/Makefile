get-python:
	kubeless function deploy get-python --trigger-http --runtime python2.7 --handler helloget.foo --from-file python/helloget.py
	echo "curl localhost:8080/api/v1/proxy/namespaces/default/services/get-python/"

get-python-verify:
	kubeless function call get-python |egrep hello.world

get-python-update:
	printf 'def foo():\n%4sreturn "hello world updated"\n' | kubeless function update get-python --from-file /dev/stdin
	echo "curl localhost:8080/api/v1/proxy/namespaces/default/services/get-python/"

get-python-update-verify:
	kubeless function call get-python |egrep hello.world.updated

get-python-34:
	kubeless function deploy get-python --trigger-http --runtime python3.4 --handler helloget.foo --from-file python/helloget.py
	echo "curl localhost:8080/api/v1/proxy/namespaces/default/services/get-python/"

get-python-34-verify:
	kubeless function call get-python |egrep hello.world

get-nodejs:
	kubeless function deploy get-nodejs --trigger-http --runtime nodejs6 --handler helloget.foo --from-file nodejs/helloget.js
	echo "curl localhost:8080/api/v1/proxy/namespaces/default/services/get-nodejs/"

get-nodejs-verify:
	kubeless function call get-nodejs |egrep hello.world

get-python-metadata:
	kubeless function deploy get-python-metadata --trigger-http --runtime python2.7 --handler helloget.foo --from-file python/helloget.py --env foo:bar,bar=foo,foo --memory 128Mi --label foo:bar,bar=foo,foobar
	echo "curl localhost:8080/api/v1/proxy/namespaces/default/services/get-python-metadata/"

get-python-metadata-verify:
	kubeless function call get-python-metadata |egrep hello.world

get-ruby:
	kubeless function deploy get-ruby --trigger-http --runtime ruby2.4 --handler helloget.foo --from-file ruby/helloget.rb
	echo "curl localhost:8080/api/v1/proxy/namespaces/default/services/get-ruby/"

get-ruby-verify:
	kubeless function call get-ruby |egrep hello.world

get-ruby-deps:
	kubeless function deploy get-ruby-deps --trigger-http --runtime ruby2.4 --handler hellowithdeps.foo --from-file ruby/hellowithdeps.rb --dependencies ruby/Gemfile
	echo "curl localhost:8080/api/v1/proxy/namespaces/default/services/get-ruby-deps/"

get-ruby-deps-verify:
	kubeless function call get-ruby-deps |egrep hello.world

get-dotnetcore:
	kubeless function deploy get-dotnetcore --trigger-http --runtime dotnetcore2.0 --handler helloget.foo --from-file dotnetcore/helloget.cs 
	echo "curl localhost:8080/api/v1/proxy/namespaces/default/services/get-dotnetcore/"

get-dotnetcore-verify:
	kubeless function call get-dotnetcore |egrep hello.world


get: get-python get-nodejs get-python-metadata get-ruby get-ruby-deps

post-python:
	kubeless function deploy post-python --trigger-http --runtime python2.7 --handler hellowithdata.handler --from-file python/hellowithdata.py
	echo "curl --data '{\"hello\":\"world\"}' localhost:8080/api/v1/proxy/namespaces/default/services/post-python/ --header \"Content-Type:application/json\""

post-python-verify:
	kubeless function call post-python --data '{"it-s": "alive"}'|egrep "it.*alive"

post-nodejs:
	kubeless function deploy post-nodejs --trigger-http --runtime nodejs6 --handler hellowithdata.handler --from-file nodejs/hellowithdata.js
	echo "curl --data '{\"hello\":\"world\"}' localhost:8080/api/v1/proxy/namespaces/default/services/post-nodejs/ --header \"Content-Type:application/json\""

post-nodejs-verify:
	kubeless function call post-nodejs --data '{"it-s": "alive"}'|egrep "it.*alive"

post-ruby:
	kubeless function deploy post-ruby --trigger-http --runtime ruby2.4 --handler hellowithdata.handler --from-file ruby/hellowithdata.rb
	echo "curl --data '{\"hello\":\"world\"}' localhost:8080/api/v1/proxy/namespaces/default/services/post-ruby/ --header \"Content-Type:application/json\""

post-ruby-verify:
	kubeless function call post-ruby --data '{"it-s": "alive"}'|egrep "it.*alive"

post-dotnetcore:
	kubeless function deploy post-dotnetcore --runtime dotnetcore2.0 --handler hellowithdata.handler --from-file dotnetcore/hellowithdata.cs --trigger-http

post-dotnetcore-verify:
	kubeless function call post-dotnetcore --data '{"it-s": "alive"}'|egrep "it.*alive"

post: post-python post-nodejs post-ruby

pubsub-python:
	kubeless topic create s3-python
	kubeless function deploy pubsub-python --trigger-topic s3-python --runtime python2.7 --handler pubsub.handler --from-file python/pubsub.py

# Generate a random string to inject into s3 topic,
# then "tail -f" until it shows (with timeout)
pubsub-python-verify:
	$(eval DATA := $(shell mktemp -u -p entry -t XXXXXXXX))
	kubeless topic publish --topic s3-python --data "$(DATA)"
	bash -c 'grep -q "$(DATA)" <(timeout 60 kubectl logs -f $$(kubectl get po -oname|grep pubsub-python))'

pubsub-python-update:
	kubeless topic create s3-python-2
	kubeless function update pubsub-python --trigger-topic s3-python-2

pubsub-python-update-verify:
	kubectl describe $$(kubectl get po -oname|grep pubsub-python) | grep -e "TOPIC_NAME:\s*s3-python-2"

pubsub-python34:
	kubeless topic create s3-python34
	kubeless function deploy pubsub-python34 --trigger-topic s3-python34 --runtime python3.4 --handler pubsub.handler --from-file python/pubsub.py

pubsub-python34-verify:
	$(eval DATA := $(shell mktemp -u -p entry -t XXXXXXXX))
	kubeless topic publish --topic s3-python34 --data "$(DATA)"
	bash -c 'grep -q "$(DATA)" <(timeout 60 kubectl logs -f $$(kubectl get po -oname|grep pubsub-python34))'

pubsub-nodejs:
	kubeless topic create s3-nodejs
	kubeless function deploy pubsub-nodejs --trigger-topic s3-nodejs --runtime nodejs6 --handler pubsub.handler --from-file nodejs/helloevent.js

pubsub-nodejs-verify:
	$(eval DATA := $(shell mktemp -u -p entry -t XXXXXXXX))
	kubeless topic publish --topic s3-nodejs --data '{"test": "$(DATA)"}' 
	bash -c 'grep -q "{\"test\": \"$(DATA)\"}" <(timeout 60 kubectl logs -f $$(kubectl get po -oname|grep pubsub-nodejs))'

pubsub-ruby:
	kubeless topic create s3-ruby
	kubeless function deploy pubsub-ruby --trigger-topic s3-ruby --runtime ruby2.4 --handler pubsub.handler --from-file ruby/helloevent.rb

pubsub-ruby-verify:
	$(eval DATA := $(shell mktemp -u -p entry -t XXXXXXXX))
	kubeless topic publish --topic s3-ruby --data "$(DATA)"
	bash -c 'grep -q "$(DATA)" <(timeout 60 kubectl logs -f $$(kubectl get po -oname|grep pubsub-ruby))'

post: pubsub-python pubsub-nodejs pubsub-ruby

webserver:
	kubeless function deploy --env APACHE_HTTP_PORT_NUMBER=8080 --runtime-image bitnami/apache:2.4.27-r0 webserver

webserver-verify:
	kubeless function call webserver |egrep "It works!"

webserver-update:
	kubeless function update --env NGINX_HTTP_PORT_NUMBER=8080 --runtime-image bitnami/nginx:1.12.1-r2 webserver

webserver-update-verify:
	kubeless function call webserver |egrep "Welcome to nginx!"

