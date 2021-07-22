---
title: "Why You Should Be Using Terraform Remote State"
date: 2017-11-28T23:10:18+01:00
draft: false
thumbnail: images/terraform.png
---

[Terraform](https://terraform.io) is an amazing tool which has transformed the way we manage infrastructure. In this opinionated article, i’m going to explain why I personally believe *everyone* should be using remote state in Terraform.

Chances are, that, if you are using Terraform within your team you’re probably already using remote state. If you’re not, then pause everything and read on! It’s also likely that if you are just starting out with Terraform or you work on your own, you’re using the out of the box behaviour by storing ‘state’ on your local filesystem in a file called terraform.tfstate

So, maybe you’re wondering what ‘state’ is. In Terraform terms, state is a snapshot of your infrastructure from when you last ran the terraform apply command i.e. your s3 buckets, VPCs, DNS records and maybe even your Github repositories. All it really is, is a JSON file which contains details about every configuration attribute of each managed resource since you last applied your terraform scripts. It’s vitally important since it’s how Terraform understands what’s changed, what is new and what is no longer needed.

![Terraform State](/images/terraform-state.png)

## So, we all understand the importance of state. Why would I want ‘remote state?’

Remote state comes into play through the use of ‘backends’. A backend tells Terraform that the snapshot of your infrastructure should no longer live in the `terraform.tfstate` file on your local filesystem and that it should be stored in a remote location, such as Amazon S3. I personally use the S3 backend for all my projects so I’ll concentrate on that backend provider here, but the same principles apply to all the supported backends.

The main benefit you achieve with remote state is having a centralised source of truth. If you work in a team, or use more than one machine this is so important. When running the `terraform apply` command, the state of your infrastructure after any changes is uploaded to your backend, meaning that any person or machine which wants make further changes will know what changes have already been applied and will not inadvertently remove or undo them.

# Here’s a practical example of what problem remote state solves

Let’s say you have a website hosted in an S3 bucket and you use a MacBook and an iMac. You have your Terraform repository on both machines running local state with a single S3 bucket which both machines know about (from their own state files). In the morning you decide, whilst using your MacBook, that you want to enable logging on one of your buckets. You add the attributes to your resource and apply your changes. You return later that day, to add a new S3 bucket. This time, you decide that you want to use your iMac — so you create the new resource and run `terraform plan`. What you’ll see this time is that Terraform is going to create your new bucket as expected, but you’re also going to remove the logging from your existing bucket. This is because your state file on your iMac isn’t aware that the bucket should have logging enabled so it’ll remove it to ensure the resource is built to the specification which it has been told about. In contrast, when you have a backend configured the first thing Terraform does is read the state from the remote server, so it’ll always be working with the most up to date information.

Now, you may be thinking that surely the easiest solution to this is to commit your `terraform.tfstate` file to your git repository. You’re right that will solve the problem, but this method has some serious drawbacks. The first and most important one is that your state file can sometimes include secret details, such as database passwords or API keys. You definitely do not want this kind of information in version control. Secondly, if you, or a teammate of yours forgets to commit their `terraform.tfstate` file you’ll end up in exactly the same situation as before, potentially making unwanted changes.

## There’s more advantages to remote state

These advantages are mainly a plus for teams using Terraform, but still apply to lone wolves going it alone.

A backend such as S3 allows you to encrypt your files in transit and at rest adding confidence that any secrets stored in your state are less susceptible to falling into the wrong hands. Object versioning in S3 also provides a valuable resource for understanding and debugging what was changed between each terraform apply if a change negatively impacts your configuration.

Storing your state remotely also adds an increased layer of durability, it’s a lot harder to accidentally delete your tfstate file when it is stored remotely.

Locking support (for most backend providers) is essential when working in teams. It prevents anybody from writing to the remote state file whilst someone else is writing to it. This is to prevent data corruption that could occur if two people were applying their changes at the same time. With locking enabled, Terraform will make everyone else wait until your changes have been applied and written to state so that everyone’s plan of changes is always accurate. It’s the technological equivalent of the [token system used on the railways](https://en.wikipedia.org/wiki/Token_(railway_signalling)).

## So, how do I setup remote state?

I’m sure by now you agree that there is no reason not to configure remote state and are wondering how we configure it. Thankfully Terraform, like everything else it does, makes this painless.

Firstly, we need to tell Terraform which backend to use. We do this by adding the below snippet to our main terraform file. If you haven’t already, you should checkout the [Terraform backend documentation](https://www.terraform.io/docs/backends/types/index.html) to learn about how you can configure each provider.

```
terraform {
    backend "s3" {
        bucket = "my-bucket-name"
        key = "filename.tfstate"
        region = "eu-west-2"
        encrypt = true
    }
}
```

The code example above, assumes that you have created an S3 bucket named ‘my-bucket-name’ and that you want to store your state in an encrypted file called ‘filename.tfstate’.

Thats it! You’re now using S3 as a backend to store your remote state. If you already have a local `terraform.tfstate` file which you want to use in your remote state for the first time you can use the push command to upload that state to the remote server.

```
terraform init
terraform state push
```