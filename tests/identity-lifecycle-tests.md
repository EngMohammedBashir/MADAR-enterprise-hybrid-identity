# Identity Lifecycle Tests

## Joiner

1. create/enable synthetic employee identity,
2. place user in intended corporate group,
3. synchronize/integrate identity,
4. verify expected AWS assignment appears,
5. verify successful SSO access.

## Mover

1. change user from one corporate group to another,
2. verify old access is removed,
3. verify new permission set becomes effective,
4. repeat positive and negative tests.

## Leaver

1. disable/remove employee identity according to the selected directory model,
2. verify the user cannot establish new AWS access,
3. record revocation timing and evidence,
4. confirm no employee-specific long-lived AWS key remains.
