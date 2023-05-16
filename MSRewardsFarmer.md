# MS Rewards Farmer Bot

## Create the LXC

- Create a [Ubuntu LXC](https://tteck.github.io/Proxmox/), 1 core, 512 mb ram but give it **4gb** space (Chrome takes a lot of space)

## Create a user (optional)

- I gave mine a root password for ssh, and added a user:
- added a user with `sudo adduser [username]`

- gave them a password

- go back to root sudo su (or exit), and give them sudo privileges `usermod -aG sudo [username]`


## Install the Bot

- install github `sudo apt-get install git-all`

- install pip `sudo apt install python3-pip`

- install chrome 

  ```bash
  wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
  sudo apt install ./google-chrome-stable_current_amd64.deb
  ```

- clone the repo `git clone https://github.com/MehdiRtal/MicrosoftRewardsclaimer && cd MicrosoftRewardsFarmer`

- install dependencies `pip install -r requirements.txt
  `

- Update the config with your rewards account info, `nano config.json.sample` and be sure when saving to remove `sample`

- Run the script with `python3 MicrosoftRewardsBot.py --headless --everyday 06:00` to run it daily at 6:00 am. You can also run it manually with `python3 MicrosoftRewardsBot.py --headless`. You might want to do this first to get today's points.


