Write bash script with echo colorful logs that look like professional like :

Update and install sudo and then git , curl

and then this :

git clone https://github.com/boundless-xyz/boundless
cd boundless
git checkout release-0.10

and then rm scripts/setup.sh this file and then download this : https://raw.githubusercontent.com/Stevesv1/Boundless-Guide/refs/heads/main/setup.sh and save it as setup.sh at the same location which u deleted

this use this command : sudo ./scripts/setup.sh

curl -L https://risczero.com/install | bash
source ~/.bashrc
rzup install rust
rzup update r0vm
cargo install --git https://github.com/risc0/risc0 bento-client --bin bento_cli
just bento
cargo install --locked boundless-cli

then check how many GPUs are there in the systme using this command : nvidia-smi -L
if more than 1 then u need to add these in compose.yml

u will see someting like this already present for GPU 0 :


  gpu_prove_agent0:
    <<: *agent-common
    runtime: nvidia
    mem_limit: 4G
    cpus: 4
    entrypoint: /app/agent -t prove

    # comment-out if running in CPU proving mode
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              # TODO: how to scale this with N gpus?
              device_ids: ['0']
              capabilities: [gpu]

so when there is more gpu so let's oif there is 2 gpu then u need add these lines like this :

  gpu_prove_agent1:
    <<: *agent-common
    runtime: nvidia
    mem_limit: 4G
    cpus: 4
    entrypoint: /app/agent -t prove

    # comment-out if running in CPU proving mode
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              # TODO: how to scale this with N gpus?
              device_ids: ['1']
              capabilities: [gpu]

  gpu_prove_agent2:
    <<: *agent-common
    runtime: nvidia
    mem_limit: 4G
    cpus: 4
    entrypoint: /app/agent -t prove

    # comment-out if running in CPU proving mode
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              # TODO: how to scale this with N gpus?
              device_ids: ['2']
              capabilities: [gpu]

u need to change the device_ids and gpu probver agent number like this so if rtheere 3 gpu it should be generated in the file 2 times and in toital there will 3 

after that in the bwlow u will see someting like this :

  broker:
    restart: always
    depends_on:
      - rest_api
      - gpu_prove_agent0
      - exec_agent0
      - exec_agent1
      - aux_agent
      - snark_agent
      - redis
      - postgres

u need to add the name of gpur prover agent, if there is total 3 gpu in the system so it willl look like this :

  broker:
    restart: always
    depends_on:
      - rest_api
      - gpu_prove_agent0
      - gpu_prove_agent1
      - gpu_prove_agent2
      - exec_agent0
      - exec_agent1
      - aux_agent
      - snark_agent
      - redis
      - postgres

like this , make suire that everyting should be in the same row.

After that ask them on which network they want to run this prover

- Base Mainnet
- Base Sepolia
- Ethereum Sepolia

SUser can choose all these 3, like this  : 1,2,3

if all then u need to also create 3 files like this :

cp .env.broker-template .env.broker.base
cp .env.broker-template .env.broker.base-sepolia
cp .env.broker-template .env.broker.eth-sepolia

like this and then ask users their private key and then RPC url forbase mainnet, base sepolia, beth sepolia, depending on their previus sleection.

and after this u will see in those env file like this :

PRIVATE_KEY=

BOUNDLESS_MARKET_ADDRESS=
SET_VERIFIER_ADDRESS=

# Note you must use an RPC URL that supports event filters. Many free RPC providers 
# do not support them.
RPC_URL=
ORDER_STREAM_URL=

u need to input the private in all the 3 files and the rpc based on the network and file

and here are some more info :

# For Base env file
BOUNDLESS_MARKET_ADDRESS=0x26759dbB201aFbA361Bec78E097Aa3942B0b4AB8
SET_VERIFIER_ADDRESS=0x8C5a8b5cC272Fe2b74D18843CF9C3aCBc952a760
ORDER_STREAM_URL="https://base-mainnet.beboundless.xyz"

# For eth-sepolia env file
BOUNDLESS_MARKET_ADDRESS=0x13337C76fE2d1750246B68781ecEe164643b98Ec
SET_VERIFIER_ADDRESS=0x7aAB646f23D1392d4522CFaB0b7FB5eaf6821d64
ORDER_STREAM_URL="https://eth-sepolia.beboundless.xyz/"

# For Base-sepolia env file
BOUNDLESS_MARKET_ADDRESS=0x6B7ABa661041164b8dB98E30AE1454d2e9D5f14b
SET_VERIFIER_ADDRESS=0x8C5a8b5cC272Fe2b74D18843CF9C3aCBc952a760
ORDER_STREAM_URL="https://base-sepolia.beboundless.xyz"

u need to use these details in respcetive env file and then 

cp .env.broker-template .env.broker

then edit .env.broker file based on GPU number 

If 1 GPU then no need to modify, if 2 then make twice the value of max_concurrent_proofs = 2 and peak_prove_khz = 100 to max_concurrent_proofs = 4 and peak_prove_khz = 200
if 3 then max_concurrent_proofs = 6 and peak_prove_khz = 300 .

Like this and then the final 3 command :

just broker up ./.env.broker.base
just broker up ./.env.broker.base-sepolia
just broker up ./.env.broker.eth-sepolia
