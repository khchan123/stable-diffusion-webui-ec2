# Stable Diffusion Web UI on EC2

A quickstart automation to deploy AUTOMATIC1111 [stable-diffusion-webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui) with AWS EC2.

The CloudFormation template deploys an EC2 instance in your AWS environment. The [user data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) will install and configure Stable Diffusion WebUI automatically at launch. The one-click deployment takes about 30 minutes.

## Overview

- A GPU-based EC2 instance
  
  - Running on AMI: `Deep Learning AMI GPU PyTorch 1.13.1 (Ubuntu 20.04) 20230530`
  - Associated with static public IP address (Elastic IP)
  - Security group allows only whitelist IP range (CIDR) to access

- Auto install and configure Stable Diffusion WebUI, and run as a systemctl service

- Auto install some popular models, extensions and embeddings
  
  - Installed models
    - [ChilloutMix](https://civitai.com/models/6424/chilloutmix)
    - [JapaneseDollLikeness (v1.5)](https://civitai.com/models/28811)
    - [KoreanDollLikeness (v2.0)](https://civitai.com/models/26124)
    - [TaiwanDollLikeness (v2.0)](https://civitai.com/models/48363)
  - Installed extensions
    - [Stable-Diffusion-Webui-Civitai-Helper](https://github.com/butaixianran/Stable-Diffusion-Webui-Civitai-Helper)
    - [sd-webui-additional-networks](https://github.com/kohya-ss/sd-webui-additional-networks)
    - [sd-webui-infinite-image-browsing](https://github.com/zanllp/sd-webui-infinite-image-browsing)
    - [sd-webui-roop](https://github.com/s0md3v/sd-webui-roop)
    - [sd-webui-controlnet](https://github.com/Mikubill/sd-webui-controlnet)
      - [Model files for ControlNet 1.1](https://huggingface.co/lllyasviel/ControlNet-v1-1)
    - [openpose-editor](https://github.com/fkunn1326/openpose-editor)
  - Installed textual inversion
    - [EasyNegative](https://civitai.com/models/7808)
    - [bad_prompt](https://civitai.com/models/55700/badprompt-negative-embedding)
    - [negative_hand](https://civitai.com/models/56519/negativehand-negative-embedding)
    - [bad-hands-5](https://huggingface.co/yesyeahvh/bad-hands-5)

## How to install?

### Pre-requisites

#### <i>1. Request AWS service quota increase for GPU instances</i>

AWS accounts have quotas in each region that limit how many vCPUs of a particular instance type you can run at once. The quota for GPU instances are all 0 for a new AWS account. Click this [link](https://aws.amazon.com/contact-us/ec2-request) or this [link](https://console.aws.amazon.com/servicequotas/home/services/ec2/quotas/L-DB2E81BA) to request for service quota increase. Make sure you select the correct AWS region and request for <b>G instances</b>. The limit should be at least <b>4</b> (8 or 16 is also acceptable).

> It could take 3-5 business days for AWS to grant the quota increase.

#### <i>2. Create EC2 key pair</i>

Follow this [link](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html) to create a key pair so that you can connect to the instance using SSH.

### Create CloudFormation stack

1. Download template [sd-webui-ec2.yaml](sd-webui-ec2.yaml) and save it locally.

2. Log in to the AWS CloudFormation console at https://console.aws.amazon.com/cloudformation. Choose <b>Create stack</b> with new resources (standard), then choose <b>Upload a template file</b> as template source, with the file that you downloaded in the first step.

3. For stack details,
   
   - Stack Name: <b>sd-webui</b> <i>(or anything you like)</i>
   - Ec2KeyPair: Choose a EC2 key pair
   - Ec2InstanceType: Prefer to choose <b>g5.xlarge</b> but it is only supported in limited AWS regions. Choose <b>g4dn.xlarge</b> otherwise.
   - CidrBlock: Choose <b>your public address</b> so only you can access the Web UI and ssh. Choose <b>0.0.0.0/0</b> if you don't mind everyone can access.

4. The stack takes up to 15 minutes to create, the status will become `CREATE_COMPLETE`.
   
   > Command to view installation log:  `tail -f /var/log/cloud-init-output.log`

5. The first launch of `stable-diffusion-webui` takes up to 10 minutes to install it's python dependencies. Then you can open the stable diffusion web UI using the link in output tab.
   
   > Command to view WebUI log: `journalctl -f -u sd-webui`

### Commonly used commands in EC2

You can connect to the EC2 instance using the ssh key pair.

- View installation log: `tail -f /var/log/cloud-init-output.log`

- View WebUI log: `journalctl -f -u sd-webui`

- Start WebUI: `sudo systemctl start sd-webui`

- Stop WebUI: `sudo systemctl start sd-webui`

- Check WebUI service status: `systemctl status sd-webui`

- Disable WebUI start on boot: `sudo systemctl disable sd-webui`

- Run WebUI foreground: `cd stable-diffusion-webui/; ./webui.sh --listen`

## FAQ

### Preprocessing images error when auto focal point crop enabled

If you see this error `stable diffusion webui pretraining cv2.error: OpenCV(4.8.0) /io/opencv/modules/dnn/src/net_impl.cpp:279: error: (-204:Requested object was not found) Layer with requested id=-1 not found in function 'getLayerData'`, trying downgrade OpenCV to 4.7.0.72 as suggested [here](https://github.com/AUTOMATIC1111/stable-diffusion-webui/discussions/11675).

```sh
cd stable-diffusion-webui/
source venv/bin/activate
pip install opencv-python==4.7.0.72
deactivate
sudo systemctl restart sd-webui
```

### How to change COMMANDLINE_ARGS

Modify `/home/ubuntu/stable-diffusion-webui/webui-user.sh` and restart WebUI by `sudo systemctl start sd-webui`.

### Some models that I like...

- [ChilloutMix](https://civitai.com/models/6424/chilloutmix)
- [Beautiful Realistic Asians](https://civitai.com/models/25494/beautiful-realistic-asians)
- [DreamShaper](https://civitai.com/models/4384/dreamshaper)

You can download models from Civitai easily by pasting the link in `Civitai Helper` -> `Civitai URL` in `Download Model`.

## Credits

- https://koding.work/use-aws-ec2-to-build-stable-diffusion-webui/

- https://ivonblog.com/posts/stable-diffusion-webui-manuals/
