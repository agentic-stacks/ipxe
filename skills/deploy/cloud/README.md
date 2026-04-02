# Deploy iPXE in the Cloud

iPXE provides pre-built images for AWS EC2 and Google Cloud, enabling network boot in cloud environments. Boot scripts are passed via instance metadata.

| Platform | Image Source | Script Delivery | Console Access |
|---|---|---|---|
| AWS EC2 | Public AMIs (account `833372943033`) | User-data | `aws ec2 get-console-output` |
| Google Cloud | `ipxe-images` project | Instance metadata `ipxeboot` key | `gcloud compute instances get-serial-port-output` |

## Choose Your Platform

- **AWS EC2** — see `ec2.md`
- **Google Cloud** — see `gce.md`
