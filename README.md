🔥 Supports [Release 1.5.1 · AUTOMATIC1111/stable-diffusion-webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui/releases/tag/v1.5.1) - released at July 27, 2023

# Stable Diffusion Web UI on EC2

A quickstart automation to deploy AUTOMATIC1111 [stable-diffusion-webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui) with AWS EC2.

The CloudFormation template deploys an EC2 instance in your AWS environment. The [user data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) will install and configure Stable Diffusion WebUI automatically at launch. The one-click deployment takes around 20 minutes to complete.

## Overview

- A GPU-based EC2 instance
  
  - AMI: `Deep Learning AMI GPU PyTorch 1.13.1 (Ubuntu 20.04) 20230530`
  - static public IP address (Elastic IP)
  - swap volumes from instance store
  - security group allows only whitelist IP CIDR to access

- Auto install stable-diffusion-webui and run as a systemctl service

- Auto install some popular models, extensions and embeddings

## How to install?

### Step 1: Request AWS service quota increase

Open this [link](https://console.aws.amazon.com/servicequotas/home/services/ec2/quotas/L-DB2E81BA) to request for service quota increase for G instances. Make sure you are requesting the correct AWS region! The limit has to be at least 4 for running one xlarge instance. You may request for a higher quota such as **8** or **16**.

> AWS accounts have quotas in each region that limit how many vCPUs of a particular instance type you can run at once. The quota for GPU instances are all 0 for a new AWS account.
> It may take a couple days for AWS to approve the quota increase.

### Step 2: Create EC2 key pair

Follow this [link](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html) to create a key pair so that you can connect to the instance using SSH.

### Step 3: Create CloudFormation stack

1. Download template [sd-webui-ec2.yaml](sd-webui-ec2.yaml) and save it locally.

2. Log in to the AWS CloudFormation console at https://console.aws.amazon.com/cloudformation. Choose <b>Create stack</b> with new resources (standard), then choose <b>Upload a template file</b> as template source, with the file that you downloaded in the first step.

3. For stack details,
   
   - Stack Name: `sd-webui` <i>(any name you like)</i>
   - Ec2KeyPair: Choose a EC2 key pair
   - Ec2InstanceType: `g5.xlarge` or `g4dn.xlarge` should be good.

     > - `g5.xlarge` - $0.526 per hour in us-east-1
     >  - 4 vCPU + 16 GB RAM + NVIDIA A10G GPU / 16 GB VRAM
     >  - available in limited regions, such as us-east-1	(N. Virginia), us-east-2 (Ohio), us-west-2 (Oregon), eu-west-1 (Ireland), eu-west-2 (London) and ap-southeast-1 (Singapore)
     >  - it takes around 0.5 minutes to generate an image of 512x768px with 28 steps + highres fix 2x upscaling with 20 steps
     > - `g4dn.xlarge` - $1.006 per hour in us-east-1
     >   - 4 vCPU + 16 GB RAM + NVIDIA T4 GPU / 16 GB VRAM
     >  - available in many AWS regions
     >  - it takes around 1.5 minutes to generate an image of 512x768px with 28 steps + highres fix 2x upscaling with 20 steps

   - CidrBlock: Input <b>your public address</b> so only you can access the Web UI and ssh. Choose `0.0.0.0/0` if you don't mind everyone can access.

4. The stack status will become `CREATE_COMPLETE` around 10 minutes.
   
   > Command to view installation log:  `tail -f /var/log/cloud-init-output.log`

5. The first launch of `stable-diffusion-webui` takes up to 10 minutes to install its python dependencies. Then you can open the stable diffusion web UI using the link in output tab.
   
   > Command to view WebUI log: `journalctl -f -u sd-webui`

### Common commands in EC2

> You can connect to the EC2 instance using one of the following methods.
> 
> - ssh key pair <pre>ssh -i *<key-pair.pem>* ubuntu@*\<ip-address>*</pre>
>
> - SSM Session Manager (from the `Connect` button in AWS console), then switch to `ubuntu` user
>   
>   ```bash
>   sudo su ubuntu
>   ```

Common commands:

- View installation log: `tail -f /var/log/cloud-init-output.log`

- View WebUI log: `journalctl -afu sd-webui`

- Start WebUI: `sudo systemctl start sd-webui`

- Stop WebUI: `sudo systemctl start sd-webui`

- Check WebUI service status: `systemctl status sd-webui`

- Disable WebUI start on boot: `sudo systemctl disable sd-webui`

- Run WebUI foreground: `cd stable-diffusion-webui/; ./webui.sh --listen`

## Documentation

https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki

## Installed models, extensions and embeddings

Installed Models

- [ChilloutMix](https://civitai.com/models/6424/chilloutmix)

Installed Loras

- [JapaneseDollLikeness (v1.5)](https://civitai.com/models/28811)

- [KoreanDollLikeness (v2.0)](https://civitai.com/models/26124)

- [TaiwanDollLikeness (v2.0)](https://civitai.com/models/48363)

Installed Extensions

- [Stable-Diffusion-Webui-Civitai-Helper](https://github.com/butaixianran/Stable-Diffusion-Webui-Civitai-Helper)
- [sd-webui-additional-networks](https://github.com/kohya-ss/sd-webui-additional-networks)
- [sd-webui-infinite-image-browsing](https://github.com/zanllp/sd-webui-infinite-image-browsing)
- [sd-webui-roop](https://github.com/s0md3v/sd-webui-roop)
- [sd-webui-controlnet](https://github.com/Mikubill/sd-webui-controlnet)
  - [Model files for ControlNet 1.1](https://huggingface.co/lllyasviel/ControlNet-v1-1) - canny and openpose only
- [openpose-editor](https://github.com/fkunn1326/openpose-editor)

Installed Textual Inversion (Embeds)

- [EasyNegative](https://civitai.com/models/7808)
- [bad_prompt](https://civitai.com/models/55700/badprompt-negative-embedding)
- [negative_hand](https://civitai.com/models/56519/negativehand-negative-embedding)
- [bad-hands-5](https://huggingface.co/yesyeahvh/bad-hands-5)

Some models that I like

> You can download models easily by pasting the Civitai link in `Civitai Helper` -> `Civitai URL` in `Download Model`.

- [SDXL 1.0](https://civitai.com/models/101055)
- [Beautiful Realistic Asians](https://civitai.com/models/25494/beautiful-realistic-asians)
- [DreamShaper](https://civitai.com/models/4384/dreamshaper)

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

Modify `/home/ubuntu/stable-diffusion-webui/webui-user.sh` and restart WebUI by `sudo systemctl start sd-webui`. More details of command line arguments can be found at  https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Command-Line-Arguments-and-Settings.

## Credits

- https://koding.work/use-aws-ec2-to-build-stable-diffusion-webui/

- https://ivonblog.com/posts/stable-diffusion-webui-manuals/

- https://towardsdatascience.com/create-your-own-stable-diffusion-ui-on-aws-in-minutes-35480dfcde6a
