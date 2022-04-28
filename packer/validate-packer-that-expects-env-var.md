# Locally validate packer for EC2 images that expects an environment variable

I was working on creating EC2 images using Packer.
For that, there's the `amazon-ebs` [Packer Builder](https://www.packer.io/plugins/builders/amazon/ebs) to use.

After checking that there's nothing wrong with the syntax,
thanks to `packer validate --syntax-only /path/to/packter-template.pkr.hcr`,
I ran `packer validate /path/to/packter-template.pkr.hcr` and got the error below:

```text
packer validate --syntax-only /path/to/packter-template.pkr.hcr
Syntax-only check passed. Everything looks okay.
packer validate /path/to/packter-template.pkr.hcr
Error: 2 error(s) occurred:

* A source_ami or source_ami_filter must be specified
* For security reasons, your source AMI filter must declare an owner.

  on/path/to/packter-template.pkr.hcr line 42:
  (source code not available)
```

The question became
> is `source_ami` set ? If so, why isn't packer finding it?

The `source_ami` is set but expects a value from an environment variable.

```hcl
. . .
variable "source_ami" {
  type    = string
  default = env("SOURCE_AMI")
}
. . .
source "amazon-ebs" "ami_name" {
. . .
  source_ami   = var.source_ami
}
```

Because packer uses the [`env` function](https://www.packer.io/docs/templates/hcl_templates/functions/contextual/env),
the next step is to set `SOURCE_AMI` as an environment variable.

In the CI part with AWS CodeBuild, `SOURCE_AMI` is set by Cloudformation.
To validate packer locally means that I need to set `SOURCE_AMI` to some value.

Setting `SOURCE_AMI` as a ["normal" variable](https://www.packer.io/docs/templates/hcl_templates/variables#a-variable-value-must-be-known),
even after taking into account the [precedence of how packer evaluates variables](https://www.packer.io/docs/templates/hcl_templates/variables#variable-definition-precedence), doesn't work.
E.g., using the `-var` option when ivoking packer:  `packer validate -var SOURCE_AMI=1234 Packer/template.pkr.hcl`

In my Powershell console, I set the environment variable with
`$env:SOURCE_AMI = '1234'`
and
invoke `packer validate --syntax-only /path/to/packter-template.pkr.hcr`
