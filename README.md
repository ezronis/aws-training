# Architechting on AWS

## Getting Started
Follow each of these labs to walk through services available on AWS.

### "On the test" notes
- EC2 metadata: from the EC2 instance, you can query the instance metadata from the hypervisor is in by curling the following:
```
curl 169.254.169.254/latest/meta-data/
curl 169.254.169.254/latest/meta-data/ami-id
curl 169.254.169.254/latest/meta-data/hostname
```
