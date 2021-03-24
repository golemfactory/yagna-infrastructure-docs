# Level 0 readme

This test runs a requestor agent using the mocked client api code generated by Swagger / openapi-generator.

## To generate the agent code:

* clone ya-client containing the swagger definitions
* install openapi-generator@5.x and add it to your path [https://openapi-generator.tech/docs/installation](https://openapi-generator.tech/docs/installation)
  * \_ I have added the alias `alias openapi-generator='java -jar ~/Downloads/openapi-generator-cli-5.0.0-20200424.033601-40.jar'`
* run `cd test/level0/mocked_client` followed by `sh combine-mocks.sh [../../../ya-client]` \( default expects ya-client to be beside yagna-integration\)
* Apply manual patches, cherry pick all required

## To run the test:

* `python setup.py develop` in the root of the `yagna-integration` repository
* run `cd test/level0/unix` followed by `sh ./start_mocked_requestor.sh`
