# t3rn Executor Process Management

This guide provides detailed instructions for setting up, managing, and monitoring Node.js processes using PM2 and tmux. It covers installation, configuration, auto-start setup, and using tmux for persistent and enhanced monitoring. Tailored for [t3rn](https://docs.t3rn.io/executor/become-an-executor/binary-setup) executors.

## Requirements
Before setting up the executor, make sure you have the following installed:

- [Node.js](https://nodejs.org/en/)
- [npm](https://www.npmjs.com/)
- [PM2](https://pm2.keymetrics.io/)
- [tmux](https://github.com/tmux/tmux) | Optional

## Setup Instructions

### 1. Install Node.js, npm, PM2 and tmux
Follow the steps below to install the required components:

- **Install Node.js**:
    ```bash
    sudo apt install nodejs
    ```
    Verify the installation:
    ```bash
    node -v
    ```

- **Install npm**:
    ```bash
    sudo apt install npm
    ```
    Verify the installation:
    ```bash
    npm -v
    ```

- **Install PM2**:
    ```bash
    npm install pm2 -g
    ```
    Verify the installation:
    ```bash
    pm2 -v
    ```
- **Install tmux**:
    ```bash
    sudo apt install tmux
    ```
    Verify the installation:
    ```bash
    tmux -v
    ```
    
### 2. Setup Shell Script and PM2
Follow these steps to set up the executor script and manage it using PM2:

1. **Open a terminal in the executor folder.**
2. **Create a new file, for example, `start_executor.sh`**:
    ```bash
    nano start_executor.sh
    ```
3. **In the script file, add the following content** (Ensure you **update the settings and variables** according to your needs):
    ```bash
    #!/bin/bash

    export NODE_ENV=testnet  # Only the 'testnet' environment is available
    export LOG_LEVEL=debug  # Logging level for outputting detailed logs for debugging and troubleshooting
    export LOG_PRETTY=false  # It's recommended to keep this feature set to 'false' for now, as enabling it may cause unexpected behavior while it's still being refined
    export EXECUTOR_PROCESS_ORDERS=true  # Set this to 'false' if you don't want to process orders
    export EXECUTOR_PROCESS_CLAIMS=true  # Set this to 'false' if you don't want to process claims
    export EXECUTOR_MAX_L3_GAS_PRICE=000  # Update your gas max limit with the appropriate value (in Gwei). You can check the gas tracker link at the end of this page
    export RPC_ENDPOINTS_ARBT='0000000000000000000000000000000000000000000000000000000000000000'  # Update with your Arbitrum Sepolia RPC URL
    export RPC_ENDPOINTS_BSSP='0000000000000000000000000000000000000000000000000000000000000000'  # Update with your Base Sepolia RPC URL
    export RPC_ENDPOINTS_BLSS='0000000000000000000000000000000000000000000000000000000000000000'  # Update with your Blast Sepolia RPC URL
    export RPC_ENDPOINTS_OPSP='0000000000000000000000000000000000000000000000000000000000000000'  # Update with your OP Sepolia RPC URL
    export RPC_ENDPOINTS_L1RN='https://brn.rpc.caldera.xyz/'  # L1RN RPC URL
    export PRIVATE_KEY_LOCAL=0000000000000000000000000000000000000000000000000000000000000000  # Update with your private key
    export ENABLED_NETWORKS='arbitrum-sepolia,base-sepolia,blast-sepolia,optimism-sepolia,l1rn'  # Specify the networks you want to enable for processing/claiming orders
    export EXECUTOR_PROCESS_PENDING_ORDERS_FROM_API=false  # Set to 'true' if you want to process pending orders from the API

    ./executor  # Run the executor with the current configuration
    ```
4. **OPTIONAL | Monitor logs in terminal and write to Log file simultaneously** (Note that the log file size can become **large**, depending on the duration of your executor process):
    ```bash
    #!/bin/bash

	export NODE_ENV=testnet  # Only the 'testnet' environment is available
	export LOG_LEVEL=debug  # Logging level for outputting detailed logs for debugging and troubleshooting
	export LOG_PRETTY=false  # It's recommended to keep this feature set to 'false' for now, as enabling it may cause unexpected behavior while it's still being refined
	export EXECUTOR_PROCESS_ORDERS=true  # Set this to 'false' if you don't want to process orders
	export EXECUTOR_PROCESS_CLAIMS=true  # Set this to 'false' if you don't want to process claims
	export EXECUTOR_MAX_L3_GAS_PRICE=000  # Update your gas max limit with the appropriate value (in Gwei)
	export RPC_ENDPOINTS_ARBT='0000000000000000000000000000000000000000000000000000000000000000'  # Update with your Arbitrum Sepolia RPC URL
	export RPC_ENDPOINTS_BSSP='0000000000000000000000000000000000000000000000000000000000000000'  # Update with your Base Sepolia RPC URL
	export RPC_ENDPOINTS_BLSS='0000000000000000000000000000000000000000000000000000000000000000'  # Update with your Blast Sepolia RPC URL
	export RPC_ENDPOINTS_OPSP='0000000000000000000000000000000000000000000000000000000000000000'  # Update with your OP Sepolia RPC URL
	export RPC_ENDPOINTS_L1RN='https://brn.rpc.caldera.xyz/'  # L1RN RPC URL
	export PRIVATE_KEY_LOCAL=0000000000000000000000000000000000000000000000000000000000000000  # Update with your private key
	export ENABLED_NETWORKS='arbitrum-sepolia,base-sepolia,blast-sepolia,optimism-sepolia,l1rn'  # Specify the networks you want to enable for processing/claiming orders
	export EXECUTOR_PROCESS_PENDING_ORDERS_FROM_API=false  # Set to 'true' if you want to process pending orders from the API

	LOG_FILE="$(pwd)/executor_logs_$(date +'%Y-%m-%d_%H-%M-%S').log"  # Define log file location with timestamp

	{  # Start the executor and save all the output (including errors) to a log file while also displaying it in the terminal.
	  echo "Starting executor process at $(date)"		
	  ./executor
	} 2>&1 | tee -a "$LOG_FILE"  # This will log output to both the terminal and a log file
    ```
        
5. **Make the shell script executable**:
    ```bash
    sudo chmod +x start_executor.sh
    ```

6. **Start the executor with PM2**:
    ```bash
    pm2 start ./start_executor.sh --name "executor-process"
    ```

### 3. Managing PM2 Executor Process
You can manage the executor process using PM2 with the following commands:

- **View all running processes:**
    ```bash
    pm2 list
    ```

- **Monitor the executor process:**
    ```bash
    pm2 logs executor-process
    ```

- **Stop the executor process:**
    ```bash
    pm2 stop executor-process
    ```

- **Restart the executor process:**
    ```bash
    pm2 restart executor-process
    ```

- **Delete the executor process:**
    ```bash
    pm2 delete executor-process
    ```

### 4. Enable Auto-Start with PM2 (Recommended)
To ensure the executor restarts automatically after a reboot (Make sure the PM2 executor process status is online before proceeding):

- **Generate the PM2 startup command line:**
    ```bash
    pm2 startup
    ```
- **Run the command line generated by PM2 (It might look like this):**<br>
`sudo env PATH=$PATH:/usr/bin pm2 startup systemd -u $USER --hp $HOME`

- **Save the PM2 process list:**
    ```bash
    pm2 save
    ```

### 5. Pause Auto-Start Temporarily
To temporarily stop auto-starting the executor:

- **Stop the executor process:**
    ```bash
    pm2 stop executor-process
    ```

- **Remove the executor process from the saved list:**
    ```bash
    pm2 save
    ```

- **Restart auto-start:**
    ```bash
    pm2 start executor-process
    pm2 save
    ```

### 6. Remove Auto-Start Configuration
To disable the PM2 startup executor and remove saved process:

- **Disable the PM2 startup executor:**
    ```bash
    pm2 unstartup
    ```
- **Run the command line generated by PM2 (It might look like this):**<br>
`sudo env PATH=$PATH:/usr/bin pm2 unstartup systemd -u $USER --hp $HOME`

- **Remove the executor process from the saved list:**
    ```bash
    pm2 save --force
    ```

### 7. Executor/PM2 in tmux Session (Optional)
For executor process persistence, use **tmux** to run PM2 inside a session:

1. **Create a new tmux session:**
    ```bash
    tmux new -s executor
    ```

2. **Run PM2 inside the tmux session to start the executor process:**
    ```bash
    pm2 start ./start_executor.sh --name "executor-process"
    ```

3. **After starting the executor process, detach from the tmux session:**
    - Press `Ctrl + b`, then `d`

4. **To reattach to the tmux session:**
    ```bash
    tmux attach -t executor
    ```

5. **Monitor the executor process with PM2 inside tmux:**
    ```bash
    pm2 logs executor-process
    ```

### Useful links

- [t3rn](https://www.t3rn.io/)
- [t3rn Docs](https://docs.t3rn.io/intro)
- [t3rn Bridge](https://bridge.t1rn.io/)
- [BRN Explorer](https://brn.explorer.caldera.xyz/)
- [Gas Tracker](https://brn.explorer.caldera.xyz/gas-tracker)



