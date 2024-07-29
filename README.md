
# Description
IAMCheckerAWS is an IAM policy statement parser and query tool that aims to simplify the process of collecting and understanding permission policy statements for users and roles in AWS Identity and Access Management (IAM). Although its functionality is straightforward, this tool was developed in response to the need for an efficient solution during day-to-day AWS penetration testing.

### Blog Post
https://elvescore.io/aws/iamcheckeraws-aws-iam-permissions/

## Offensive Use
The tool can be utilized to search for potential privilege escalation opportunities in AWS accounts by querying various AWS IAM actions that might be exploited. While other tools perform scans to identify privilege escalation risks, this tool enables a more manual approach, allowing users to investigate permissions and quickly review the roles, users, and resources they apply to for targeted analysis.

## Blue Team Use
This tool also offers the ability to output and save query results in a CSV format, which is beneficial for security teams seeking a high-level overview of principal permissions and resources within an AWS account. For instance, you may want to identify users and roles with `iam:put*` permissions in an account. By executing a query and generating a CSV, you can easily review all users and roles with these permissions, along with the resources they have access to.

# Installation

Suggested:
```
pip3 install iamcheckeraws
```

Much of this functionality has also been implemented into https://github.com/Elves-Core/pacu as a module, `iam__enum_action_query` if you prefer that.

Clone and use Poetry:
```
git clone https://github.com/ElvesCore/IAMCheckerAWS.git
cd IAMCheckerAWS
# (potentially necessary: poetry env use python3.9)
poetry install
iamcheckeraws --help
iamcheckeraws --collect --profile <some-aws-profile>
```

Clone and use Pip:
```
git clone https://github.com/ElvesCore/IAMCheckerAWS.git
cd IAMCheckerAWS
pip install .
iamcheckeraws --help
iamcheckeraws --collect --profile <some-aws-profile>
```

# Usage
Help:
```
usage: iamcheckeraws [-h] [--profile PROFILE] [--account ACCOUNT] [--query QUERY] [--role ROLE] [--user USER]
                          [--all-or-none] [--collect] [--list] [--csv CSV] [--config CONFIG]

Collect all policies for all users/roles in an AWS account and then query the policies for permissions.

optional arguments:
  -h, --help         show this help message and exit
  --profile PROFILE  The name of the AWS profile to use for authentication for user/role collection.
  --account ACCOUNT  Account number to query.
  --query QUERY      Permissions to query. A string like: s3:GetObject or s3:* or s3:GetObject,s3:PutObject
  --role ROLE        Filter role to query.
  --user USER        Filter user to query.
  --all-or-none      Check if all queried actions are allowed, not just some.
  --collect          Collect user and role policies for the account.
  --list             List accounts available to query.
  --csv CSV          File name for CSV report output.
  --config CONFIG    JSON config file for preset queries.
```

## Examples
First download all IAM info for users and roles:  
`iamcheckeraws --collect --profile my-aws-profile`  

List any account data has been collected for:  
`iamcheckeraws --list`  

Then query something:  
`iamcheckeraws --account <account_number_of_profile_above> --query iam:create*`  

Then query more:  
`iamcheckeraws --account <account_number_of_profile_above> --query iam:create*,iam:put*`  

Query a particular role:  
`iamcheckeraws --account <account_number_of_profile_above> --role some_role --query iam:*`  

Query a particular user:  
`iamcheckeraws --account <account_number_of_profile_above> --user some_user --query iam:*`  

Output to a CSV:  
`iamcheckeraws --account <account_number_of_profile_above> --query iam:* --csv report.csv`  

Run a preset config:  
`iamcheckeraws --account <account_number_of_profile_above> --config dangerous_iam`

Run a query which only shows the results if a user or role has all queried permissions:  
`iamcheckeraws --account <account_number_of_profile_above> --query s3:getobject,s3:listbucket --all-or-none`

## Configs
Builtin config options are:
```
dangerous_iam
write_actions
privescs
```
`iamcheckeraws --account <account_number_of_profile> --config dangerous_iam`

## Creating Custom Configs
A config file is a JSON file that specifies some specific permissions you want to look for. Here is an example file which searches for write actions in CodeBuild.
```
[
	{
		"Description": "These are all actions which may allow some kind of write privilege in CodeBuild.",
		"Name": "CodeBuildWriteActions",
		"ActionsNeeded": [
			"codebuild:Put*",
			"codebuild:Create*",
			"codebuild:Delete*",
			"codebuild:Modify*",
			"codebuild:Update*",
			"codebuild:Attach*",
			"codebuild:Detach*",
			"codebuild:Associate*",
			"codebuild:Disassociate*",
			"codebuild:Add*",
			"codebuild:Remove*",
			"codebuild:Set*",
			"codebuild:Enable*",
			"codebuild:Disable*",
			"codebuild:Reset*",
			"codebuild:Stop*",
			"codebuild:Terminate*",
			"codebuild:Reboot*",
			"codebuild:Start*"
		],
		"AllOrNone": false
	}
]
```
You could then run the config by simply specifying the file name in the `--config` argument.  
`iamcheckeraws --account <account_number_of_profile> --config write_code_build.json`
