# Negative Access Tests

Denied actions are first-class evidence of least privilege.

Planned examples:

- Developer attempts IAM administration -> DENIED.
- Auditor attempts create/update/delete -> DENIED.
- Security user attempts infrastructure administration -> DENIED.
- User without required group assignment attempts access -> DENIED.
- Offboarded/disabled identity attempts to establish new AWS access -> DENIED.

For each test capture expected result, observed error, principal/session identity and evidence path.
