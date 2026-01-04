# Install docker on EC2 vm
sudo yum install docker
sudo systemctl enable docker
sudo systemctl start docker
usermod -aG docker ec2-user

# Installing Docker Compose
sudo mkdir -p /usr/local/lib/docker/cli-plugins
sudo curl -SL https://github.com/docker/compose/releases/download/v2.27.0/docker-compose-linux-x86_64 \
  -o /usr/local/lib/docker/cli-plugins/docker-compose
sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose
exit
# Copy files to the VM
scp -i metabase.pem -r ~/games-multi-user/backup ec2-user@3.221.161.51:/home/ec2-user/
# Connect to the VM
ssh -i metabase.pem ec2-user@3.221.161.51
# Check Docker compose version 
docker compose version
# Start your image
cd ~/backup
docker compose up -d
# Check if its running 
docker ps
# To tunnel to Maria DB directly
ssh -i metabase.pem -L 3306:localhost:3306 ec2-user@....
ssh -i metabase.pem ec2-user@....
# Once connected 
cd /home/ec2-user/db_project
# Copy files to local2EC2
scp mariadb.tar ec2-user@<EC2_PUBLIC_IP>:/home/ec2-user/
scp metabase.tar ec2-user@<EC2_PUBLIC_IP>:/home/ec2-user/
# Load copied files to EC2
docker load -i db.tar
docker load -i meta.tar

# Check secrets for if sacred in compose file
ls -la 
cat .env #Create the secrets if needed #mod secrets 400
# Start instance
docker compose up -d

# Connect to Metabase from terminal
- Step 1
ssh -i metabase.pem -L 3000:localhost:3000 ec2-user@100.27.8.97
- Step 2
http://localhost:3000

# Connect to Mariadb from terminal:

ssh -i metabase.pem -L 3006:localhost:3006 ec2-user@100.27.8.97
# Connect from within VM
docker exec -it games-db-multi mariadb -u root -p
# Check env vars for metabase #####
docker exec games-metabase-multi printenv | grep MB_DB 

# If sql dump doe not work, explicitly copying the files of the whole db.
docker cp games-db-multi:/var/lib/mysql ./mysql-data

scp -i metabase.pem -r games-migration-package.tar.gz ec2-user@3.90.244.192:/home/ec2-user/
# INITIATING DATABASE WITH IDF FRM FILE TRANSFER OF DB --on  ec2 vm 
docker run -d --name games-db-multi   -v /home/ec2-user/mysql-data:/var/lib/mysql   -e MYSQL_ROOT_PASSWORD=!   -p 3306:3306 mariadb:11



