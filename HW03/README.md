# Jetson webcamera to AWS S3


# Jetson Side Configuration and webcamera
## 1.Create docker images for MQTT

```
docker build --network=host -t mosquitto_jnx2 -f Dockerfile.mosquitto_jnx2 .
docker build --network=host -t ubuntu_jnx2 -f Dockerfile.ubuntu_jnx2 .
```

## 2.Run the Mosquitto broker and forwarder
```
docker network create --driver bridge faces
docker run --name mosquitto --network faces -p :1883 -v "$PWD":/HW03 -d mosquitto_jnx2 sh -c "mosquitto -c /HW03/mqtt_broker_jnx2.conf"
docker run --name forwarder --network faces -v "$PWD":/HW03 -d mosquitto_jnx2 sh -c "mosquitto -c /HW03/mqtt_forwarder_jnx2.conf"
```
These commands create the necessary containers that connect to the client as a broker and then forwarded towards the subscriber in the cloud.

## 3.Run the image application

```
python3 app_faces_video.py test.mosquitto.org
``` 
___

# Set up VM on AWS

## 1.Create a new virtual server in AWS
```
aws ec2 create-security-group --group-name PublicSG --description "Bastion Host Security group" --vpc-id vpc-0244ddade700ae16c

aws ec2 authorize-security-group-ingress --group-id sg-0723e95365a9db1f --protocol tcp --port 22 --cidr 0.0.0.0/0

ssh -A ubuntu@ec2-34-239-169-230.compute-1.amazonaws.com

```

## 2.Create a AWS S3 storage object 

* Bucket name: _face-app-cat_. This bucket is set to public.

## 3.Install Docker 

## 4.Install the S3 on VI

```
sudo apt-get update
sudo apt-get install automake autotools-dev g++ git libcurl4-openssl-dev libfuse-dev libssl-dev libxml2-dev make pkg-config
git clone https://github.com/s3fs-fuse/s3fs-fuse.git
cd s3fs-fuse
./autogen.sh
./configure
make
sudo make install
```

## 5.Add the Cloud Storage to VI

```
echo "<AccessKeyID>:<SecretAccessKey>" > $HOME/.passwd-s3fs
chmod 600 $HOME/.passwd-s3fs
```

```
mkdir /mnt/face-app-cat

sudo s3fs face-app-cat /mnt/face-app-cat -o passwd_file=$HOME/.passwd-s3fs -o endpoint=us-east-1 -ouid=1000,gid=1000,allow_other,mp_umask=002

```

## 6.- Create the docker image
 
```
sudo docker build -t cloud_ivs -f Dockerfile.cloud_IVS .


sudo docker run --name mosquitto -p 1883:1883 -v "/root/HW03":/HW03 -d cloud_ivs mosquitto

sudo docker run --name subscriber7 -v "/root/HW03":/HW03 -v "/mnt/face-app-cat":/HW03-faces -ti cloud_ivs bash
```
Inside _subscriber_ container bash we run

```
python3 app_faces_subscribe.py test.mosquitto.org /HW03-faces/

```

please refer to _face0-8.png_ and _my_s3_object_face-app-cat.png_ for capture results.
