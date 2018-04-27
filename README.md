# Update AWS AutoScaleGroup

Bash script to make AWS AutoScaleGroup updates faster.

## Introduction

When changes are made to instances within an AutoScaleGroup, it's necessary to update the ASG with a new Launch config so future
instances contain these changes. In non-orchastrated environments this can be time consuming with a lot of manual steps.
What this script does is do most of the heavy lifting for you. It can do everything from creating a new AMI and launch config, through to
optionally updating your ASG with the new launch config, and even churning out your previous instances while maintaining site availability.

## Installation

Ideally this script should only be run by su users, so the recommended location is /sbin/
It needs to be run from an instance within the ASG that you've performed some updates on, and want to update the launch config from.
EG: a web node within the ASG you want to update. The script won't churn the node you're on if you choose to churn instances.
1. Copy the script to /sbin/updateasg
2. sudo chmod +x /sbin/updateasg
3. Ensure [jq](https://stedolan.github.io/jq/download/) is installed before running the script

## Usage

As explained in the Installation section, the script needs to run from a node that the target ASG launches, such as a web node.

When the script runs, it takes a snapshot of the root volume of the instance the script is running on.

It shows the snapshot progress as a percentage. Bear in mind that like the console, you may not see this number change. It might display 0%
serveral times, then suddenly display "done." This is normal. The less frequently you snapshot, the longer they take. A good trick to speed up
ASG changes is to snapshot the root volume before you make changes. The subsequent snapshot for the ASG update will go much faster.

Once the snapshot completes, a new AMI is and Launch config is created based on the current instance size, keypair etc.
This is almost instant.

You now have the option to select an existing ASG to update. Enter 0 to exit and do this process manually. The script informs you of
the AMI name you should use.

After confirming the ASG to update, The new launch config is set. You're now asked if you want to churn out the old instances.

### Instance Churning

If you ask the script to churn instances, it adds two new instances as a buffer to your ASG. It increases the max-size of the ASG
to accomdate an extra two instances. Be aware of your account EC2 limits.

Once the buffer instances have been added to the ASG, existing instances are terminated one at a time, with a cooldown period to ensure
new instances are added to replace the terminated ones.

Once the churn is completed, the ASG is restored to it's previous Desired and Max-size quantities.
