# How I improved my microphone sound

My computer sits in the living room, meaning background noise is a constant. Since I talk a lot online, I wanted a cleaner, more consistent mic signal without having to move room or buy new gear. I ended up using [Carla](https://kx.studio/Applications:Carla) with [PipeWire](https://wiki.archlinux.org/title/PipeWire) to process my voice and present the result as a virtual microphone to any app.

## Creating a virtual input

I made a new virtual input device that will take Carla's processed output and show up like a normal microphone in audio settings.

```bash
cd ~/.config/pipewire/pipewire.conf.d/
echo <<EOF > 11-carla-mic.conf
context.objects = [
    {
        factory = adapter
        args = {
            factory.name = support.null-audio-sink
            node.name = "carla-mic"
            node.description = "Carla Virtual Mic"
            media.class = "Audio/Source/Virtual"
            audio.position = [ FL FR ]
            device.description = "Virtual Microphone with Carla"
            device.class = "sound"
            device.icon-name = "audio-card"
            node.virtual = false
            monitor.channel-volumes = true
            monitor.passthrough = true
            adapter.auto-port-config = {
                mode = dsp
                monitor = true
                position = preserve
            }
        }
    }
]
EOF
systemctl --user restart pipewire
systemctl --user status pipewire
wpctl status
```

After restarting PipeWire the device appears in pavucontrol and other apps just like a regular input. Now any program can use the processed audio without extra setup.

## Processing the audio in Carla

I installed the needed packages and ran Carla under PipeWire's JACK compatibility:

```bash
sudo pacman -Syu carla pipewire-jack lv2-plugins

pw-jack carla
```

In Carla: Settings > Configure > Engine > Core, I set **Audio Driver** to “JACK” and **Process mode** to “Multiple Clients”.

In the Patchbay I added and chained these plugins (in order):

1. LSP Filter (Lo‑Cut) — removes low rumble

2. LSP Gate — cuts sounds below a threshold

3. LSP Compressor — evens out level variations

Finally I routed the chain to the virtual microphone node created earlier.

## Final thoughts

Overall this gives me a reliable, low-latency way to clean up my mic that fits into my daily workflow without extra hardware.

Some ways to further improve and automate the setup:

- Keyboard OSC control: Bind keys to send OSC commands to Carla so I can toggle plugins, change presets, or add effects like reverb on the fly without opening the GUI.

- Headless startup: Launch Carla in headless mode at startup so the processing chain is available immediately and reliably.

At first I thought it'd be another endless rabbithole with JACK and stuff, but it was way easier to deal with.
