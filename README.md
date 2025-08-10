# Quickly change audio output in Ubuntu (PipeWire) without problems with dynamic IDs
Problem:
In PipeWire, commands like ```wpctl set-default 33``` only work until the next restart, because the numbers (33, 81, etc.) change every time.

Solution:
We use pw-dump (which provides a JSON list of all audio nodes) + jq to identify the sink by description (name visible in wpctl status), not by ID.
Thus, the script works even if the IDs change.
## 1. Find out the descriptions of your audio outputs

Run:
```bash
pw-dump | jq -r '
  .[] | select(.type=="PipeWire:Interface:Node")
  | select(.info.props["media.class"]=="Audio/Sink")
  | "\(.id)\t\(.info.props["node.name"])\t\(.info.props["node.description"])"
'
```
See the result, for example:
```bash
33	alsa_output.pci-0000_09_00.4.analog-stereo	etc... HD Audio Controller Analog Stereo
81	alsa_output.pci-0000_07_00.1.hdmi-stereo	etc... HDMI/DP Audio Controller Digital Stereo (HDMI)
```
## 2. Create the script for the first output (e.g. Analog / headphones) and save it as ~/.local/bin/audio-to-headphones.sh, or any other name.

Make it executable
```bash
chmod +x ~/.local/bin/audio-to-hdmi.sh
```
Make two keyboard shortcuts

Add one for HDMI → Command: ```~/.local/bin/audio-to-hdmi.sh```

Add one for Analog → Command: ```~/.local/bin/audio-to-headphones.sh```

### Advantages over classic methods:
- You don't depend on volatile IDs.
- Works even after update/reboot.
- Automatically moves all active streams to the new output.
- Can be run from terminal or shortcut.
