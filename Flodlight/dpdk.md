### FLOODLIGHT CONTROLER
```apt-get install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
apt-get update
apt-get install -y docker-ce

# floodlight
# https://hub.docker.com/r/glefevre/floodlight/
docker run -d -p 6653:6653 -p 8080:8080 --name=floodlight --restart always glefevre/floodlight 
```
