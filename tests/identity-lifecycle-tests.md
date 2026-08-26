# Identity Lifecycle Tests

## Joiner

Implemented model:

1. create/enable synthetic employee identity in `madar.local`,
2. place the user in the intended corporate security group,
3. verify local domain authentication,
4. if a cloud desktop is required, assign the user to the registered WorkSpaces directory,
5. verify the WorkSpace computer object appears in the dedicated `WorkSpaces` OU,
6. verify successful WorkSpaces login with the corporate AD identity.

The project validated the cloud-authentication pattern using `sara.ibrahim`.

## Mover

Directory-side procedure:

1. change the user's corporate group membership,
2. verify old authorization is removed,
3. verify new authorization is effective,
4. re-authenticate as required to refresh group membership,
5. keep the WorkSpace computer object in the dedicated device OU.

Local positive/negative authorization behavior is already demonstrated through Sara's IT-share success and Finance-share denial.

## Leaver

Target test procedure:

1. disable the employee identity in Active Directory,
2. remove group membership according to policy,
3. stop/delete the employee WorkSpace as appropriate,
4. verify a new domain-authenticated WorkSpaces session cannot be established,
5. record revocation timing and evidence.

## Future production extension

The original IAM Identity Center Joiner/Mover/Leaver tests remain future work for an AWS account where the organizational prerequisites are intentionally enabled. They are not claimed as executed in this Free Plan lab.
