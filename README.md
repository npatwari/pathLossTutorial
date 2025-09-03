## Path Loss Measurements using the Shout Measurement Framework

### Learning objectives

- Be able to use the Shout measurement framework to automate TX/RX functions across multiple POWDER nodes
- Work with complex-valued signals in the time domain and frequency domain
- Compute the received power from a received complex-valued signal
- Know how received power decreases as a function of path length

#### Group resource assignments table

Refer to this table when instructed to do so in the steps below.  Be sure to look at the correct entry for your group!

| Group # | Nodes | Frequency Min | Frequency Max | Center Frequency | TX/RX Gains |
|---------|--------------|--------|-------|------|---------|
| 1 | `cbrssdr1-browning`, `cbrssdr1-ustar`, `cbrssdr1-hospital` | 3401 | 3405 | 3403 | TX: 27, RX:30 |
| 2 | `cbrssdr1-bes`, `cbrssdr1-fm`, `cbrssdr1-honors` | 3406 | 3410 | 3408 | TX: 27, RX:30 |
| 3 | ~~`cnode-mario`~~, `cnode-moran`, `cnode-guesthouse` | 3411 | 3415 | 3413 | TX: 85, RX:70 |
| 4 | `cnode-wasatch`, `cnode-ebc`, `cnode-ustar` | 3416 | 3419.9 | 3418 |  TX: 85, RX:70 |

#### Within group assignments
Assuming four people in your group, assign each person:
1. Work A
2. Work B
3. Work C
4. Work D

If you have fewer than four, some will do multiple jobs. You will mostly need to use one laptop, and pass it around.

All should clone this git repo to their laptop.

### Preliminary steps:

(Work A)

You will run these steps before proceeding with the manual measurement gathering exercise so that your individual experiments can be instantiating while we proceed with part 1 below.

* Instantiate the `shout-long-measurement` profile and start an experiment for your group
  - Navigate to: https://www.powderwireless.net/
  - Go to Experiments: Start Experiment
  - Click "Change Profile" and select "shout-long-measurement"
  - Click "Next" to get to the "Parameterize" tab.
    - Use a compute node and orchestrator node type of d430
    - Use a Dataset to connect of "None"
    - If you're in Groups 1 or 2, expand the "CBAND X310 radios.".  If you're in Groups 3 or 4, expand the "Dense Site NUC+B210 radios to allocate." 
    - Select the node names according to your group assignment (in the table above)
    - Expand "Frequency ranges for CBAND operation." and use a min and max as given in the table above according to your group.
    - Click "Next"
  - On the "Finalize" step:
    - Give your experiment a short name including your group number
    - Select the "WNS2025" project if you see a project selection box, 
    - Click "Next".
  - On the "Schedule" step, leave all fields at their defaults and click "Finish"
  - Wait for the experiment to instantiate (turn green), and the startup scripts to finish

You should now have your own three SDR, four compute node experiment. One "Orchestrator" computer for interacting with the Shout measurement clients, each running on a separate compute node and connected to an SDR. 

### Check Each SDR

(Work B)

This task will check each SDR configuration by logging into its compute node. I find it easiest to copy and paste the following commands and edit them. That way you can copy and paste your three ssh commands exactly as you want them.

```
ssh -Y -p 22 -t username@pcXXX.emulab.net 'cd /local/repository/bin && tmux new-session -A -s shout &&  exec $SHELL'
ssh -Y -p 22 -t username@pcYYY.emulab.net 'cd /local/repository/bin && tmux new-session -A -s shout &&  exec $SHELL'
ssh -Y -p 22 -t username@pcZZZ.emulab.net 'cd /local/repository/bin && tmux new-session -A -s shout &&  exec $SHELL'
```
where "username" should be replaced with your username, and pcXXX, pcYYY, and pcZZZ are the node names that you are using.

Open up three terminal windows. Use one of the above on each window, so that you are connected to three compute nodes. Once connected, run `uhd_usrp_probe`. If successful, it will show some information about the setup of the SDR.

If instead you get 

> Error: RuntimeError: Expected FPGA compatibility number 38, but got 39:
> The FPGA image on your device is not compatible with this host code build. 

then do the following steps to flash the FPGA with the correct version of build:
1. Run: `/local/repository/bin/update_x310.sh`, which takes about 5 minutes.
2. When that is complete, Power Cycle the x310: Find the row in the List View of your POWDER experiment page with the name of your node "-x310". Using the Settings icon in that row (all the way to the right), select "Power Cycle" and click "Confirm".
3. After about a minute, run `uhd_find_devices` and `uhd_usrp_probe` to check that the radio is back on and now has the correct FPGA build version.


### Run Shout

(Work C)

Copy and paste the following into a text editor to keep handy:
```
ssh -Y -p 22 -t username@pcWWW.emulab.net 'cd /local/repository/bin && tmux new-session -A -s shout1 &&  exec $SHELL'
ssh -Y -p 22 -t username@pcWWW.emulab.net 'cd /local/repository/bin && tmux new-session -A -s shout2 &&  exec $SHELL'
```
Change `username` to your username, and change pcWWW are the node name of your orchestrator that you are using.

Copy and paste the first command into a terminal window to connect to the orchestrator.

#### Edit the Experiment Parameters JSON File

We're going to need to edit a text file. People have their own preference for editor. I like `vi` but I know no one else who does. I suggest the `nano` editor. So run

`nano /local/repository/etc/cmdfiles/save_iq_w_tx_cw.json`

You're going to edit the entries of the JSON file parameter structure to have the following values:
 - "cmd": "save_iq_w_tx",
 - "sync": true,
 - "timeout": 30,
 - "nsamps": 8192,
 - "wotxrepeat": 0,
 - "txrate": 500e3,
 - For "txfreq": use the center frequency for your group. Because it is in MHz, write it with an `e6` at the end. For example, for 3386 MHz as a center frequency, write `3386e6`.
 - For "rxfreq": use the exact same number as "txfreq".
 - "txgain": `{"fixed": XX.0}`, where `XX` is your gain for your SDR transmitter, either 27 or 85.  
 - "txwait": 3,
 - "rxrate": 500e3,
 - "rxgain": `{"fixed": YY.0}`, where `YY` is your gain for your SDR receiver, either 30 or 70.  
 - "rxrepeat": 4,
 - "rxwait": {"min": 50, "max": 2000, "res": "ms"},
 - "txclients":  Make a list of your three compute node names -- the ones ending in "-comp" if you're in Group 1 or 2. For example: `["cbrssdr1-honors-comp", "cbrssdr1-hospital-comp", "cbrssdr1-ustar-comp"]`,
 - "rxclients": The exact same list as for txclients.

Save the file (`^O`) and close the editor (`^X`).

### Execute the Shout commands

(Work D)

(The following commands assume you are in the `/local/repository/bin/` directory -- if you're not, do a `cd /local/repository/bin/`.)

One one of your orchestrator windows, run
```
./1.start_orch.sh
```
When it is done, on each of your compute nodes (attached to your three SDRs), run
```
./2.start_client.sh
```
(these can be started in parallel.)

When all three compute nodes are done, go to the 2nd orchestrator terminal window and run
```
./3.run_cmd.sh
```


### Copy your data back to your local laptop

(Work D)

After the 3.run_cmd.sh is done, while still on the orchestrator window, do a `ls /local/data/` to see your data directory. Make a note of the directory name. If there is more than one, use the most recent.

Back on a terminal window on your local laptop, run
```
scp -r username@pcWWW.emulab.net:/local/data/<Data dir> <local git directory for pathLossTutorial>
```
where, again, change username, and change pcWWW to the orchestrator name; and change `<Data dir>` to the directory you saw when running the `ls /local/data/` command, and `<local git directory for pathLossTutorial>` to the folder on your local laptop where you're running the pathLossTutorial.

Zip this local directory `data/` directory and share it with everyone on the team.

## Analyze the Data

(All team members individually)

In this step, you will load and run a Jupyter notebook. The file, `CheckShoutData.ipynb` is in this repo. It is also a [public Google Colab notebook](https://colab.research.google.com/drive/1g0DbZOUlDLly6y1m5UzvOpGiR9h4DKFC?usp=sharing) if you prefer to run the notebook on the cloud.

You'll edit and save your .ipynb file, and turn it in to the Canvas assignment.
