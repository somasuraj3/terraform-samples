								Terraform Quickstart

Installation 

1. Download a binary of terraform from website
2. Set binary path in environment variable

____________________________________________________________________________________


Configuration : Text files that describes infrastructure and set of variables

1. .tf : Human readable, supports comments
2. .tf.json

____________________________________________________________________________________


Load Order and Semantics

1. Loads all .tf or .tf.json files withing the current directory in alphabetical order
2. Other files are ignored
3. Override files are exception
4. configuration within the loaded files are appended to each other
5. Order of variables, resources does not matter

____________________________________________________________________________________


Configuration Syntax

1. The syntax of Terraform configurations is called HashiCorp Configuration Language (HCL)

2. Example

	# An AMI
	variable "ami" {
  		description = "the AMI to use"
	}

	/* A multi
   		line comment. */
	resource "aws_instance" "web" {
  		ami               = "${var.ami}"
  		count             = 2
  		source_dest_check = false

  		connection {
   	 		user = "root"
  		}
	}

	a. Single line comment 
	
		# single line comment


	b. Multiline comment
	
		/* Multiline
			Comment */


	c. Value assignment (Values can be primitive type or list or map)
	
		key = value


	d. Strings
		
		"string" |	"${aws_instance.web.id}" | 	"${var.vpc_cidr}"


	e. Multiline strings

		<<EOF
			Multiline values 
			like shell script 
			goes here
		EOF

	f. Boolean values

		true	|	false

	g. List

		["value-1", "value-2", "value-3"]

	h. Maps

		{ "foo": "bar", "bar": "baz" }


3. .tf Vs .tf.json files

	.tf syntax :

	variable "ami" {
  		description = "the AMI to use"
   	}


   	.tf.json syntax :

   	variable = [{
  		"ami": {
    		"description": "the AMI to use",
  		}
	}]

____________________________________________________________________________________


Overrides

1. A way to create files that are loaded last and merged into your configuration, rather than appended

2. Use cases :
	
	a. Modification of terraform behavior by Machines

	b. Temporary modifications to terraform configuration

3. Valid file names for overriding terraform configuration

	override.tf  |	override.tf.json	|	temp_override.tf

4. Override files are loaded last in alphabetical order

____________________________________________________________________________________


Resource Configuration

1.	resource "TYPE" "NAME" {
		key1 = value1
		key2 = value2
		.
		.
		key3 {
			# configuration
		}
	}

2.	resource "aws_instance" "web" {
  		ami = "ami-408c7f28"
  		instance_type = "t1.micro"
	}

3. TYPE and NAME combination must be unique

4. Meta-Parameters
	
	1. count (int)

	resource "aws_instance" "web" {
  		count = 3
  		ami = "ami-408c7f28"
  		instance_type = "t1.micro"
	}

	2. depends_on (list of strings)

	resource "aws_instance" "web" {
  		depends_on = ["aws_instance.leader", "module.vpc"]
  		ami = "ami-408c7f28"
  		instance_type = "t1.micro"
	}

	3. provider (string)

		TYPE.ALIAS

		aws.west

		resource "aws_instance" "foo" {
    		provider = "aws.west"

    		# ...
		}
	
		provider "aws" {
  			alias = "west"

  			# ...
		}


	4. lifecycle (Configuration block)
		
		a. create_before_destroy (bool)
		
		b. prevent_destroy (bool)
		
		c. ignore_changes (list of strings)

		lifecycle {
    		[create_before_destroy = true|false]
    		[prevent_destroy = true|false]
    		[ignore_changes = [ATTRIBUTE NAME, ...]]
		}


	5. Timeouts

		resource "aws_db_instance" "timeout_example" {
  			allocated_storage = 10
  			engine            = "mysql"
  			engine_version    = "5.6.17"
 			instance_class    = "db.t1.micro"
  			name              = "mydb"

  			# ...

  			timeouts {
    			create = "60m"
    			delete = "2h"
    			update = "30m"
  			}
		}

	6. Connection Block

	connection {
    	user = ubuntu
    	key_path = "/tmp/privatekey.pem"
	}
	
	7. Provisioners

	provisioner NAME {
    	CONFIG ...

    	[when = "create"|"destroy"]
    	[on_failure = "continue"|"fail"]

    	[CONNECTION]
	}

	8. Full Syntax

	resource TYPE NAME {
    	CONFIG ...
    	# key = value


    	[count = COUNT]
    	[depends_on = [NAME, ...]]
    	[provider = PROVIDER]

    	[LIFECYCLE]

    	[CONNECTION]
    	[PROVISIONER ...]
	}

____________________________________________________________________________________



	Provider

	1. Every resource in Terraform is mapped to a provider based on longest-prefix matching
	   eg. The 'aws_instance' resource type would map to the aws provider (if that exists) 

	2. Provider Configuration

		provider "aws" {
  			access_key = "foo"
  			secret_key = "bar"
  			region = "us-east-1"
		}

	3. Multiple Provider Instances

		# The default provider
		provider "aws" {
  			# ...
		}

		# West coast region
		provider "aws" {
  			alias  = "west"
  			region = "us-west-2"
		}

		resource "aws_instance" "foo" {
  			provider = "aws.west"

  			# ...
		}

____________________________________________________________________________________

Variables

1. Defines the parameterization of Terraform configurations
	
   variables.tf

2. Variable Syntax

	variable NAME {
  		[type = TYPE] 				# String | List | Map
  		[default = DEFAULT]
  		[description = DESCRIPTION]
	}

3. Example
	
	a. String :

	variable "key" {
  		type = "string"
	}


	b. List :

	variable "images" {
  		type = "map"

  		default = {
    		us-east-1 = "image-1234"
    		us-west-2 = "image-4567"
  		}
	}


	c. Map :

	variable "zones" {
  		default = ["us-east-1a", "us-east-1b"]
	}

4. Booleans

	variable "active" {
  		default = false
	}

5. Environment Variables
	
	variable "somelist" {
  		type = "list"
	}

	TF_VAR_somelist='["ami-abc123", "ami-bcd234"]' terraform plan

____________________________________________________________________________________


Outputs

1. Outputs define values that will be highlighted to the user when Terraform applies

2. Can be queried easily using the output command

3. 	output "address" {
  		value = "${aws_instance.db.public_dns}"
	}

	a. value (required)
	b. description (optional)
	c. depends_on (list of strings)
	d. sensitive (optional, boolean)

____________________________________________________________________________________


Commands (CLI)

--> apply              Builds or changes infrastructure
    console            Interactive console for Terraform interpolations
--> destroy            Destroy Terraform-managed infrastructure
    env                Environment management
    fmt                Rewrites config files to canonical format
    get                Download and install modules for the configuration
    graph              Create a visual graph of Terraform resources
    import             Import existing infrastructure into Terraform
    init               Initialize a new or existing Terraform configuration
    output             Read an output from a state file
--> plan               Generate and show an execution plan
    push               Upload this Terraform module to Terraform Enterprise to run
    refresh            Update local state file against real resources
--> show               Inspect Terraform state or plan
    taint              Manually mark a resource for recreation
    untaint            Manually unmark a resource as tainted
    validate           Validates the Terraform files
--> version            Prints the Terraform version

____________________________________________________________________________________


Provisioners

1. Chef

2. Connection

3. file

4. local-exec

5. remote-exec

6. null_resource