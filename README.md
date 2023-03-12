# EC2 instance child resource tagger

## Copy specific tags from EC2 instances to their child resources

This simple Ansible project updates the tags on AWS resources that are attached to an EC2 instance. For example, when adding a cost allocation tag to an instance, it's nice to have it propagated to the attached network interface and ebs volume.

## Features

Updates the tags to match those of the parent EC2 instance on:

* Elastic network interfaces
* EBS volumes

## Requirements

* ansible
* boto3

## Usage

Start by specifying the tags you wish to have copied from the parent instances to the child resources by updating the 'tag_resources.yaml' file:

```yaml:
- name: Propagate specific instance tags to child resources
  amazon.aws.ec2_tag:
    region: "{{ aws_region }}"
    aws_profile: "{{ aws_profile }}"
    resource: "{{ item.resource_id }}"
    purge_tags: false
    state: present
    tags:
      project: "{{ item.tags | json_query('project') }}"
      environ: "{{ item.tags | json_query('environ') }}"
  loop: "{{ all_resources }}"
  when:
    - item.tags | json_query('project')
    - item.tags | json_query('environ')
```

You can then run ansible-playbook:

```shell:
ansible-playbook -i inventories/aws/ playbooks/main.yaml -e aws_region=eu-central-1
```

### Note

* Only the instances with the tag specified in the 'when' condition will be affected
* Non matching tags will *not* be removed
* If the tag already exists it will have it's value updated
* If the instance is created by an autoscaling group, updated tags will be removed after a scaling action

## TODO

* Tag snapshots
