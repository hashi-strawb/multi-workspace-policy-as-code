# Example CIDR Range Policy

This is an example policy which checks if a specified CIDR range overlaps with any
deployed AWS VPCs.

As an example of this running through the CLI:

```
$ sentinel test -verbose
Installing test modules for test/example/fail.hcl
Installing test modules for test/example/pass.hcl

PASS - example.sentinel
  PASS - test/example/fail.hcl


    logs:

      ws-Y6VUTq1nCHtQydeX webserver-aws-prod
        module.webserver.module.vpc aws_vpc.this
                This VPC overlaps with our CIDR Range
                cidr_block: 10.0.0.0/16
                our cidr: 10.0.0.0/24
    trace:
      example.sentinel:86:1 - Rule "main"
        Description:
          Does our CIDR range overlap with any existing VPCs?

        Value:
          false
  PASS - test/example/pass.hcl

    trace:
      example.sentinel:86:1 - Rule "main"
        Description:
          Does our CIDR range overlap with any existing VPCs?

        Value:
          true
1 tests completed in 7.267478363s
```

In this example, the failed test includes the CIDR range `10.0.0.0/24`. The policy found a VPC in the
`webserver-aws-prod` workspace with CIDR range `10.0.0.0/16`. This overlaps with our specified CIDR, causing the policy
to fail.

## Disclaimers

This is meant as a proof-of-concept of the idea, and not a fully functional policy.

Notabaly, the policy DOES NOT check any resources in your Terraform Plan. It is only testable with params.

Getting CIDR ranges for planned VPCs is considered trivial and left as an exercise for the reader.

A complete and useful version of this policy would also want to compare CIDR ranges within a single workspace.
It should probably do that part first, and fail fast, before ever making any API requests.



Also, pay close attention to how long these policy tests took to run: 7.2 seconds (or 3.6 per policy check).
While testing, I had a TFC Org with 18 workspaces, of which 4 are currently managing resources, of which 1 has a VPC.
So you can imagine that this policy would probably take an unreasonable amount of time to run.

The reason for this is that we are making a large number of API requests to TFC in sequence:
* List all workspaces
* For some of these workspaces, give me the latest state version
* For some of these states, give me the full state file

Realistically, a better solution would be to integrate this with an IPAM solution.

If you do not have an IPAM solution... you should probably invest in an IPAM solution.

For example, there is a community provider for phpIPAM: https://registry.terraform.io/providers/Ouest-France/phpipam/latest/docs
