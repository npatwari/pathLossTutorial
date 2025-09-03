## H2: Path loss measurements & Shout measurement framework

### Learning objectives

- Understand how to request spectrum resources in POWDER
- Understand how to safely perform basic RF transmission 
- Understand how the Shout framework can be used to automate TX/RX functions across multiple POWDER nodes
- Understand how Gold Codes/Sequences can be used to identify user generated transmission (from other RF transmissions)
- Understand path loss within the context of POWDER

### Intro slides for this session

[Slides for this session are located here](https://docs.google.com/presentation/d/16bR0SpjxCsY6oy0OHGzySBcarQEMr0YzxRmyMd1lxMk/edit?usp=share_link)

#### Preliminary steps:

We will run these steps before proceeding with the manual measurement gathering exercise so that your individual experiments can be instantiating while we proceed with part 1 below.

* Instantiate the `shout-iface-node` profile
  - Navigate to: https://www.powderwireless.net/p/mww2023/shout-iface-node
  - Click "Instantiate", then click "Next" on the subsequent page to go to the "Parameterize" step
    - Enter the name of the orchestrator node if told to do so (otherwise, leave this field at the default)
    - Leave all other parameters at their defaults and click "Next"
  - On the "Finalize" step, select the `mww2023` project if you see a project selection box, then click "Next"
    - Don't enter an experiment name
  - On the "Schedule" step, leave all fields at their defaults and click "Finish"
  - Wait for the experiment to instantiate (turn green), and the startup scripts to finish

You should now have your own single node experiment for interacting with the Shout measurement clients running in a separate experiment. We will have the Designated Person (DP) in your group run the commands that actually gather measurements from the real radio devices.

### H2 hands-on walkthrough instructions

***For both parts of this hands-on exercise, a designated person (DP) in each group will run the commands related to actually performing measurements.  Others in the group should not run these commands!***

The walkthrough is broken into three parts: preliminary experiment swap-in, manual measurements where the DP logs into specific nodes in an already-running experiment to conduct measurements, and automated measurements using the Shout framework from a separate single-node experiment.

#### Group resource assignments table

Refer to this table when instructed to do so in the steps below.  Be sure to look at the correct entry for your group!
| Group # | Part 1 nodes | Part 2 TX node |
|---------|--------------|----------------|
| 1 & 8 | node1: browning, node2: bes | browning |
| 2 & 9 | node1: bes, node2: fm | bes |
| 3 & 10 | node1: fm, node2: honors | fm |
| 4 & 11 | node1: honors, node2: hospital | honors |
| 5 & 12 | node1: hospital, node2: smt | hospital |
| 6 & 13 | node1: smt, node2: ustar | smt |
| 7 & 14 | node1: ustar, node2: browning | ustar |

#### Part 1: Manual measurements

***Only the Designated Person (DP) from each group should follow these steps!***

* Navigate to the running experiment with the outdoor radio resources in it
  - Link: [No longer active]
* Open an in-browser X windows session for both nodes - **Look up your group's node names in the table above.**
  - Click the "List View" tab to get to the list of nodes and devices in the experiment
  - At the end of the row for each of the two nodes, click the "gear" icon and select "Open VNC Window"
    - This will open a new browser window with a VNC-carried X windows session running in it.
    - Click inside the terminal window a couple of times to ensure it has focus
* Prepare to start the transmitter script on your "node1" node by pasting the following command into the terminal:
  ```
  /local/repository/qam-dsss/standalone/separate_TX.py -c /local/repository/etc/sa-params.json
  ```
  - DO NOT hit enter yet!
* Prepare to start the receiver script on your "node2" node by pasting the following command into the terminal:
  ```
  /local/repository/qam-dsss/standalone/separate_RX.py -p -t -c /local/repository/etc/sa-params.json
  ```
  - DO NOT hit enter yet!
* ***WHEN INSTRUCTED TO DO SO:*** Hit enter first in your "node1" terminal, then hit enter in your "node2" terminal
  - The receiver will run and then print some data that we will discuss after all runs are complete

#### Part 2: Automatic measurements with Shout

Everyone will go through the initial steps below to prepare the Shout configuration file, but only the Designated Person (DP) in each group will actually run the measurement commands (marked prominently below).

* Open a VNC window for the node in your new experiment
  - On the "List View" tab, click the gear icon at the end of the node's row and select "Open VNC Window"
    - This will pop up a browser-based VNC window
* Edit the Shout configuration file for doing a DSSS with Gold Codes measurement run
  - At the shell in the VNC session, open up the following file in an editor: `/local/repository/etc/cmdfiles/save_iq_w_tx_gold.json`
    - E.g., to open the file with `nano`: `nano /local/repository/etc/cmdfiles/save_iq_w_tx_gold.json`
  - Check that the `txfreq` parameter is set to the _exact_ value the instructor has specified
    - This will be the center frequency that the Shout measurement transmitter client tunes to
  - Check that the `rxfreq` parameter is set to the _exact_ value the instructor has specified
    - This will be the center frequency that the Shout measurement receiver clients tune to
  - Replace the `node` portion of the single entry in the `txclients` array parameter with the node ID listed in the table above
    - Do NOT use a different value!  Double check that you have the right node name.
  - Leave all other parameters alone
    - Instructor will describe some other parameters of interest, but they should not be changed for this exercise
  - Save your modified configuration file and exit the editor 
    - (Ctrl-S to save, then Ctrl-X to exit the `nano` editor)

**NEXT STEPS ARE FOR THE DESIGNATED PERSON IN EACH GROUP ONLY!**

* **WHEN INSTRUCTED TO DO SO:** Execute the measurement run
  - We will have each group's DP execute a measurment run in turn (sequentially)
  - **DO NOT RUN UNTIL INSTRUCTED TO PROCEED**: `/local/repository/bin/3.run_cmd.sh`
  - This command will show computed RSSI for the signal received at each receiver as the measurements proceed.
    - Total execution time should be under a minute
    - Wait for the command to exit back to the shell command prompt
    - **DO NOT** execute additional measurement runs unless instructed to do so
* Plot the results of the measurements after they have been collected
  - DPs execute the following commands from the shell in their VNC session:
```
cd /local/data
/local/repository/shout/dsss/DSSS_Processing.py -o ./ -f Shout_meas_datestr_timestr
```
  - Hit the tab key after typing `Shout_meas_` to autocomplete the rest of the file name, then "enter" to run
      - Do not literally type "Shout_meas_datestr_timestr"; the "datestr" and "timestr" strings are just placeholders in this guide
  - You will get three plots:
    - Plot of the first run showing the correlation graph for each receiver
    - Plot showing RSSI and SINR vs. the receiver's distance from the transmitter
    - Plot of the SINR and RSSI values vs. measurement run for each receiver
      - There are 10 measurements total for each
  - We will discuss these plots, but feel free to zoom in, pan around, etc.
      - The button with the house icon will reset the view on the plot
