#### Step1: Clone the repository

cd $HOME
git clone https://github.com/harsha544/supplier-retailer-app

#### Step 2: Change Directory

cd supplier-retailer-app

#### Step 3: Bring up Blockchain Network

sh build_ubuntu.sh

#### Step 4: Install Composer via nvm

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash

#### Step 5 : Install Node

nvm install v8.11.2

#### Step 6: Installing composer utilities

cd $HOME
npm install composer-cli
npm install composer-rest-server
npm install composer-playground

#### Step 7: Update PATH to include composer binaries

cd $HOME/
export PATH=$PATH:$PWD/node_modules/.bin/

#### Step 8: Update Supplier-Peer PEM Values in $PWD/composer/supplier-retailer.json

cd $HOME/supplier-retailer-app

awk 'NF {sub(/\r/, ""); printf "%s\\n",$0;}' cli/peers/peerOrganizations/supplier1-org/peers/supplier1-peer/tls/ca.crt > composer/supplier1Peer/supplier1-ca.txt 
cat composer/supplier1Peer/supplier1-ca.txt 

awk 'NF {sub(/\r/, ""); printf "%s\\n",$0;}' cli/peers/peerOrganizations/supplier2-org/peers/supplier2-peer/tls/ca.crt > composer/supplier2Peer/supplier2-ca.txt 
cat composer/supplier2Peer/supplier2-ca.txt 

awk 'NF {sub(/\r/, ""); printf "%s\\n",$0;}' cli/peers/peerOrganizations/supplier3-org/peers/supplier3-peer/tls/ca.crt > composer/supplier3Peer/supplier3-ca.txt
cat composer/supplier3Peer/supplier3-ca.txt 

for i in 1 2 3 ; do  printf "\n supplier$i \n"; cat composer/supplier"$i"Peer/supplier"$i"-ca.txt 
; done

Open *composer/supplier-retailer.json* and update 
**INSERT_ORG1_CA_CERT , INSERT_ORG2_CA_CERT, INSERT_ORG3_CA_CERT**
with supplier$i values from above respectively.


#### Step 9: Update Orderer PEM Values in $PWD/composer/supplier-retailer.json

awk 'NF {sub(/\r/, ""); printf "%s\\n",$0;}' cli/peers/ordererOrganizations/orderer-org/orderers/orderer0/tls/ca.crt > composer/orderer-ca.crt
cat composer/orderer-ca.crt
Open *composer/supplier-retailer.json* and update **INSERT_ORDERER_CA_CERT** with above value


#### Step 10:  Customizing the connection profile for respective supplier organization

cp composer/supplier-retailer.json composer/supplier-retailer_org.json

Update this block of code between sections version and channels, via which we enforce timeout period in *supplier-retailer_org.json*

    "client": {
        "organization": "Supplier1Org",
        "connection": {
            "timeout": {
                "peer": {
                    "endorser": "300",
                    "eventHub": "300",
                    "eventReg": "300"
                },
                "orderer": "300"
            }
        }
    },

Copy above generated supplier-retailer_org.json to respective supplierXPeer 

cp composer/supplier-retailer_org.json composer/supplier1Peer/supplier-retailer_org1.json
cp composer/supplier-retailer_org.json composer/supplier2Peer/supplier-retailer_org2.json
cp composer/supplier-retailer_org.json composer/supplier3Peer/supplier-retailer_org3.json


#### Step 11: Locating the certificate and private key for the Hyperledger Fabric administrator for each Supplier 

Locating certificates files respective supplier i.e identifying public part of identity for the users usually found under signcerts

export ORG1=cli/peers/peerOrganizations/supplier1-org/users/Admin@supplier1-org/msp/
export ORG2=cli/peers/peerOrganizations/supplier2-org/users/Admin@supplier2-org/msp/
export ORG3=cli/peers/peerOrganizations/supplier3-org/users/Admin@supplier3-org/msp/


cp -p $ORG1/signcerts/A*.pem composer/supplier1Peer/
cp -p $ORG2/signcerts/A*.pem composer/supplier2Peer/
cp -p $ORG3/signcerts/A*.pem composer/supplier3Peer/

Identify private key for the user which is used for signing transactions and found under keystore

cp -p $ORG1/keystore/*_sk composer/supplier1Peer/
cp -p $ORG2/keystore/*_sk composer/supplier2Peer/
cp -p $ORG3/keystore/*_sk composer/supplier3Peer/


#### Step 12: Creating business network cards for the Hyperledger Fabric administrator for all Orgs


composer card create -p supplier1Peer/supplier-retailer-org1.json -u PeerAdmin -c supplier1Peer/Admin@supplier1-org-cert.pem -k supplier1Peer/*_sk -r PeerAdmin -r ChannelAdmin -f PeerAdmin@supplier-retailer-org1.card

composer card create -p supplier2Peer/supplier-retailer-org2.json -u PeerAdmin -c supplier2Peer/Admin@supplier2-org-cert.pem -k supplier2Peer/*_sk -r PeerAdmin -r ChannelAdmin -f PeerAdmin@supplier-retailer-org2.card

composer card create -p supplier3Peer/supplier-retailer-org3.json -u PeerAdmin -c supplier3Peer/Admin@supplier3-org-cert.pem -k supplier3Peer/*_sk -r PeerAdmin -r ChannelAdmin -f PeerAdmin@supplier-retailer-org3.card


#### Step 13: Importing the business network cards for the Hyperledger Fabric administrator for all Orgs

composer card import -f supplier1Peer/PeerAdmin@supplier1.card --card PeerAdmin@supplier-retailer-org1
composer card import -f supplier2Peer/PeerAdmin@supplier2.card --card PeerAdmin@supplier-retailer-org2
composer card import -f supplier3Peer/PeerAdmin@supplier3.card --card PeerAdmin@supplier-retailer-org3

Issue *composer card list* to verify whether cards have been imported successfully or not


#### Step 14: Installing the business network onto the Hyperledger Fabric peer nodes for

cd ../chaincode/composer/
composer network install --card PeerAdmin@supplier-retailer-org1 --archiveFile retailer-supplier@0.0.1.bna
composer network install --card PeerAdmin@supplier-retailer-org2 --archiveFile retailer-supplier@0.0.1.bna
composer network install --card PeerAdmin@supplier-retailer-org3 --archiveFile retailer-supplier@0.0.1.bna

