<h1>Stable Diffusion Web UI on EC2</h1>

A quickstart automation to deploy AUTOMATIC1111 [stable-diffusion-webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui) with AWS EC2.

The CloudFormation template deploys an EC2 instance in your AWS environment. The [user data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) will install and configure Stable Diffusion Web UI automatically at launch. That's all.

<h2>Overview</h2>

 - A GPU-based EC2 instance
   - Running on AMI: Deep Learning AMI GPU PyTorch 1.13.1 (Ubuntu 20.04) 20230530
   - Associated with static public IP address (Elastic IP)
   - Security group allows only whitelist IP range (CIDR) to access

 - Auto install and configure Stable Diffusion Web UI, and run as a systemctl service

 - Auto install some popular models, extensions and embeddings
   - Models
     - [ChilloutMix](https://civitai.com/models/6424/chilloutmix)
     - [JapaneseDollLikeness (v1.5)](https://civitai.com/models/28811)
     - [KoreanDollLikeness (v2.0)](https://civitai.com/models/26124)
     - [TaiwanDollLikeness (v2.0)](https://civitai.com/models/48363)
   - Extensions
     - [Stable-Diffusion-Webui-Civitai-Helper](https://github.com/butaixianran/Stable-Diffusion-Webui-Civitai-Helper)
     - [sd-webui-additional-networks](https://github.com/kohya-ss/sd-webui-additional-networks)
     - [sd-webui-infinite-image-browsing](https://github.com/zanllp/sd-webui-infinite-image-browsing)
     - [sd-webui-roop](https://github.com/s0md3v/sd-webui-roop)
     - [sd-webui-controlnet](https://github.com/Mikubill/sd-webui-controlnet)
       - [Model files for ControlNet 1.1](https://huggingface.co/lllyasviel/ControlNet-v1-1)
     - [openpose-editor](https://github.com/fkunn1326/openpose-editor)
   - Textual Inversion
     - [EasyNegative](https://civitai.com/models/7808)
     - [bad_prompt](https://civitai.com/models/55700/badprompt-negative-embedding)
     - [negative_hand](https://civitai.com/models/56519/negativehand-negative-embedding)
     - [bad-hands-5](https://huggingface.co/yesyeahvh/bad-hands-5)
 
<h2>Installation</h2>

<h3>Prerequisite 1 - Request AWS service quota increase for GPU instances</h3>

If you’ve never used GPU instances in AWS before, you’ll probably need to increase your quotas. AWS accounts have quotas in each region that limit how many vCPUs of a particular instance type you can run at once. The quota for GPU instances are all 0 for a new AWS account. Click this [link](https://aws.amazon.com/contact-us/ec2-request) or this [link](https://console.aws.amazon.com/servicequotas/home/services/ec2/quotas/L-DB2E81BA) to request for service quota increase. Make sure you select the correct AWS region and request for <b>G instances</b>. The limit should be at least <b>4</b> (8 or 16 is also acceptable). Note that it can take around 3-5 business days for AWS to grant the quota increase.


<h3>Create the CloudFormation stack from the AWS Management Console</h3>

1. Download template [sd-webui-ec2.yaml](sd-webui-ec2.yaml) and save it locally.

2. Log in to the AWS CloudFormation console at https://console.aws.amazon.com/cloudformation. Choose <b>Create stack</b> with new resources (standard), then choose <b>Upload a template file</b> as template source, with the file that you downloaded in the first step.

3. For stack details,
   - Stack Name: <b>sd-webui</b> <i>(or anything you like)</i>
   - Ec2KeyPair: Choose a EC2 key pair
   - Ec2InstanceType: Prefer to choose <b>g5.xlarge</b> but it is only supported in limited AWS regions. Choose <b>g4dn.xlarge</b> otherwise.
   - CidrBlock: Choose <b>your public address</b> so only you can access the Web UI and ssh. Choose <b>0.0.0.0/0</b> if you don't mind everyone can access.

4. The status of the stack will be `CREATE_COMPLETE` in 1 minute after creation, but <b>wait for 30 minutes</b> for the installation to complete. Then you can open the stable diffusion web UI using the link in output tab.

You can connect to the EC2 instance using the ssh key pair.

The installation can take 15-20 minutes. Run this command to check the installation progress in EC2 user data:
 - To view the installation log (commands run in EC2 user data):
   - `tail -f /var/log/cloud-init-output.log `

fully launched after 30 mins
 - To view log of web UI:
   - `journalctl -f -u sd-webui`

 - To start web UI:
   - `sudo systemctl start sd-webui.service`

 - To stop web UI:
   - `sudo systemctl start sd-webui.service`

 - To check service status:
   - `systemctl status sd-webui.service`

 - To disable web UI always run on boot (i.e. disable systemctl service):
   - `sudo systemctl disable sd-webui.service`

 - Then run the UI foreground:
   - ```sh
     cd stable-diffusion-webui/
     ./webui.sh --listen
     ```

Civitai Helper -> Download Model

Some popular models from Civitai:
 - [ChilloutMix](https://civitai.com/models/6424/chilloutmix)
 - [Beautiful Realistic Asians](https://civitai.com/models/25494/beautiful-realistic-asians)
 - [DreamShaper](https://civitai.com/models/4384/dreamshaper)


<h2>Troubleshooting</h2>
<h3> Error on "OutOfMemoryError: CUDA out of memory."</h3>
Reboot the EC2.

<h3>Preprocessing Images Run Error, Can't Batch Crop Images</h3>

If you see this error `stable diffusion webui pretraining cv2.error: OpenCV(4.8.0) /io/opencv/modules/dnn/src/net_impl.cpp:279: error: (-204:Requested object was not found) Layer with requested id=-1 not found in function 'getLayerData'`, trying downgrade OpenCV to 4.7.0.72 as suggested [here](https://github.com/AUTOMATIC1111/stable-diffusion-webui/discussions/11675).

```sh
cd stable-diffusion-webui/
source venv/bin/activate
pip install opencv-python==4.7.0.72
deactivate
sudo systemctl restart sd-webui
```

<h2>Credits</h2>

 - https://koding.work/use-aws-ec2-to-build-stable-diffusion-webui/

 - https://ivonblog.com/posts/stable-diffusion-webui-manuals/
