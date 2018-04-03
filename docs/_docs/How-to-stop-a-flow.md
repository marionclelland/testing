## When
You need to stop an integration on a users behalf.

## Do

1. SSH to `fireflybuild.hursley.ibm.com`
2. Login to Bluemix by following the steps in /docs/Restart-apps-in-production
3. Set the following env vars on firefly-ops-commands:
```
cf set-env firefly-ops-commands COMMAND_TO_RUN flow-stop
cf set-env firefly-ops-commands FLOW_ID <flow_service_id_of_flow>
cf set-env firefly-ops-commands INSTANCE_ID <instance_id_of_flow_owner>
```
4. Start firefly-ops-commands using `cf start firefly-ops-commands`
5. Once `Waiting for stop` messages appear, stop the app with `cf stop firefly-ops-commands`
6. Inspect the command output to ensure successful completion.
